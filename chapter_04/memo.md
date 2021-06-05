# 4章 Webサーバーソフトをインストールする

## 4-1 Apache HTTP Serverのインストール

サーバーにSSH接続して、以下のコマンドを入力

```
sudo yum -y install httpd
```

以下でApacheを起動

```
sudo systemctl start httpd.service
```

以下で、自動起動するように構成する

```
sudo systemctl enable httpd.service
```

以下で、Apacheのプロセスを確認する

```
ps -ax
```

このような行が表示されていればOK。

```
3564 ?        Ss     0:00 /usr/sbin/httpd -DFOREGROUND
```

また、ネットワークの待受状態を以下で確認する。

```
sudo lsof -i -n -P | grep httpd
```

Apacheが80番ポートで待ち受けていることがわかる。

```
httpd    3564     root    4u  IPv6  23962      0t0  TCP *:80 (LISTEN)
httpd    3565   apache    4u  IPv6  23962      0t0  TCP *:80 (LISTEN)
httpd    3566   apache    4u  IPv6  23962      0t0  TCP *:80 (LISTEN)
httpd    3567   apache    4u  IPv6  23962      0t0  TCP *:80 (LISTEN)
httpd    3568   apache    4u  IPv6  23962      0t0  TCP *:80 (LISTEN)
httpd    3569   apache    4u  IPv6  23962      0t0  TCP *:80 (LISTEN)
```

## 4-2 ファイアウォールを設定する

現状のままではHTTP接続を許可していないのでブラウザから確認することができない。セキュリティグループを変更する必要がある。以下のようにしてHTTPを追加する。

![](https://i.imgur.com/50TfN04.jpg)

ルールが適用されるとブラウザからテストページを参照することができる。

![](https://i.imgur.com/AsvPA13.jpg)

## 4-3 ドメイン名と名前解決

DNSを用いて、あるドメイン名から、それに対応するIPアドレスを引き出すことを「名前解決」と呼ぶ。

IPアドレスとドメイン名を変換するのはDNSサーバーの役割。実際にDNSサーバーを構成する。

- VPCの設定ページを開く
- 最初に作った「VPC領域」を選択する
- アクション→DNSホスト名の編集を選択
- 「DNSホスト名の有効化」をチェックして保存する
  - ![](https://i.imgur.com/UM2IYiI.jpg)

実際にどのようなDNS名称がつけられたのかを確認する。

先程作成したインスタンスを選択して、パブリックDNSとプライベートDNSを確認する。両方設定されていることがわかる。パブリックDNS名も起動するたびにランダムとなり、IPアドレス固定化とともに固定される。

パブリックDNSでも、テストページにアクセスすることが可能。