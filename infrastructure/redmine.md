# Redmineの運用について
## はじめに
Redmineは、herokuでホスティングされ運用されています。
Redmineやそれに付随するソースコードは、Githubで管理されています。
このドキュメントでは、Redmineの運用についての情報を説明します。

## 運用するために必要な環境
Redmineを運用するためには、最低限下記の環境が必要になります。

* git
* heroku-toolbelt
* Dropbox

ログインに必要なユーザ名やパスワードなどは、このドキュメントには記載しません。
それらの情報は、運用担当者から直接聞いてください。

## ホスティング環境
heroku上のアプリとしてプロダクション環境とステージング環境を用意しています。

| 環境名               | アプリ名          | 説明                                                              |
|----------------------|-------------------|-------------------------------------------------------------------|
| プロダクション環境   | ugp-rdm           | 本運用として使用されるアプリです。                                |
| ステージング環境     | ugp-rdm-staging   | プロダクション環境に配信する前に、この環境で動作確認を行います。  |

### デプロイ手順
1. ソースコードのクローン

    下記のコマンドを実行します。

    ```bash
    $ git clone https://github.com/ugp-2014/redmine
    $ cd redmine
    ```

1. herokuのリモートリポジトリの設定

    下記のコマンドを実行します。

    ```bash
    $ git remote add staging git@heroku.com:ugp-rdm.git
    $ git remote add production git@heroku.com:ugp-rdm-staging.git
    ```

1. herokuへのプッシュ

    下記のコマンドを実行します。

    ```bash
    $ git push HEROKU_REMOTE_REPOSITORY master
    ```

    `HEROKU_REMOTE_REPOSITORY`には、プロダクション環境(production)かステージング環境(staging)のリモートリポジトリ名を指定します。

## データベース環境
Redmineで使用するデータベースとしてHeroku Postgresを使用しています。

### データベースのエクスポート手順
1. データベースのキャプチャー 

    下記のコマンドを実行します。

    ```bash
    $ heroku pgbackups:capture --app=HEROKU_APP_NAME
    $ heroku pgbackups --app=HEROKU_APP_NAME
    ```

    `HEROKU_APP_NAME`には、プロダクション環境かステージング環境のアプリ名を指定します。

    上記の結果でバックアップされたIDをメモします。

1. ダンプファイルのダウンロード

    下記のコマンドを実行します。

    ```bash
    $ heroku pgbackups:url BACKUP_ID --app=HEROKU_APP_NAME
    ```

    上記の結果のURLを任意の方法でローカルにダウンロードします。

### データベースのインポート手順
1. ダンプファイルのアップロード

    Dropboxにダンプファイルを配置して、リンクを共有します。
    そのリンクに対して下記のように変換してメモします。

    ```text
    https://www.dropbox.com/s/XXXXXXXXXXXXX/Staging_20141123.dump?dl=0
    ↓
    https://dl.dropboxusercontent.com/s/XXXXXXXXXXXXX/Staging_20141123.dump
    ```

1. データベースのリストア

    下記のコマンドを実行します。

    ```bash
    $ heroku pgbackups:restore DATABASE_URL 'DROPBOX_URL' --app=HEROKU_APP_NAME
    ```

    `DROPBOX_URL`には、前回の手順でメモしたDropboxのリンクを指定します。
