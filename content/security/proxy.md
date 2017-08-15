Title: Proxy設定まとめ
Date: 2017-05-22 17:31
Modified: 2017-05-22 17:31
Slug: overproxy
Tags: proxy, security
Authors: yktsr
Summary: ありとあらゆるProxyを超えてみる

ご利用は自己責任で。

Proxyはときに守ってくれるようで、時には爆発してほしいと思うこともある。

仲良く付き合うためのtips

## 環境変数を解釈してくれる場合
```shell
$ export http_proxy=proxy-server:8080
$ export https_proxy=proxy-server:8080
```
http://を入れると、正しく解釈してくれるツールとしてくれないツールがある。


## github
```shell
$ git config --global http.proxy http://proxy-server:8000
$ git config --global https.proxy http://proxy-server:8000
```

## wget
```shell
# vim /etc/wgetrc
$ vim ~/.wgetrc
```

に以下を追加
```shell
http_proxy=http://proxy-server:8000
```

## curl
```shell
$ vim ~/.curlrc
proxy=http://proxy-server:8000
```

## npm
    :::shell
    $ npm -g config set proxy http://proxy-server:8000
    $ npm -g config set https-proxy http://proxy-server:8000
    $ npm -g config set registry http://registry.npmjs.org/
    $ npm config list


## pip
設定ファイルでするやり方が分からなかったので, aliasでコマンドを上書きする.
```shell
$ vim ~/.bash_rc
alias pip3='pip3 --proxy=proxy-server:8000'
$ source ~/.bash_rc または $ . ~/.bash_rc
```

## ATOM
```shell
$ apm config set https-proxy http://proxy-server:8000
```

## SSH
proxycommandなるものが用意されている。これは入力を標準入力から受け取り、出力を標準出力へ返すフィルタを指定する。
ここに透過プロキシとして動作するコマンドを指定する。
例えば、GithubへProxyを経由してSSH接続する場合（SSH over HTTPS）、
次のように設定ファイルを指定する。

```shell
$ cat ~/.ssh/config
Host github.com
  HostName ssh.github.com
  port 443
  ProxyCommand ncat --proxy "proxy-server:8000" %h %p
```
ただし、
```shell
# apt install nc
```
ここで用いるnetcat(nc, ncatなど)にはオリジナル版、nmap付属版など複数あるらしく、現在はnmapに付属しているものが高機能なようだ

ここで、ncatはフィルタであるので、
```shell
$ echo "GET /" | ncat --proxy proxy-server:8000 google.com 80
HTTP/1.0 302 Found
Cache-Control: private
Content-Type: text/html; charset=UTF-8
Referrer-Policy: no-referrer
Location: http://www.google.co.jp/?gfe_rd=cr&ei=
Content-Length: 261
Date: Mon, 22 May 2017 08:49:27 GMT

<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>302 Moved</TITLE></HEAD><BODY>
<H1>302 Moved</H1>
The document has moved
<A HREF="http://www.google.co.jp/?gfe_rd=cr&amp;ei=">here</A>.
</BODY></HTML>
```
ということもできる。
もし、標準入出力ですべての操作が行えるのであればこのような方法も可能だ。

## これら以外で超える
まだ万策尽きてはない。
[proxychains-ng](https://github.com/rofl0r/proxychains-ng)で強引に超えてみる。
proxychains-ngはネットワーク系のライブラリを置き換え、任意のコマンドでProxyを利用させる。

```shell
$ unzip proxychains-ng-master.zip
$ ./configure
$ make
$ sudo make install

$ sudo vi /etc/proxychains.conf
strict_chain

[ProxyList]
http YOUR_PROXY_IP_ADDRESS PORT


$ proxychains4 curl http://example.com/
```
例えば、SSHなら上のSSHの設定を行わなくとも
```shell
$ proxychains4 ssh -T github.com
```
で接続することができる。

## もぅﾏﾁﾞ無理......浅漬けにしよ
これらでも回避できない場合、外のサーバを踏み台にする。
SSHを使って外のサーバとVPNを構築し、VPNの中を通って外と通信させることを考える。
[参考にさせて頂いたサイト](http://www.unixuser.org/~euske/doc/openssh/book/chap6.html#real-vpn)

```shell
# cat /root/.ssh/config
Host 192.168.1.128
    Hostname            192.168.1.128
    Tunnel              point-to-point
    TunnelDevice        0:0
    PermitLocalCommand  yes
    LocalCommand        ( ifconfig tun0 192.168.3.2 pointopoint 192.168.3.1 ) &

# ssh root@192.168.1.128
# ifconfig tun0 192.168.3.1 pointopoint 192.168.3.2
# ifconfig
tun0      Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
      inet addr:192.168.3.1  P-t-P:192.168.3.2  Mask:255.255.255.255
      UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1
      RX packets:12 errors:0 dropped:10 overruns:0 frame:0
      TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
      collisions:0 txqueuelen:500
      RX bytes:1288 (1.2 KB)  TX bytes:0 (0.0 B)

# ping 192.168.3.2
```
あとはルーティングとかなんやかんやを設定する。

### 参考
[入門OpenSSH 様](http://www.unixuser.org/~euske/doc/openssh/book/appendix.html)
<br>SSHのありとあらゆる機能が書かれています

[ももいろテクノロジー 様](http://inaz2.hatenablog.com/entry/2014/08/20/004106)
