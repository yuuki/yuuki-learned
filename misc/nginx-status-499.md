nginxのステータス499
====================

nginx v1.10のソースコードのコメントによると、

```c
// https://github.com/nginx/nginx/blob/9842cff/src/http/ngx_http_request.h#L120-L126

/*
 * HTTP does not define the code for the case when a client closed
 * the connection while we are processing its request so we introduce
 * own code to log such situation when a client has closed the connection
 * before we even try to send the HTTP header to it
 */
#define NGX_HTTP_CLIENT_CLOSED_REQUEST     499
```

- HTTP標準では定義されていないnginx独自のステータスコード
- nginxがリクエストを処理している間にクライアントがコネクションを切ったときのためのもの
- nginxがHTTPヘッダをクライアントに送信する前に、クライアントがコネクションを切断済みであれば、コード499をログに記録する

クライアントとのコネクションが切れているため、実際にレスポンスヘッダにコード499を含めてクライアントに返すことはない。おそらくログに状態を記録するためだけのステータスコード。

## ELB + nginxの例

```
ELB -> nginx -> upstream
```

上記のような構成の場合、ELBのタイムアウト設定が60秒なため、ELB-nginx間のコネクションをELB(クライアント)が切断する。したがって、upstream(アプリケーションサーバなど)が60秒を超えるリクエスト処理をしていたりすると、コード499が記録される。

## nginx v1.10 での挙動

- 以下の状況で、コード499が記録される
  - クライアントとのコネクションに対して getsocktopt(2) がエラーを返したとき
  - クライアントとのコネクションに対して recv(2) したときに、読み込んだデータサイズが0であるまたはエラーを返しつつerrnoがEAGAINでない場合

```c
// https://github.com/nginx/nginx/blob/9842cff/src/http/ngx_http_request.c#L2753-L2776

#if (NGX_HAVE_EPOLLRDHUP)

    if ((ngx_event_flags & NGX_USE_EPOLL_EVENT) && rev->pending_eof) {
        socklen_t  len;

        rev->eof = 1;
        c->error = 1;

        err = 0;
        len = sizeof(ngx_err_t);

        /*
         * BSDs and Linux return 0 and set a pending error in err
         * Solaris returns -1 and sets errno
         */

        if (getsockopt(c->fd, SOL_SOCKET, SO_ERROR, (void *) &err, &len)
            == -1)
        {
            err = ngx_socket_errno;
        }

        goto closed;
    }

#endif
```c

```c
// https://github.com/nginx/nginx/blob/9842cff/src/http/ngx_http_request.c#L2780-L2798

void
ngx_http_test_reading(ngx_http_request_t *r)
{

// ...

    n = recv(c->fd, buf, 1, MSG_PEEK);

    if (n == 0) {
        rev->eof = 1;
        c->error = 1;
        err = 0;

        goto closed;

    } else if (n == -1) {
        err = ngx_socket_errno;

        if (err != NGX_EAGAIN) {
            rev->eof = 1;
            c->error = 1;

            goto closed;
        }
    }
```

```c
// https://github.com/nginx/nginx/blob/9842cff/src/http/ngx_http_request.c#L2811-L2821

void
ngx_http_test_reading(ngx_http_request_t *r)
{

// ...

closed:

    if (err) {
        rev->error = 1;
    }

    ngx_log_error(NGX_LOG_INFO, c->log, err,
                  "client prematurely closed connection");

    ngx_http_finalize_request(r, NGX_HTTP_CLIENT_CLOSED_REQUEST);
}
```


```c
// https://github.com/nginx/nginx/blob/9842cff/src/http/ngx_http_request.c#L2290-L2305

void
ngx_http_finalize_request(ngx_http_request_t *r, ngx_int_t rc)
{

// ...

    if (rc == NGX_ERROR
        || rc == NGX_HTTP_REQUEST_TIME_OUT
        || rc == NGX_HTTP_CLIENT_CLOSED_REQUEST
        || c->error)
    {
        if (ngx_http_post_action(r) == NGX_OK) {
            return;
        }

        if (r->main->blocked) {
            r->write_event_handler = ngx_http_request_finalizer;
        }

        ngx_http_terminate_request(r, rc);
        return;
    }

```

### upstreamへのリクエスト時

nginxがクライアントとしてupstreamサーバにリクエストする場合にも、コード499が記録される。 TODO

```c
// https://github.com/nginx/nginx/blob/9842cff/src/http/ngx_http_upstream.c#L1220-L1267

static void
ngx_http_upstream_check_broken_connection(ngx_http_request_t *r,
    ngx_event_t *ev)
{

// ...

#if (NGX_HAVE_EPOLLRDHUP)

    if ((ngx_event_flags & NGX_USE_EPOLL_EVENT) && ev->pending_eof) {
        socklen_t  len;

        ev->eof = 1;
        c->error = 1;

        err = 0;
        len = sizeof(ngx_err_t);

        /*
         * BSDs and Linux return 0 and set a pending error in err
         * Solaris returns -1 and sets errno
         */

        if (getsockopt(c->fd, SOL_SOCKET, SO_ERROR, (void *) &err, &len)
            == -1)
        {
            err = ngx_socket_errno;
        }

        if (err) {
            ev->error = 1;
        }

        if (!u->cacheable && u->peer.connection) {
            ngx_log_error(NGX_LOG_INFO, ev->log, err,
                        "epoll_wait() reported that client prematurely closed "
                        "connection, so upstream connection is closed too");
            ngx_http_upstream_finalize_request(r, u,
                                               NGX_HTTP_CLIENT_CLOSED_REQUEST);
            return;
        }

        ngx_log_error(NGX_LOG_INFO, ev->log, err,
                      "epoll_wait() reported that client prematurely closed "
                      "connection");

        if (u->peer.connection == NULL) {
            ngx_http_upstream_finalize_request(r, u,
                                               NGX_HTTP_CLIENT_CLOSED_REQUEST);
        }

        return;
    }

#endif
```

## 参考

- [499 CLIENT CLOSED REQUEST](https://httpstatuses.com/499)
- [nginxのログに499(レスポンスコード)が記録される場合](http://d.hatena.ne.jp/hiroi10/20130306/1362591114)
- [HTTPステータスコード499のとき、GETリクエストが即座に再送される？](http://d.hatena.ne.jp/raugisu/20120619/1340120685)

