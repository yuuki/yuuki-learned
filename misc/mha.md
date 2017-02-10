# MHA for MySQL

## 拡張ポイント

- secondary_check_script
  - マスタがダウンしたかどうかを複数のネットワーク経路から判定
  - マスタに接続エラーになったときに呼ばれる
- shutdown_script
  - 電源強制OFF
- master_ip_failover_script
  - フェイルオーバの直前と、新マスタ移行時に呼ばれる
- master_ip_online_change_script
  - master_ip_failover_scriptと似ている。スイッチオーバ時に呼ばれる
  - 現マスタのgraceful shutdownを行う
- report_script
  - フェイルオーバの終了後の呼ばれる
- init_conf_load_script

## Xen環境で強制電源OFF

旧マスタにSSHで接続できない場合、旧マスタのDom0にSSHして `xm destroy $OLD_MASTER` 
Dom0にmhaユーザを作成し、mhaユーザで `/usr/local/bin/xm_destroy` をNOPASSWD許可する。

## VIP付け替え

- `ip addr add 192.168.0.2/24 dev eth0`

## 検証

- manager down時の挙動
- node down時の挙動
- バージョンアップ方法
- フェイルオーバ時に `no_master=1` を指定したノードのポジションが最も進んでいる場合

## References

- [mysql-master-ha 公式ドキュメント](https://code.google.com/p/mysql-master-ha/)
- [MHA for MySQL の概要](http://mizzy.org/blog/2013/02/06/1/)
- [](http://lamanotrama.hateblo.jp/entry/20141218)
