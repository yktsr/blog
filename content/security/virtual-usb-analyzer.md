Title: USBプロトコル解析
Date: 2017-05-24 18:16
Modified: 2017-05-24 18:16
Slug: usb-protocol-analyzer
Tags: security, usb, protocol
Authors: yktsr
Summary: Virtual USB AnalyzerとVMwareを利用し、仮想USBからパケットを解析する


シリアル通信を開始したものの、なぜか繋がらない。断線しているのか？
USBプロトコル・アナライザなんて高価なものは持ってないし、
確かめようにもオシロスコープは手元にないし、グランドがどの端子なのかもわからないし..... (´-ω-`) 

なんとかならないか......
[Virtual Usb Analyzer](http://vusb-analyzer.sourceforge.net/)などという神ツールを発見。早速やってみる

1. [VMware](http://www.vmware.com/products/player/playerpro-evaluation.html)を拾ってきてインストール

1. VMwareに適当なOS(Ubuntuなど）をインストール

1. ホスト側PCで
```
sudo apt install vusb-analyzer
```
1. VMwareのゲストOSがインストールされているフォルダ内、vmxファイルに以下を追記する
```
   monitor = "debug"
   usb.analyzer.enable = TRUE
   usb.analyzer.maxLine = 8192
   mouse.vusb.enable = FALSE
```

1. シリアル通信を開始
1. VMwareのゲストOSがインストールされているフォルダ内、vmware.logにUSBのログが吐き出される
```
grep USBIO vmware.log > vmware-usb.log
```
1. vusb-analyzer vmware-usb.log


VM使わなくても、Wiresharkでできたのか...
