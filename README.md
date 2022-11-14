
-------------------------------------
# プリザンター公式dockerの構築Tips
## ◆起動時Tips<br>
KHB07334用にコミットしたものをdockerhubに登録し、それに合わせてdocker-compose.ymlを修正。
(プロキシ内構築軽減とbuildで構築している場合時間が立つとVersionの整合性が合わなくなり起動に失敗するリスクを回避するため)<br>

→  〇  wsl(v2)単独環境上では起動確認。メッセージ通りのURLで起動可。<br>

→  〇  VirtualBox環境では起動Webはvirtualbox上Dockerの場合ゲストOSのIPである　http://10.0.2.15:8080/　(nginx.conf　にて設定確認) <br> 

→ ×  会社プロキシ＋Ubuntu18の環境ではスクリプト完了せず。<br>

→ ◎ (220914)37Ubuntu18
<details><summary>(220914)37Ubuntu18でやった事</summary><div>

環境ファイル”.env”は変更せずPython3 init.pyで先ず構築できることを確認。Postgreのポート5432が当たる場合は.env内容を一旦5432以外にしてinit.pyを実行。
構築時にdotnetだけエラーを吐く状態で構築完了したらpls-baseコンテナに入って
Linux版
`
Implem.Pleasanter/App_Data/Parameter/Rds.json
` 
Windows版
`
C:¥inetpub¥wwwroot¥pleasanter¥App_Data¥Parameters
` 

にあるPostgresqlポートが先の.envに設定した値になっているので5432に変更する。
```
 {
        "Dbms": "PostgreSQL",
        "Provider": "Local",
        "TimeZoneInfo": "Tokyo Standard Time",
        "SaConnectionString":"Server=postgres-db;Port=5432;Database=postgres;UID=postgres;PWD=mypass-abc",
        "OwnerConnectionString":"Server=postgres-db;Port=5432;Database=#ServiceName#;UID=#ServiceName#_Owner;PWD=SetAdminsPWD","UserCon
nectionString":"Server=postgres-db;Port=5432;Database=#ServiceName#;UID=#ServiceName#_User;PWD=SetUsersPWD",
        "SqlCommandTimeOut": 600,
        "MinimumTime": 3,
        "DeadlockRetryCount": 4,
        "DeadlockRetryInterval": 1000,
        "DisableIndexChangeDetection": false
 }
```
変更後
```
dotnet /web/pleasanter/Implem.CodeDefiner/Implem.CodeDefiner.NetCore.dll _rds
```
を実行しエラーが出ない事を確認。その後<br>
Linux版　(Windows版はIIS再起動)
`
systemctl daemon-reload && systemctl restart pleasanter
`
でサービスを再起動しWebにアクセス出来れば完了。<br>
ホストサーバの80ポートを使用するようなのでapache2とか動いてたら止めないとpleasanterのポートを変更してても到達できない。<br>
</div></details>

### ◆初回アカウント設定
ログインID:  Administrator<br>
パスワード: pleasanter

### ◆[特権設定](https://pleasanter.org/manual/user-management-privileged-users)

操作手順<br> プリザンターが動作するサーバにログインします。<br> プリザンターのパラメータファイルが格納されているディレクトリ（\App_Data\Parameters）を開き「Security.json」をメモ帳などで開きます。"PrivilegedUsers"に、対象とするユーザのログインIDを配列形式で指定します。<br>

JSON<br> ` { "PrivilegedUsers": ["Administrator", "AdminUser1", "AdminUser2"] } ` <br> ファイルを保存してプリザンターを再起動します。<br><br>

## ◆dockerイメージについて

### ◆docker(k-is-k/docker-pleasanter) こちらのほうが構築しやすいかも
https://github.com/k-is-k/docker-pleasanter

-----
### ◆公式プリザンターdocker（postgres対応／.netcore版）<br>
[以下は作者のReadmeの原文](https://github.com/twintee/pleasanter-docker)
<details><summary>  内容</summary><div>
 
Postgres対応版.netcoreプリザンターのdocker構築用Dockerfileとdocker-compose.ymlその他諸々。  
コマンドで動くようにしたかったのとDockerfileに可能な限り詰め込んで作業を軽減してみた。  
公式プリザンターzipの内容で動作変わるのが嫌だったのでリポジトリに含めたのがアレ。  

#### 参考にしたページ
https://qiita.com/ta24toy27/items/986b3057e08f3da2fc06

#### 検証済み環境
- ubuntu :18.*

#### 準備

- `./.env`を編集して設定変更
 ```
    - DISTRIBUTION: pleasanterコンテナのベースOS(cent -> centos7, deb -> buster-slim)
    - TZ: タイムゾーン
    - POSTGRES_PORT: postgresqlの公開ポート
    - POSTGRES_PASSWORD: postgresqlユーザーのパスワード
    - POSTGRES_MEM: postgresql使用メモリ
    - PLS_PORT: プリザンター用ポート=8080
    - PLS_MEM: プリザンター使用メモリ
```
- pythonコマンド実行
`python3 ./init.py`
    1. `initialize container? (y/*):` 処理開始なら `y[enter]`
    1. `initialize volumes? (y/*):` コンテナ削除時にボリューム削除するなら `y[enter]`
    1. `rebuild image? (y/*) :` docker imageの再構成したいときは `y[enter]`  
※5分程度かかる
    1. `make container? (y/*) :` コンテナを作成するなら `y[enter]`  
- 正常終了時の出力にローカルIP込みのURLが含まれるが、osにより取得されるipが変わる？為参考までに
</div></details>

## ◆利用TIPS
### ◆開発者向け機能：拡張機能：拡張SQL：APIから拡張SQLを実行する　[リンク先](https://pleasanter.org/manual/extended-sql-api)
<details><summary>  内容</summary><div>

#### 概要
「[API](https://pleasanter.org/manual/api)」と「[拡張SQL](https://pleasanter.org/manual/extended-sql)」を組み合わせてデータベースから直接データを取得したり更新したりすることができます。

#### 注意事項
1.  誤って使用するとプリザンターが利用できなくなったり、データが壊れたりする可能性がありますので、十分なテストを行った上でご利用ください。
2.  APIキーやセッションによるログイン確認は行いますが、テーブルへの権限チェックなどが行われないため、SQLの中で実施する必要があります。
#### 制限事項
1.  拡張SQLのJSONファイルやSQLファイルを更新した後は「アプリケーションを再起動」するまで反映しません。
2.  セキュリティ上の理由により拡張SQLはWeb画面から設定できません。

#### 事前準備
APIの操作を行う前に[APIキーの作成](https://pleasanter.org/manual/api-key)を実施してください。
.jsonファイルにて以下の2項目は必須項目となります。
|パラメータ名 |値の例 |説明 |
|--|--|--|
|Name|例) Sample|APIから実行させる際の名前を設定します。|
|Api|例) true|trueの場合APIからの実行を許可します。|

CommandTextに直接SQL文を記載できますが、SQL文が長文になる場合など、ファイルを分けることが可能です。 その場合のファイル名は以下の画像のように、.jsonファイルと同名.sqlとしてください。  
例)  
APItoSQL.json　←定義ファイル  
APItoSQL.json.sql　←SQL文

![image](https://pleasanter.org/binaries/2d01585929d44f339937c9873ba5293e)

#### URL
下記のURLを使用します。 HTTPメソッドはPOSTです。  
http://{servername}/pleasanter/api/extended/sql  
"http://{servername}/pleasanter" の部分は、適宜、環境に合わせて編集してください。

#### リクエスト
下記例のようにParamsに入れたパラメータは@RefIDのように拡張SQLに渡すパラメータとして使用することができます。 標準で用意されている@_Uなどのパラメータも使用できます。

```json
{
    "ApiVersion": 1.1,
    "ApiKey": "XXXXXXXXXX...",
    "Name": "Sample",
     "Params": {
        "RefID": 1
    }
}
```

#### JSONファイル

APItoSQL.json

```json
{
    "Name": "Sample",
    "Api": true
}
```

#### SQL文(外部ファイル)

APItoSQL.json.sql

```sql
SELECT [ReferenceId]
      ,[DeptId]
      ,[GroupId]
      ,[UserId]
      ,[Ver]
      ,[PermissionType]
  FROM [Implem.Pleasanter].[dbo].[Permissions]
  WHERE [ReferenceId]=@RefID

```

#### レスポンス

テーブル内にはレコードの配列があり、レコード内はKey, ValueのHashでカラム名と値が入ってきます。

```json
{
    "StatusCode": 200,
    "Response": {
        "Data": {
            "Table": [
                {
                    "ReferenceId": 1,
                    "DeptId": 0,
                    "GroupId": 0,
                    "UserId": 1,
                    "Ver": 1,
                    "PermissionType": 511
                },
                {
                    "ReferenceId": 1,
                    "DeptId": 0,
                    "GroupId": 0,
                    "UserId": 9,
                    "Ver": 1,
                    "PermissionType": 31
                },
                {
                    "ReferenceId": 1,
                    "DeptId": 0,
                    "GroupId": 0,
                    "UserId": 10,
                    "Ver": 1,
                    "PermissionType": 31
                },
                {
                    "ReferenceId": 1,
                    "DeptId": 0,
                    "GroupId": 1,
                    "UserId": 0,
                    "Ver": 1,
                    "PermissionType": 511
                }
            ]
        }
    }
}
```
複数のテーブルから値を取得した場合は以下のようなレスポンスとなります。  
Response.Data.Table （1つめのテーブル）  
Response.Data.Table1 （2つめのテーブル）  
Response.Data.Table2 （3つめのテーブル）
</div>
</details>
