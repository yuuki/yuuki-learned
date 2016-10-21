whisper (Graphite)
==================

## 自動丸め処理

- file_update関数 1 datapoint書き込む https://github.com/graphite-project/whisper/blob/b7dbc7c/whisper.py#L525
- 最も高精度なarchiveに書き込む https://github.com/graphite-project/whisper/blob/b7dbc7c/whisper.py#L545-L562
  - 切りの良いインターバルを計算  https://github.com/graphite-project/whisper/blob/b7dbc7c/whisper.py#L546
  - 書き込む領域へオフセット計算してseekする。 https://github.com/graphite-project/whisper/blob/b7dbc7c/whisper.py#L552-L562
- 精度の高いarchive順に丸めた値を書き込む https://github.com/graphite-project/whisper/blob/b7dbc7c/whisper.py#L564-L569
- __propagate関数: 自動丸めする https://github.com/graphite-project/whisper/blob/b7dbc7c/whisper.py#L432
  - 一つ上の精度のarchiveもhigherとして渡しているところに注意
  - 引数で渡したtimestamp近傍のdatapointをhigher領域から読み出して、。
  - 近傍の値のリストを取得 https://github.com/graphite-project/whisper/blob/b7dbc7c/whisper.py#L477-L481
    - https://github.com/graphite-project/whisper/blob/b7dbc7c/whisper.py#L479 ここはなぜ == でうまく動くのかまだわかってない。neighborValues
  - 近傍値がなかったら処理を終了。低精度archiveの丸め処理もしない https://github.com/graphite-project/whisper/blob/b7dbc7c/whisper.py#L483-L486
  - xffはユーザが設定する。近傍値リストの各要素が一定割合存在していないと丸めずにNoneのままにする。丸める前の値が1個しかないときの場合はNoneにしたいとかいうときに使う。 https://github.com/graphite-project/whisper/blob/b7dbc7c/whisper.py#L489
  - ここで丸めた値を計算 https://github.com/graphite-project/whisper/blob/b7dbc7c/whisper.py#L490

