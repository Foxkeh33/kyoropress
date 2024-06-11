# docker-wp-local
 docker-compose settings for local wordpress development enviroment.

 Dockerを利用したWordpressローカル開発環境用スタックです。

 ## 構築できるもの

* SSL対応、WP CLIインストール済みのwordpressコンテナ
* WORDMOVEでサーバーと同期※
* mailcatcherでwordpressのメールをテスト

※WORDMOVEは構築方法のみを案内しています。利用方法等は公式ドキュメントをご確認ください。作者はテンプレートのアップロードのみで利用しています。

 ## 利用条件

MacOS用の環境用です。

ローカルホストを任意のホスト名で利用したい場合は、hostsファイルへの設定が必要です。

wordmoveを利用する場合はsshで秘密鍵を利用した接続を想定しています。サーバーとのssh接続および、ローカルのssh/configへの設定をしているものと想定しています。

### docker & docker-compose

現在はDocker Desktopをインストールすると、パッケージに含まれている様子。

[Overview of installing Docker Compose](https://docs.docker.com/compose/install/)

### mkcert

ローカル環境でSSL認証を行うためにmkcertを使用するので、インストールしてください。

[mkcert](https://github.com/FiloSottile/mkcert)

## 利用方法

wordpressのみ->Aまで設定します。wordmove利用->A+Bまで設定します。mailcatcher->wordpress立ち上げ後に設定します

# Aパート Wordpress

### A-1 リポジトリをクローン

リポジトリをローカルの開発用ディレクトリにクローンしてください。

次にローカルで使うwordpressのURLを決めます。

例: local-wp.proto

### A-2 証明書の準備

最初にSSL用の証明書を作成します。certsフォルダでmkcertを利用し、証明書を作成します。

```
cd certs

mkcert localhost 127.0.0.1 local-wp.proto
```

certsフォルダ内に証明書が作成されます。下記のようになれば成功です。

```
Created a new certificate valid for the following names 📜
 - "localhost"
 - "127.0.0.1"
 - "local-wp.proto"

The certificate is at "./localhost+2.pem" and the key at "./localhost+2-key.pem" ✅

It will expire on 5 September 2026 🗓
```

localhost+2.pemとlocalhost+2-key.pemが作成されたので、.envファイルに設定を追加します。

```
# -------------------------------------------
# wordpress PROJECT_SETTING
# -------------------------------------------
# PROJECT NAME 接頭辞に使用
PROJECT_NAME=local-wp
# SUFFIX PROJECT_NAME.SUFFIXでホスト名を作成する
SUFFIX=proto
# TIMEZONE
TIMEZONE=Asia/Tokyo
# mkcertの証明書名(.より前)
CERT_NAME=localhost+2
```

wordpressのみの場合は、Bは不要なので、Cのビルドに進んでください。wordmoveはBも続けて設定します。

# Bパート WORDMOVE

### B-1 wordmoveの.env設定

.envファイルのMOVEファイルの部分を追加します。設定はご利用のサーバーに合わせて変更してください。

```
# -------------------------------------------
# WORDMOVE
# https://github.com/welaika/wordmove
# -------------------------------------------

# WordMoveのmovefile.ymlに設定する内容
# LOCAL_VHOST=https://local-wp.proto
LOCAL_VHOST=

STAGING_VHOST=
STAGING_WORDPRESS_PATH=
STAGING_DB_NAME=
STAGING_DB_USER=
STAGING_DB_PASSWORD=
STAGING_DB_HOST=
STAGING_SSH_HOST=
STAGING_SSH_USER=
STAGING_SSH_PORT=
STAGING_SSH_KEY=

# PRODUCTION_VHOST=
# PRODUCTION_WORDPRESS_PATH=
# PRODUCTION_DB_NAME=
# PRODUCTION_DB_USER=
# PRODUCTION_DB_PASSWORD=
# PRODUCTION_DB_HOST=
# PRODUCTION_SSH_HOST=
# PRODUCTION_SSH_USER=
# PRODUCTION_SSH_PORT=
# PRODUCTION_SSH_KEY=

# hostのsshエージェントを利用
SSH_AUTH_SOCK_APP=/run/host-services/ssh-auth.sock
```

### B-2 ssh接続の設定

wordmoveはコンテナ内部からサーバーに接続が必要です。
sshフォルダ内にある、configファイルに接続したいhostの情報を設定します。

### CONFIGサンプル
```
Host ****
  HostName 0.0.0.0
  Port *****
  User *****
  IdentityFile ~/.ssh/id_rsa
  TCPKeepAlive yes
  IdentitiesOnly yes

Host *
  IgnoreUnknown UseKeychain,AddKeysToAgent
  AddKeysToAgent yes
  UseKeychain yes
```


### id_rsa

sshが鍵方式の場合はsshフォルダ内のid_rsaに接続用の秘密鍵の情報をコピーしておきます。



# C ビルド

### Dockerコンテナの起動

docker-compose.ymlがある階層以下で以下のコマンドを実行します。

```
docker-compose build
```

続けて以下のコマンドを実行します。

```
docker-compose up -d
```

これでwordpressが立ち上がるので、ローカルのブラウザでアクセスして立ち上がっているか確認できます。SSLが設定されているかhttpsでアクセスしてください。

```
open https://localhost
```

### Wordpressの設定

ローカルホストのままで設定しても良いのですが、複数のプロジェクトを動かす場合にローカルでドメインを分けたほうが開発しやすいです。またwordmoveなどの際にも便利です。

/etc/hostsに.envで設定したローカルホストのドメインを追加します。

```
127.0.0.1  local-wp.proto
```

これでローカルホストのドメインでアクセスできるようになります。

```
open https://local-wp.proto
```

# WORDMOVEについて

本番とローカルサーバーの同期に便利ですが、誤ってpush --allしてしまうと、クラッシュすることが多いので、最近はテンプレートのpushしか使ってません。DBの同期もしっかりやりたい方は、しっかりと解説されているサイトなどをご確認ください。

[WordMove](https://github.com/welaika/wordmove)

また鍵の管理にご注意ください。レポジトリに誤ってpushしないように。

# MAILCATCHERについて

docker内のwordpressでcontactフォームなどでメールの送信をテストしたい場合に、MAILCATCHERが便利です。

使用するためには立ち上げたwordpressにSMTPを設定するプラグインを導入します。

[WP Mail SMTP](https://ja.wordpress.org/plugins/wp-mail-smtp/)

こちらのプラグインを利用してSMTPを設定します。

設定内容は以下の通りです。

```
SMTP ホスト: mailcatcher
SMTP ポート: 1025
認証: なし
```

プラグインのテストメール送信を利用します。

MailCatcherの画面を開きます。

```
open http://localhost:1080
```

鳥のマークのHTMLメールが届いていれば成功です。
あとはcontact-form7なでフォームからの送信などを試してください。

