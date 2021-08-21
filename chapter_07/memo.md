# NATを構築する

## NATの用途と必要性

プライベートサブネットに配置したサーバーは、インターネットからの接続ができないため、安全ではあるが不便でもある。
そのため、「インターネットへの向き」だけを許可するNATを構築する。

これから、プライベートサブネットに配置したDBサーバーにDBソフトをインストールする。その前に、NATを構築してインターネット接続できるようにする。

### NATの仕組み

NATは、IPアドレスを変換する装置で、2つのネットワークインタフェースを持つ。
片側のインタフェースには、一般に、「パブリックIPアドレス」を設定し、インターネット接続可能な構成にしておく。
もう片側のインタフェースには、「プライベートIPアドレス」を設定し、プライベートサブネットに接続する。

プライベートサブネットに存在するホストがインターネットにパケットを送信しようとしたとき、NATは、パケットの送信元IPアドレスを自分のパブリックIPアドレスに置換する。
こうすることで、送信元がパブリックIPとなり、インターネットに出ていくことが可能になる。
応答パケットは、このNATに戻ってきて、その応答パケットの宛先をもとのホストのIPアドレスに置換してプライベートサブネットに置換します。

NATを用いるを逆にインターネットからプライベートサブネットの方向に接続することはできない。

AWSではNATゲートウェイという機能が提供されており、かんたんにNATを利用することができるようになっている。今回はこのNATゲートウェイを使用する。

### NATインスタンスとNATゲートウェイ

2つの方法がある。

- NATインスタンス
  - NATがインストールされたAMIから起動したEC2インスタンスを使う方法。
  - 通常のEC2インスタンスと同様に、利用していないときは停止することもできる。
- NATゲートウェイ
  - NAT専用に構成された仮想的なコンポーネント。配置するサブネットを選ぶだけで構成できる。
  - 負荷に応じてスケールアップし、時間あたりと転送バイトの転送量で決められる。

今回はNATゲートウェイを使う。

- パブリックサブネットにNATゲートウェイを設置する。
- プライベートサブネットからインターネット宛のパケットが、NATゲートウェイを経由する用に、ルートテーブルを変更する。

## NATゲートウェイを構築する

### NATゲートウェイを起動する

「VPC」→「NATゲートウェイ」を選択、「NATゲートウェイの作成」を選択する

サブネットではパブリックサブネットを選択する

Elastic IPは新規に割り当てる

### ルートテーブルを更新する

プライベートサブネットからインターネットに対して通信するとき、パケットがNATゲートウェイの方向に流れるように構成する

ルートテーブルのメニューを開き、「VPC領域」からメインが「はい」になっているものを選択する

ルートの編集メニューに入り、

- 送信先は `0.0.0.0/0` 
- ターゲットはNATゲートウェイのIDを入力

を設定して保存する。

## NATゲートウェイを通じた疎通確認

今回はDBサーバーにログインして、インターネットに対してHTTP、HTTPS通信ができるかを確認する。今回はcurlコマンドをしようする。

```html
[ec2-user@ip-10-0-2-10 ~]$ curl www.kantei.go.jp
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="description" content="首相官邸のホームページです。内閣や総理大臣に関する情報をご覧になれます。">
<meta name="keywords" content="首相官邸,政府,内閣,総理,内閣官房">
<meta property="og:title" content="首相官邸ホームページ">
<meta property="og:type" content="website">
<meta property="og:url" content="https://www.kantei.go.jp/index.html">
<meta property="og:image" content="https://www.kantei.go.jp/jp/n5-common/img/kantei_ogp.jpg">
<meta property="og:site_name" content="首相官邸ホームページ">
<meta property="og:description" content="首相官邸のホームページです。内閣や総理 大臣に関する情報をご覧になれます。">
<meta name="twitter:card" content="summary_large_image">
<meta name="format-detection" content="telephone=no">
<meta name="cmsdate" content="Thu Jan 01 00:00:00 JST 2015">
<title>首相官邸ホームページ</title>
<link rel="stylesheet" type="text/css" href="/jp/n5-common/css/aly.css">
<link rel="stylesheet" type="text/css" href="/jp/n5-common/css/common.css">
<link rel="stylesheet" type="text/css" href="/jp/n5-common/css/swiper.css">
</head>
<body id="page-top" class="top-page">
...
```

実際にインターネットに通信することができた。

## NATゲートウェイの削除

NATゲートウェイを使う必要がなくなったら、削除すると良い。
使わなくても課金され、中止状態にもできないため。