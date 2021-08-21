# DBを用いたブログシステムの構築

プライベートサブネットにあるDBサーバーに各種ソフトウェアをインストールして、WordPressを構築する。

## この商の内容

- DBサーバーにMariaDBをインストールする
- WebサーバーにWordPressをインストールする
- WordPressの初期設定

## DBサーバーのインストール

以下のコマンドを実行する

```
sudo yum -y install mariadb-server
```

以下で起動する

```
sudo systemctl start mariadb

# 初期設定
mysqladmin -u root password
<好きなパスワードを設定する>
<もう一度入力する>
```

### WordPress用のデータベースを作成する

```
mysql -u root -p
```

WP用のデータベースを作成する

```sql
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

ユーザーを作成し、wordpressデータベースの操作権限を与える

```sql
grant all on wordpress.* to wordpress@"%" identified by 'password';
```

上記の権限を反映させる

```sql
flush privileges;
```

作成されたユーザーの確認

```
select user , host from mysql.user;
```

wordpressユーザーが「%」として登録されているため、このユーザーはすべてのホストから接続できるはず。

MariaDBはexitコマンドで終了する。

### 自動起動するように構成する

```
sudo systemctl enable mariadb
```

## WebサーバーにWordPressをインストールする

### PHPの最新版をインストールする

```
sudo amazon-linux-extras install php7.3
```

### PHPやMariaDBライブラリのインストール

```
sudo yum -y install php php-mbstring
sudo yum install mariadb-server
# mariadbとの疎通確認
mysql -h 10.0.2.10 -u wordpress -p
```

### WordPressのダウンロード

```
cd ~
wget https://ja.wordpress.org/latest-ja.tar.gz
```

### 展開とインストール

```
tar xzvf latest-ja.tar.gz
cd wordpress
# プログラム一式をApacheから見える場所に配置する
sudo cp -r * /var/www/html
# ファイルの所有者・グループをapache/apacheに変更する
sudo chown apache:apache /var/www/html -R
```

## WordPressを設定する

Apacheを起動する

```
sudo systemctl start httpd
```

### WordPressの初期設定

パブリックインスタンスのIPを入力すると、初期設定画面が表示される

データベースを以下のように設定する

- DB名：wordpress
- ユーザー名：wordpress
- パスワード：入力したもの
- データベースのホスト名：10.0.2.10
- テーブル接頭辞：wq_

次のページに移ったらインストールを実行をクリックする

サイトのタイトルなどは適当に設定する。

以上で設定は完了する。使わなくなったNATゲートウェイはひとまず削除しておく。
