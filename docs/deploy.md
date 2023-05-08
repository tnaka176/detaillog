# デプロイ・初期設定

## manifestの設置

任意の場所にcloneしてdocker-compose/.envを作成し、build&run

```
git clone 
cd detaillog/docker-compose
cp env.sample .env
docker compose build
docker compose up -d
```

### 環境変数

|環境変数|説明|
|--------|----|
|TZ|timezone|
|ELASTICSEARCH_VERSION|elasticsearch version(docker image tag)|
|KIBANA_VERSION|kibana version(docker image tag)|
|FLUENTD_PLUGINS|fluentd plugins (space 区切り)|
|FLUENTD_DIR|fuluentd dir(in container)|
|KIBANA_EXTERNAL_PORT|listen kibana port|
|KIBANA_SERVER_PUBLICBASEURL|kibana URL|


## 初期設定(kibana)

kibanaへアクセスし、画面右下「スタック管理」へ移動します。

### スタック管理

#### 管理 - kibana - 高度な設定

- 一般  
  - フォーマットロケール：Japanese

- 使用データ（最下部にあります）  
  - 使用状況データを提供：オフ

- 変更を保存

#### 管理 - データ - インデックスライフサイクルポリシー

##### ポリシーを作成

- ポリシー名 : detaillog-ILM

- ホットフェーズ - 高度な設定
  - ロールオーバー
    - 推奨のデフォルト値を使用 : オフ
    - ロールオーバーを有効にする : オフ
  - インデックスの優先順位
    - インデックスの優先度を設定 : オン
    - インデックスの優先順位 : 100

- ウォームフェーズ : オン : 7日

- ウォームフェーズ - 高度な設定
  - インデックスの優先順位
    - インデックスの優先度を設定 : オン
    - インデックスの優先順位 : 50
  - データを永久にこのフェーズで保持します
    - ゴミ箱マークをクリック　⇒　削除フェーズが表示される

- 削除フェーズ : 30日

- ポリシーを保存


#### 管理 - インデックス管理 - インデックステンプレート

##### テンプレートを作成

- ロジスティクス

|項目|設定値|
|----|------|
|名前|detaillog|
|インデックスパターン|radius.detail*|
|データストリーム|なし|
|優先度|なし|
|バージョン|なし|
|_meta field|なし|

- コンポーネントテンプレート : なし

- インデックス設定  
以下をコピペ  
```json
{
  "index": {
    "lifecycle": {
      "name": "detaillog-ILM",
      "rollover_alias": "detaillog-index"
    },
    "refresh_interval": "1s",
    "number_of_shards": "1",
    "number_of_replicas": "0"
  }
}
```

- マッピング - マッピングフィールド  
以下の様にマッピングする

|フィールド名|フィールド型|
|------------|------------|
|@timestamp|日付|
|NAS-IP-Address|IP|
|Framed-IP-Address|IP|

または以下のjsonを読み込み
```
{
  "properties": {
    "@timestamp": {
      "type": "date"
    },
    "Framed-IP-Address": {
      "type": "ip"
    },
    "NAS-IP-Address": {
      "type": "ip"
    }
  }
}
```

- マッピング - ランタイムフィールド : なし

- マッピング - 動的テンプレート : なし

- マッピング - 高度なオプション : なし（初期値のまま変更しない）

- エイリアス : なし


### ダミーデータ登録

インデックスパターンを作成したいが、データ無しでは作成できない。<br />
ダミーデータを登録しインデックスパターンを作成する。<br />
以下のデータを`fluentd:/fluentd/logs/detail`にアップロードする。<br />
※末尾には空行を入れること<br />
※2行目以降は先頭にTABを入れること
```
Sat Apr  1 00:00:00 2023
	Acct-Session-Id = "01234567"
	Framed-Protocol = PPP
	User-Name = "abcdefg"
	Cisco-AVPair = "connect-progress=Call Up"
	Acct-Authentic = RADIUS
	Acct-Status-Type = Start
	NAS-Port-Type = Virtual
	NAS-Port = 0
	NAS-Port-Id = "0/0/0/000"
	Connect-Info = "Connect-Info"
	Cisco-AVPair = "client-mac-address=0123.4567.89ab"
	Service-Type = Framed-User
	NAS-IP-Address = 123.123.123.123
	Acct-Delay-Time = 0
	Acct-Unique-Session-Id = "0123456789abcdef"
	Timestamp = 1680307200

```

- スタック管理 - データ - インデックス管理  
日付毎にインデックスが作成されます。<br />
※インデックス名の日付部分はUTC日にちで作成。（日本時間-9時間）


### インデックスパターンの作成

- スタック管理 - Kibana - データビュー - データビューを作成

|項目|設定値|
|----|------|
|名前|detail|
|インデックスパターン|radius.detail*|
|タイムスタンプフィールド|@timestamp|

データビューをKibanaに保存


### ダミーデータの削除

- スタック管理 - インデックス管理  
該当インデックスを選択(check)し、1個のインデックスの管理 - インデックス削除  
確認画面が出るので再度 インデックス削除

以上でデプロイ、初期設定は完了です。
