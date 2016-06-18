リバースプロキシのコンフィグテスト
==================================

## やりたいこと

- Apacheをnginxに置き換えたい
- Apacheやnginxをバージョンアップしたい

## テストパターン

- パターン1: 本番のproxyサーバに実際にリクエストする
- パターン2: proxyサーバをテスト用に起動しつつ、upstreamやlistenディレクティブなどconfigの一部を書き換える
- パターン3: ngx_mrubyやmod_mrubyで設定をmrubyで書き、[mruby-mtest](https://github.com/iij/mruby-mtest)などでテストコードを書く

パターン2はconfigの一部を書き換えるため、本当に本番の環境で正しく動作するかわからない問題がある。テストコードがnginxやApacheなど特定の実装に依存する。CIは回しやすい。
パターン3はngx_mrubyやmod_mrubyでconfigを書かなければならないため、今動いているproxyの設定をテストできない。

まず、今動いているproxyの設定を基にテストコードを書き、移行先のproxyをテストするのが望ましい。
今すぐCIを回したいわけではないためパターン1でいく。

## Perlでテストを書く

- テストを書くための学習コストをなるべく下げたい
  - Perlの標準モジュール Test::More と HTTP::Tiny だけでテストコードを書く
  - サーバに入ってるデフォルトのPerlで動くため、CPANモジュールのインストールがいらない (ただし、Perl 5.14以上)
    - サーバ上でテストを実行しやすいため、サービスイン前に自動でテスト実行するなどの仕組みにのせやすい
  - Rubyなど他の言語でもよい。ただし、RSpecのような学習コストがそれなりにあるフレームワークを使いたくなかった。PerlのTest::Moreは非常にシンプル

```perl
use Test::More;
use HTTP::Tiny;
subtest 'top' => sub {
    my $http = HTTP::Tiny->new->get('http://example.com/');
    is $res->{status}, 200;
};
```

- 本番投入前のproxyサーバにテストしたい
  - 環境変数 TEST_PROXY_HOST に接続先ホストを指定できるようにする。
  - HostヘッダはリクエストURLのHost部分そのまま
  - TCPのDestination HostがTEST_PROXY_HOSTに指定されたホスト

```perl
# 環境変数で接続ホストを書き換える
# 同様の動作をさせるためには、HTTP::Tiny 0.0.57 から peer オプションが使える。
# しかし、Perl 5.24でも標準で入っているHTTP::Tinyは0.056なため、peer オプションは使えない。
BEGIN {
    if (defined $ENV{TEST_PROXY_HOST}) {
        my $host = $ENV{TEST_PROXY_HOST};
        my $port = $host =~ s/:(\d*)\z// && length $1 ? $1 : undef;
        *IO::Socket::INET::new = sub {
            my $class = shift;
            unshift(@_, "PeerAddr") if @_ == 1;
            my %arg = @_;
            $arg{PeerHost} = $host;
            $arg{PeerPort} = $port if defined $port;
            return IO::Socket::new($class, %arg);
        };
        *IO::Socket::IP::new = sub {
            my $class = shift;
            my %arg = (@_ == 1) ? (PeerHost => $_[0]) : @_;
            $arg{PeerHost} = $host;
            $arg{PeerPort} = $port if defined $port;
            return IO::Socket::new($class, %arg);
        };
    } else {
        warn 'TEST_PROXY_HOST env is not defined.';
    }
}
```

## 今後

コンテナ化によるCIをしたい。今のところ、テストを実行すると本番のupstreamサーバにリクエストがとぶため、CIさせにくい。
閉じた環境でテスト実行するためには以下の要素が必要。

- :ok_woman: 環境変数でコンテナのhost:portを指定できる
- :no_good: upstreamサーバをコンテナに差し替える
  - nginxならば、envディレクティブとperl/luaモジュールにより、upstreamディレクティブの内容を環境変数で置き換えられそう
- :no_good: upstreamサーバのさらに後ろにデータの入ったDBを用意する 
  - テスト実行ごとに用意するのではなく、stagingのDBでよいかもしれない

### コンテナ化の例

[Docker と infrataster で nginx の振る舞いをテストする](http://heartbeats.jp/hbblog/2016/06/docker-infrataster-nginx.html]

- Docker networkでVPCと同じアドレス帯をつくる
- ELBをHAProxyで代替

## 参考

- [Test::Nginxでnginxモジュールのテストを自動化する](http://qiita.com/cubicdaiya/items/36e10ed35848919dc05c)
- [nginxのproxy設定ファイルも自動テストしよう](http://blog.shibayu36.org/entry/2014/04/09/075409)
- [Test::Apache::RewriteRules で mod_rewrite のテストを書こう](http://d.hatena.ne.jp/onishi/20101017/1287277579)
- [nginxをdockerで動かす時のTips 3選](http://heartbeats.jp/hbblog/2014/07/3-tips-for-nginx-on-docker.html)
- [100行あったmod_rewirteを ngx_mrubyで書き換えた話](https://speakerdeck.com/buty4649/100xing-atutamod-rewirtewo-ngx-mrubydeshu-kihuan-etahua)
- [Infratasterでリバースプロキシのテストをする](http://techlife.cookpad.com/entry/2014/11/19/151557)
- [Docker と infrataster で nginx の振る舞いをテストする](http://heartbeats.jp/hbblog/2016/06/docker-infrataster-nginx.html]
