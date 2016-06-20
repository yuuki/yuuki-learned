Perlで外部コマンド実行
======================

- 1. 組み込み関数の `system(@cmd)` を使う
- 2. \`$cmd\` を使う
- 3. 標準モジュールの [IPC::Open3](https://metacpan.org/pod/IPC::Open3)
- 4. [Capture::Tiny](https://metacpan.org/pod/Capture::Tiny) を使う

## system

[perldoc system](http://perldoc.jp/func/system)

`system(@cmd)` はexit codeのみ取得できる。

```perl
$ret = system(@cmd) >> 8;
```

## `$cmd`

シェルコマンドとして実行される。\`$cmd\`はコマンドの標準出力を得る。標準エラー出力は以下のようにして得る。

```perl
$output = `cmd 2>&1`;
```

## IPC::Open3

標準モジュールのみで、実行するコマンドの標準出力と標準エラー出力、exit codeを取得するならこれを使う。
[IPC::Open3 の正しい使い方](http://d.hatena.ne.jp/kazuhooku/20100813/1281690025)

```perl
sub run_cmd {
    my @cmd = @_;

    my($wtr, $rdr);
    my $pid = IPC::Open3::open3($wtr, $rdr, 0, @cmd);
    close $wtr;
    my @out = <$rdr>;
    waitpid($pid, 0);
    close $rdr;

    my $ret = $? >> 8;

    return $ret, \@out;
}
```

ただし、このコードでは、標準出力と標準エラー出力を分けられない。

## Capture::Tiny

CPANモジュール使うならこれ。


