github.com/go-sql-driver/mysql
==============================

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

```

```

## 関連

