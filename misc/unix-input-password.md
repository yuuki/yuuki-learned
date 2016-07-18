UNIXのパスワード入力
=====================

- 入力したパスワードを端末に表示させないように、エコーバッグする必要がある。
- パスワードを標準入力経由で読み出すか、`/dev/tty` から読みだすのか。

## bash のビルドイン関数 read 
- -p prompt プロンプトを表示
- -s サイレントモード。入力をエコーバックさせないように。

```
read -s -p "Password: " password
```

## stty


```bash
#!/bin/bash
stty -echo
printf "Password: "
read PASSWORD
stty echo
printf "\n"
```

## Python の例

https://docs.python.org/3/library/getpass.html

まず`/dev/tty`をオープンし、失敗するとstdin、stderrにフォールバックするとのこと。

Linuxプログラミングインタフェースによると、`/dev/tty`のオープンに成功するのは、プロセスが制御端末を持つ場合である。標準入力や標準出力がリダイレクトされていても有効な方法で、制御端末に対する確実なI/Oが可能。

## 参考

- [read - read a line from standard input](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/read.html)
- [How to get a password from a shell script without echoing](http://stackoverflow.com/questions/3980668/how-to-get-a-password-from-a-shell-script-without-echoing)
