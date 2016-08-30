nginxのステータスコード444
==========================

nginx独自のステータスコード444は、レスポンスヘッダを返さずにコネクションを切断できる。レスポンスヘッダを返さないので、ステータスコード444をクライアントが受け取ることはない。

http://nginx.org/en/docs/http/ngx_http_rewrite_module.html#return

> The non-standard code 444 closes a connection without sending a response header.

DoS攻撃など、特定条件のアクセスをBANするときに、なるべく計算機リソース消費を抑えたいときに使えるはず。

- 403を返すときなどと比べて、レスポンス送信のためのCPUコストを節約
- 403を返すときなどと比べて、レスポンスヘッダ分のネットワーク帯域を節約
- HTTP Keepalive接続してきた場合、即切断することで、リクエストレートを抑えられる
-- 特にHTTP2では1本のTCP接続上で並行してリクエストを送信できてしまうため、過剰アクセスがより簡単になる

nginx v1.10.0のソースコードを眺めてみる。

```c
// https://github.com/nginx/nginx/blob/9842cff/src/http/ngx_http_request.h#L102-L103

/* The special code to close connection without any response */
#define NGX_HTTP_CLOSE                     444
```

ngx_http_finalize_request -> ngx_http_terminate_request -> ngx_http_close_request -> ngx_http_close_connection -> ngx_close_connection -> ngx_close_socket とコールされ、最終的にはclose(2)される。

```c
// https://github.com/nginx/nginx/blob/9842cff/src/http/ngx_http_request.c#L2311-L2314

void
ngx_http_finalize_request(ngx_http_request_t *r, ngx_int_t rc)
{
// ...
        if (rc == NGX_HTTP_CLOSE) {
            ngx_http_terminate_request(r, rc);
            return;
        }
// ...
}
```

```c
// https://github.com/nginx/nginx/blob/9842cff/src/http/ngx_http_request.c#L2445-L2490
static void
ngx_http_terminate_request(ngx_http_request_t *r, ngx_int_t rc)
{
// ...

    ngx_http_close_request(mr, rc);
}
```

```c
// https://github.com/nginx/nginx/blob/9842cff/src/http/ngx_http_request.c#L3377-L3407
static void
ngx_http_close_request(ngx_http_request_t *r, ngx_int_t rc)
{
// ...
    ngx_http_free_request(r, rc);
    ngx_http_close_connection(c);
}
```

```c
// https://github.com/nginx/nginx/blob/9842cff/src/http/ngx_http_request.c#L3516-L3547
void
ngx_http_close_connection(ngx_connection_t *c)
{
//  ...
    ngx_close_connection(c);

    ngx_destroy_pool(pool);
}
```

```c
// https://github.com/nginx/nginx/blob/9842cff/src/core/ngx_connection.c#L1110-L1198
void
ngx_close_connection(ngx_connection_t *c)
{
// ...

    if (ngx_close_socket(fd) == -1) {
// ...
}
```

```c
// https://github.com/nginx/nginx/blob/9842cff/src/os/unix/ngx_socket.h#L60-L61
#define ngx_close_socket    close
#define ngx_close_socket_n  "close() socket"
```
