
-------------------------------------
# プリザンター公式dockerの構築Tips
## ■起動時Tips<br>
KHB07334用にコミットしたものをdockerhubに登録し、それに合わせてdocker-compose.ymlを修正。
(プロキシ内構築軽減とbuildで構築している場合時間が立つとVersionの整合性が合わなくなり起動に失敗するリスクを回避するため)<br>

→  〇  wsl(v2)単独環境上では起動確認。メッセージ通りのURLで起動可。<br>

→  〇  VirtualBox環境では起動Webはvirtualbox上Dockerの場合ゲストOSのIPである　http://10.0.2.15:8080/　(nginx.conf　にて設定確認) <br> 

→  ×  会社プロキシ＋Ubuntu18の環境ではスクリプト完了せず。<br>

→  ◎(220914)37Ubuntu18で
###  ◎でやった事。
.envは変更せずPython3 init.pyで構築。Postgreのポート5432が当たる場合は.env内容を一旦5432以外にしてinit.pyを実行。
構築時にdotnetだけエラーを吐く状態で構築完了したらpls-baseコンテナに入って
`
Implem.Pleasanter/App_Data/Parameter/Rds.json
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
```
systemctl daemon-reload && systemctl restart pleasanter
```
でサービスを再起動しWebにアクセス出来れば完了。<br>
ホストサーバの80ポートを使用するようなのでapache2とか動いてたら止めないとpleasanterのポートを変更してても到達できない。<br>

### ■初回アカウント設定
ログインID:  Administrator
パスワード: pleasanter

### ■[特権設定](https://pleasanter.org/manual/user-management-privileged-users)

操作手順<br> プリザンターが動作するサーバにログインします。<br> プリザンターのパラメータファイルが格納されているディレクトリ（\App_Data\Parameters）を開き「Security.json」をメモ帳などで開きます。"PrivilegedUsers"に、対象とするユーザのログインIDを配列形式で指定します。<br>

JSON<br> ` { "PrivilegedUsers": ["Administrator", "AdminUser1", "AdminUser2"] } ` <br> ファイルを保存してプリザンターを再起動します。<br><br>

## ■dockerイメージについて

### ■docker(k-is-k/docker-pleasanter) こちらのほうが構築しやすいかも
https://github.com/k-is-k/docker-pleasanter

-----
### ■公式プリザンターdocker（postgres対応／.netcore版）<br>
[以下は作者のReadmeの原文](https://github.com/twintee/pleasanter-docker)

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
