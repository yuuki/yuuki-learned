# MySQL プロトコル

## パケットフォーマット

- サーバ応答パケットは4つのカテゴリに分類される。
  - データパケット
  - データストリームの終端パケット
  - 成功レポート(OK)パケット
  - エラーメッセージパケット
- すべてのパケットは4バイトヘッダをもつ
  - 先頭3バイトはパケットボディの長さ
  - 残り1バイトはパケットシーケンス番号

## ハンドシェイク認証

- 1. サーバ => クライアント: グリーティングパケット
- 2. クライアント => サーバ: 資格情報パケット
- 3. サーバ => クライアント: 応答パケット
- (4. クライアント => サーバ: old passwordを使っている場合はSHA1ハッシュでなくXORハッシュによる古い形式のパスワードを送る)
- (5. サーバ => クライアント: 応答パケット)

## コマンドパケット

## サーバ応答

## 参考資料

- [MySQL Internals Manual Chapter 14 MySQL Client/Server Protocol](https://dev.mysql.com/doc/internals/en/client-server-protocol.html)
- [詳解 MySQL](https://www.oreilly.co.jp/books/9784873113432/) 
- [MySQLのプロトコル解説](http://slide.rabbit-shocker.org/authors/tommy/mysql-protocol/)
- [MySQLのプロトコルを学ぶ](http://d.hatena.ne.jp/ASnoKaze/20141227/1419697189)
