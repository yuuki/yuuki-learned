DNS権威サーバ引っ越し
=====================

ここに書かれた内容の保証は一切しません。必ず、検証してから引っ越ししましょう。

## JPRSの推奨手順

- 手順 1：引っ越し先の権威DNSサーバの構築 
- 手順 2：引っ越し元の権威DNSサーバのゾーンデータの切り替え
- 手順 3：親に登録した委任情報の切り替え 
- 手順 4：双方の権威DNSサーバを並行運用
- 手順 5：引っ越し元の権威DNSサーバの停止 

### 参考文献
- https://jprs.jp/related-info/guide/019.pdf
- https://jprs.jp/tech/material/iw2011-lunch-L1-01.pdf
- 書籍 https://www.amazon.co.jp/dp/B00IJNDKDE/

## BINDドキュメントの手順

JPRSの手順との違いは、NSレコードセットに新旧の権威サーバを含んだ中間状態を作ること。Noteに追加操作と削除操作を同時にやるべきではないと書かれている。

http://www.opensource.apple.com/source/bind9/bind9-31/bind9/FAQ?txt

>
Q: How to change the nameservers for a zone?
>
A: Step 1: Ensure all nameservers, new and old, are serving the same zone
   content.
>
   Step 2: Work out the maximum TTL of the NS RRset in the parent and
   child zones. This is the time it will take caches to be clear of a
   particular version of the NS RRset. If you are just removing
   nameservers you can skip to Step 6.
>
   Step 3: Add new nameservers to the NS RRset for the zone and wait until
   all the servers for the zone are answering with this new NS RRset.
>
   Step 4: Inform the parent zone of the new NS RRset then wait for all
   the parent servers to be answering with the new NS RRset.
>
   Step 5: Wait for cache to be clear of the old NS RRset. See Step 2 for
   how long. If you are just adding nameservers you are done.
>
   Step 6: Remove any old nameservers from the zones NS RRset and wait for
   all the servers for the zone to be serving the new NS RRset.
>
   Step 7: Inform the parent zone of the new NS RRset then wait for all
   the parent servers to be answering with the new NS RRset.
>
   Step 8: Wait for cache to be clear of the old NS RRset. See Step 2 for
   how long.
>
   Step 9: Turn off the old nameservers or remove the zone entry from the
   configuration of the old nameservers.
>
   Step 10: Increment the serial number and wait for the change to be
   visible in all nameservers for the zone. This ensures that zone
   transfers are still working after the old servers are decommissioned.
>
   Note: the above procedure is designed to be transparent to dns clients.
   Decommissioning the old servers too early will result in some clients
   not being able to look up answers in the zone.
>
   Note: while it is possible to run the addition and removal stages
   together it is not recommended.


- ステップ1: 新旧の権威サーバのゾーン内容が全て一致しているかチェックせよ。
- ステップ2: 親と子の権威サーバのゾーンにあるNSレコードセットのうち、最大TTLをもつものを調べよ。それがNSレコードセットのキャッシュが切れるまでの時間である。
- ステップ3: 対象ゾーンのNSレコードセットに新しい権威サーバを追加せよ。対象ゾーンの全ての権威サーバが新しいNSレコードセット付きで応答するのを待て。
- ステップ4: 新しいNSレコードセットの親ゾーンを報告せよ。それから全ての親の権威サーバが新しいNSレコードセット付きで応答するのを待て。
- ステップ5: 古いNSレコードセットのキャッシュが切れるのを待て。待ち時間はステップ2をみよ。
- ステップ6: NSレコードセットから古い権威サーバを削除せよ。それから対象ゾーンの全ての権威サーバが新しいNSレコードを応答するのを待て。
- ステップ7: 新しいNSレコードセットの親ゾーンを報告せよ。それから全ての親の権威サーバが新しいNSレコードセット付きで応答するのを待て。
- ステップ8: 古いNSレコードセットキャッシュが切れるのを待て。待ち時間はステップ2をみよ。
- ステップ9: 古い権威サーバを止めて、古い権威サーバの設定からレコードエントリを全て削除せよ。
- ステップ10: シリアル番号をインクリメントして、全ての権威サーバでその変更が見えるようになるのを待て。古い権威サーバが退役した後に、ゾーン転送がまだ動いているかどうかを確認している。


