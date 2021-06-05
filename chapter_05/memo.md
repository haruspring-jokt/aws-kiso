# 5章 HTTPの動きを確認する

## 5-3 Telnetを使ってHTTPをしゃべってみる

自らの手を動かしてHTTP通信をして体験してみる。

Macの場合、brewでtelnetをインストールしたあと、

```
telnet www.kantei.go.jp 80
GET / HTTP/1.1
HOST: www.kantei.go.jp

```

で、ブラウザで見たときと同じようなHTMLを文字列として受け取ることができる。

この他、AWSとはあまり関係ない内容だったため読み流した。