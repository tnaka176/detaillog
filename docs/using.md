# 使い方

## 分析データの準備

RADIUS detailログが必要です。


## 分析データの登録

fluentdコンテナ内`/fluentd/logs/detail`に追記してください。<br />
detailを0バイトにしてコピーしても良いです。<br />
ファイルポインタがファイル末尾を記憶しているため、上書きでは取りこぼしが発生します。<br />

```
# docker compose exec fluentd cp /dev/null /fluentd/logs/detail
# docker compose cp detail fluentd:/tmp
# docker compose exec fluentd sh
(fluentd) $ cat /tmp/detail >> /fluentd/logs/detail
(fluentd) $ exit
```

同じデータは複数回登録しないでください。<br />
（2重チェックは行っていないため、2重データとなります）

追記後データが登録されますが、反映に時間がかかる場合があります。

登録の状況は右上の三から Analytics - Discover

右上の期間を目的の期間（昨日ならLast 2 days）とします。<br />
（この期間は日本時間で指定してください）

「更新」をクリックの都度更新されてゆきます。


## グラフ作成

データ登録が完了したら分析グラフを作成します。

右上の三から Analytics - Dashboard - ビジュアライゼーションを作成<br />
（初回アクセス時は「新規ダッシュボードを作成」）

対象期間を右上で指定してください。

レポート対象物を左ペインから中央ペインへドラッグ＆ドロップしてください。

下部の提案からグラフ形状を選択します。

候補数は右側ペインのトップの値から変更できます。

不要なもの（User-Name=abcdefg等を除外したい場合は、上部の検索欄に入力します。<br />
例）User-Name=abcdefgを除外したい場合
```
not User-Name : "abcdefg"
```

KQLを使用しています。入力フォーマットはgoogle検索等を参考にしてください。


## データのクリア

インデックスライフサイクルポリシーに従い自動的に消去されてゆきます。<br />
（インデックスを作成した日から30日後）

直ちに消去したい場合は、左上の三から「スタック管理」 - インデックス管理<br />
該当日付のものを選択(check)し、*個のインデックスの管理 - インデックス削除<br />
確認画面が出るので再度 インデックス削除
