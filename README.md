
-------------------------------------
# プリザンター公式dockerの構築Tips
### ◆初回起動方法
`python(3) init.py`にて自動構築。この時.env等のパラメータは変更しないがポート番号が80であることを確認する。<br>
<details><summary>起動時の詳細について(220914)</summary><div>

環境ファイル”.env”は変更せずPython3 init.pyで先ず構築できることを確認。Postgreのポート5432が当たる場合は.env内容を一旦5432以外にしてinit.pyを実行。<br>
このときdotnetだけエラーを吐く状態で構築完了したらpls-baseコンテナに入って<br>
Linux版
`
Implem.Pleasanter/App_Data/Parameter/Rds.json
` 
<br>Windows版
`
C:¥inetpub¥wwwroot¥pleasanter¥App_Data¥Parameters
` 
<br>内にあるPostgresqlポートが先の.envに設定した値になっているので5432に変更する。
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
5432に変更後は
```
dotnet /web/pleasanter/Implem.CodeDefiner/Implem.CodeDefiner.NetCore.dll _rds
```
を実行しエラーが出ない事を確認。その後<br>
Linux版　(Windows版はIIS再起動)
`
systemctl daemon-reload && systemctl restart pleasanter
`
でサービスを再起動しWebにアクセス出来れば完了。<br>
Plesanter自体は80ポートを使用するようなのでapache2とか動いてたら止めないとpleasanterのポートを変更してても到達できない。<br>
</div></details>

### ◆初回アカウント設定
ログインID:  Administrator<br>
パスワード: pleasanter

### ◆[特権設定](https://pleasanter.org/manual/user-management-privileged-users)
操作手順<br> プリザンターが動作するサーバにログインします。<br> プリザンターのパラメータファイルが格納されているディレクトリ（\App_Data\Parameters）を開き「Security.json」をメモ帳などで開きます。"PrivilegedUsers"に、対象とするユーザのログインIDを配列形式で指定します。<br>

JSON<br> ` { "PrivilegedUsers": ["Administrator", "AdminUser1", "AdminUser2"] } ` <br> ファイルを保存してプリザンターを再起動します。<br><br>

<details><summary>◆起動評価結果<br></summary><div>
KHB07334用にコミットしたものをdockerhubに登録し、それに合わせてdocker-compose.ymlを修正。
(プロキシ内構築軽減とbuildで構築している場合時間が立つとVersionの整合性が合わなくなり起動に失敗するリスクを回避するため)<br>

→  〇  wsl(v2)単独環境上では起動確認。メッセージ通りのURLで起動可。<br>

→  〇  VirtualBox環境では起動Webはvirtualbox上Dockerの場合ゲストOSのIPである　http://10.0.2.15:8080/　(nginx.conf　にて設定確認) <br> 

→ ×  会社プロキシ＋Ubuntu18の環境ではスクリプト完了せず。<br>

→ ◎ (220914)37Ubuntu18
</div></details>

## ◆API連携(Python)
https://qiita.com/YoshikiSawada/items/17276185f9ed12d74ab9

## ◆その他のdockerイメージについて

### (1) docker(k-is-k/docker-pleasanter) こちらのほうが構築しやすいかも
https://github.com/k-is-k/docker-pleasanter

-----
### (2)プリザンターdocker（postgres対応／.netcore版）<br>
[以下は作者のReadmeの原文](https://github.com/twintee/pleasanter-docker)
<details><summary>展開</summary><div>
 
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
### ◆[開発者向け機能：拡張機能：拡張SQL：APIから拡張SQLを実行する](https://pleasanter.org/manual/extended-sql-api)
<details><summary>展開</summary><div>

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

### ◆[プリザンターから外部DBのテーブルを参照したい | Pleasanter](https://pleasanter.org/manual/faq-link-server))
<details><summary>展開</summary><div>

#### 概要

「[拡張SQL](https://pleasanter.org/manual/extended-sql)」の「OnSelectingColumn」を使用しプリザンターから外部DBのテーブルを参照するサンプルコードです。参照したい外部DBをSQL Serverの「リンクサーバー」機能を使って参照します。

#### 前提条件

1.  事前にリンクサーバの設定が完了している必要があります。

#### 説明

プリザンターから参照する外部DBのテーブル例です。
| id |氏名|年齢|
|--|--|--|
|1|ユーザー1|21|
1.  プリザンターのテーブルの分類Aに外部DBテーブルの id が格納されていることとします。
2.  外部DBテーブルの 年齢 をプリザンターの 分類Z に表示します。

#### 操作手順

##### テーブルの設定

1.  プリザンターを開き、テーブルを作成してください。
2.  「[テーブルの管理](https://pleasanter.org/manual/table-management)」を開き「[一覧](https://pleasanter.org/manual/table-management-grid)」タブと「[エディタ](https://pleasanter.org/manual/table-editor)」タブで「分類Z」を有効化してください。
3.  対象となるテーブルのサイトIDをメモしてください。

##### 拡張SQLの設定

1.  後述のJSONファイル例を LinkSever.json として保存してください
    
    -   保存先は以下の通りです。
    -   C:\web\pleasanter\Implem.Pleasanter\App_Data\Parameters\ExtendedSqls\LinkServer.json
2.  SiteIdList をメモしたサイトIDに変更してください。
    
3.  CommandText のSQLのFROM句: [リンクサーバー名].[DB名].[スキーマ名].[テーブル名] [照合順序]は設定したいリンクサーバーと、参照したいテーブル名に変更してください。
    
4.  プリザンターを再起動してください。
    

##### JSON(LinkServer.json)

```
{
    "Name": "LinkServerTest",
    "Description": "LinkServerTest",
    "SiteIdList": [xxx],
    "ColumnList": ["ClassZ"],
    "OnSelectingColumn": true,
    "CommandText": "(select age from [LINKSERVER].[postgres].[public].[staff] where id = Results.ClassA COLLATE Japanese_CI_AS)"
}
```
</div></details>

### ◆[親レコードに子レコードの合計値を転記する](https://implem.co.jp/2017/03/31/894/))
<details><summary>展開</summary><div>

今回は、親のチケットに子のチケットの数値をサマライズするサマリ機能について紹介します。<br>
#### 仕入の合計値を表示させよう
例えば、商談と仕入のような親子関係のデータがあります。この場合、一つの商談に対して複数の仕入れが発生します。その際に、商談では仕入の合計金額を管理したいですよね。それではやり方を見ていきましょう。<br>

[![仕入の合計値を表示させよう](https://implem.co.jp/img/archive/20170330_894_1-1.gif)](https://implem.co.jp/img/archive/20170330_894_1-1.gif)  

#### サマリ機能の設定方法
下記の図はサマリのイメージです。商談サイトと仕入サイトの2つのサイト間で親子関係が設定されています。<br>

[![サマリのイメージ](https://implem.co.jp/img/archive/1fb19bafa0370fb55332346ca247d16e-1.png)](https://implem.co.jp/img/archive/1fb19bafa0370fb55332346ca247d16e-1.png) 

サマリの設定は、子サイト側の仕入れで行います。設定は下記の動画のとおりです。<br>

[![サマリ機能の設定方法](https://implem.co.jp/img/archive/20170330_894_2.gif)](https://implem.co.jp/img/archive/20170330_894_2.gif)

#### サマリの種別
サマリの種別は下記の5種類です。業務の内容に応じて選択して下さい。<br>

|件数|チケットの件数|
|--|--|
|合計|サマリ項目で指定した数値フィールドの合計値|
|平均|サマリ項目で指定したの数値フィールドの平均値|
|最小|サマリ項目で指定したの数値フィールドの最小値|
|最大|サマリ項目で指定したの数値フィールドの最大値|
 
サマライズ機能は下記のような業務で活用できます。  

|業務の例|親|子|
|--|--|--|
|プロジェクト管理|WBS(工数合計)|工数(時間)|
|在庫管理|在庫(入庫合計)|入庫(個数)|
|ヘルプデスク|製品(問合せ件数)|問合せ|

 ##### 関連する記事
[リンク機能でタスクと課題を紐づけて管理](https://implem.co.jp/2017/03/30/848/)
</div></details>
