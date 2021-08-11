# 6章 プライベートサブネットを構築する

## プライベートサブネットの利点

インターネットから直接接続させたくないデータベースなどはプライベートサブネットに配置する。

## プライベートサブネットを作る

`10.0.2.0/24` としてサブネットを作成する。

事前に、パブリックサブネットは、「ap-northeast-1a」に作成されていることを確認する。

サブネットを作成する際に、同じAZを選択すること。

### ルートテーブルの確認

デフォルトでは、「自身のネットワーク」に対してのルーティングだけが設定されている。

プライベートサブネットなので、インターネットに接続する必要がなく、ルートテーブルを変更しない。

## プライベートサブネットにサーバーを構築する

- ネットワーク関連の設定で、プライベートサブネットを選択する。
- IPアドレスは、ネット接続しないので無効化し、プライベートIPを適当に選択する。
- サーバー名は「DBサーバー」とする。
- セキュリティグループでは、新規にSGを設定し、SSHとMySQL接続用の設定を追加する。
  
### pingコマンドで疎通確認する

pingコマンドで、WebサーバーからDBサーバーに疎通できるかを確認する。

pingは「ICMP」というプロトコルを使うので、ICMPが通るように構成する必要がある。

- DBサーバーのSGに、「ICMP」を許可するインバウンドを追加する。

設定したら、WebサーバーのほうにSSH接続する。

webサーバーから以下のようにpingを実行すると、DBサーバーから応答がある。

```
[ec2-user@ip-10-0-1-10 ~]$ ping 10.0.2.10
PING 10.0.2.10 (10.0.2.10) 56(84) bytes of data.
64 bytes from 10.0.2.10: icmp_seq=1 ttl=255 time=0.405 ms
64 bytes from 10.0.2.10: icmp_seq=2 ttl=255 time=0.421 ms
64 bytes from 10.0.2.10: icmp_seq=3 ttl=255 time=0.448 ms
64 bytes from 10.0.2.10: icmp_seq=4 ttl=255 time=0.438 ms
64 bytes from 10.0.2.10: icmp_seq=5 ttl=255 time=0.453 ms
```

また、WebサーバーにもICMPを許可するようSGを設定し、ローカルからpingできるようにする。

```
ec2-3-112-71-154.ap-northeast-1.compute.amazonaws.com [3.112.71.154]に ping を送信しています 32 バイトのデータ:
3.112.71.154 からの応答: バイト数 =32 時間 =7ms TTL=235
3.112.71.154 からの応答: バイト数 =32 時間 =6ms TTL=235
3.112.71.154 からの応答: バイト数 =32 時間 =8ms TTL=235
3.112.71.154 からの応答: バイト数 =32 時間 =10ms TTL=235

3.112.71.154 の ping 統計:
    パケット数: 送信 = 4、受信 = 4、損失 = 0 (0% の損失)、
ラウンド トリップの概算時間 (ミリ秒):
    最小 = 6ms、最大 = 10ms、平均 = 7ms
```

## 踏み台サーバを経由してSSHで接続する

Webサーバーを踏み台として、ローカル環境からDBサーバーに接続する。

windowsの場合、WebサーバーにDBサーバー用の秘密鍵をアップロードする。

今回はteratermを使ってアップする。ec2-userのホームディレクトリにアップする。

```
chmod 400 my-key-db.pem
ssh -i my-key-db.pem ec2-user@10.0.2.10
```

アップしたあとの秘密鍵の許可を変更し、ssh接続する。