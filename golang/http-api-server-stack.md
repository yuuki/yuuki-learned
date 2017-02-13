GoでHTTP APIサーバ書くときのスタック
====================================

最近書いているJSON APIサーバはこういう感じ。

## リポジトリストラクチャ

[Go best practices, six years in](https://peter.bourgon.org/go-best-practices-2016/#repository-structure) を参考に。
個人的には、`pkg` ではなく、`lib` を使っている。

## HTTPサーバ

WAFを使わずに、`net/http` ベースでハンドラを書く。
[negroni](https://github.com/urfave/negroni)は、PerlのPlack::Middlewareのような感じで、`net/http` をラップしてくれる。

- panicをキャッチして、ISEにしてくれる Recovery https://github.com/urfave/negroni#recovery
- アクセスログを吐いてくれる Logger https://github.com/urfave/negroni#logger
- その他いろいろ https://github.com/urfave/negroni#third-party-middleware

## Graceful Restart

- Shutdown は Go 1.8 でもうじき入るのでそれを使う
- Restartは Server::Starter と https://github.com/lestrrat/go-server-starter-listener

## エラー処理

### custom error type 

https://blog.golang.org/error-handling-and-go に書かれているように、SyntaxErrorのようにエラー型を独自定義して、つくるのがよい。
呼び出し元で、type assertionにより、どのエラーが発生したかを区別できる。

### pkg/errors 

errorsの代わりに[pkg/errors](https://github.com/pkg/errors)を使う。通常のerrorsと違って、コールスタックを持たせることができる。

- エラーの発生は `errors.New` や `errors.Errorf`
- 呼び出し元では、`errors.WithStack` か `errors.Wrapf` で伝搬させる
- 上位層で `errors.Cause` で根源のエラーを取り出し、type assertionなどでエラーに応じて分岐する

参考: [Golangのエラー処理とpkg/errors](http://deeeet.com/writing/2016/04/25/go-pkg-errors/)

## テスト

### 標準パッケージ

極力、testingパッケージをそのまま使う。テストフレームワークの学習コストを避けるためと、役に立たないエラーメッセージを出さないようにするため。
補助的に [github.com/kylelemons/godebug/pretty](https://github.com/kylelemons/godebug) を使うと、オブジェクト同士のdiffをきれいに表示できる。

### Table Driven Tests

### データベースのモックテスト

interfaceを使う。
[Golangにおけるinterfaceをつかったテストで Mock を書く技法](http://haya14busa.com/golang-how-to-write-mock-of-interface-for-testing/)

参考: [Advanced Testing with Go](https://speakerdeck.com/mitchellh/advanced-testing-with-go)

## DB接続など永続化したオブジェクトのハンドリング


## vendoring

よく使われている [glide](https://glide.sh/)を使っている。[dep](https://github.com/golang/dep) はまだ様子見。

## Makefile
