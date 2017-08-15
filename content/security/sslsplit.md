Title: sslsplit使い方
Date: 2017-06-23 11:06
Modified: 2017-06-23 11:06
Slug: sslsplit
Tags: sslsplit, security
Authors: yktsr
Summary: sslsplitはSSL/TLSのMan In The Middleテストを行う際に用いるツールである。有名なmitmproxyに比べ、HTTPSだけでなく、あらゆるX over SSL/TLSに用いることができる。しかし、日本語の情報が圧倒的に不足しているので紹介

# sslsplitとは
中間者攻撃を行い、テスト対象機器が正しく証明書を検証しているか確認するツール。
昨今はスマートフォン向けアプリなどで証明書検証不備が多数報告されている。
有名なmitmproxyは、プロトコルをHTTP前提としているが、sslsplitはX over SSL/TLSに幅広く使える。

（正確にはmitmproxyでも証明書の交換手順に関してテストすることは可能であるが、
証明書が交換されたあとの通信をうまく扱うことができない）

# インストール
Kali Linuxはインストール済み。Ubuntuはapt経由でインストール可能。
Debianはstretchとbusterで提供されているが、手元の環境では正常に動作しなかった。
```shell
$ sudo aptitude install sslsplit
```

# 使い方
## 環境
Kali Linux on Virtual Boxで行った。
Kali Linuxをルーター化済み。

## 準備
証明局証明書の作成。manに書いてあるとおり。
```shell
$cat >x509v3ca.cnf <<'EOF'
  [ req ]
  distinguished_name = reqdn

  [ reqdn ]

  [ v3_ca ]
  basicConstraints        = CA:TRUE
  subjectKeyIdentifier    = hash
  authorityKeyIdentifier  = keyid:always,issuer:always
  EOF

$ openssl genrsa -out ca.key 2048
$ openssl req -new -nodes -x509 -sha256 -out ca.crt -key ca.key \
           -config x509v3ca.cnf -extensions v3_ca \
           -subj '/O=SSLsplit Root CA/CN=SSLsplit Root CA/' \
           -set_serial 0 -days 3650
```

## HTTPとHTTPSの例
```shell
# iptables -t nat -A PREROUTING -p tcp --destination-port 80 -j REDIRECT --to-ports 10080
# iptables -t nat -A PREROUTING -p tcp --destination-port 443 -j REDIRECT --to-ports 10443

$ sslsplit -D -k ca.key -c ca.crt -l connect.log -L sample.log https 0.0.0.0 10443 http 0.0.0.0 10080
```
これは、
```shell
$ mitmproxy -e -T -p 10443
```
と同等。

## SMTPの例
```shell
# iptables -t nat -A PREROUTING -p tcp --destination-port 465 -j REDIRECT --to-ports 10465
$ sslsplit -D -k ca.key -c ca.crt -l connect.log -L sample.log ssl 0.0.0.0 10456
```
  テスト環境動作確認を行うには以下
```shell
$ openssl s_client -connect smtp.gmail.com:465 -showcerts
```

##
間違い、ご意見などなどありましたら、twitterでお寄せください。
