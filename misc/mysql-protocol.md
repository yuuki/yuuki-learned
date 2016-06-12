MySQL プロトコル
================

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

## github.com/go-sql-driver/mysql

- database/sqlのMySQLドライバ GoのMySQLプロトコルの実装のひとつ
- [MySQL プロトコル](../mysql/protocol.md)

```mysql
  // サーバからのグリーティングパケットを受け取って解析する
	// Reading Handshake Initialization Packet
	cipher, err := mc.readInitPacket()
	if err != nil {
		mc.cleanup()
		return nil, err
	}

  // 資格情報をサーバに送信する
	// Send Client Authentication Packet
	if err = mc.writeAuthPacket(cipher); err != nil {
		mc.cleanup()
		return nil, err
	}

  // 資格情報についての応答パケットの処理
	// Handle response to auth packet, switch methods if possible
	if err = handleAuthResult(mc, cipher); err != nil {
		// Authentication failed and MySQL has already closed the connection
		// (https://dev.mysql.com/doc/internals/en/authentication-fails.html).
		// Do not send COM_QUIT, just cleanup and return the error.
		mc.cleanup()
		return nil, err
	}

  // max_allowed_packet を最初に
	// Get max allowed packet size
	maxap, err := mc.getSystemVar("max_allowed_packet")
	if err != nil {
		mc.Close()
		return nil, err
	}
	mc.maxPacketAllowed = stringToInt(maxap) - 1
	if mc.maxPacketAllowed < maxPacketSize {
		mc.maxWriteSize = mc.maxPacketAllowed
	}
```

