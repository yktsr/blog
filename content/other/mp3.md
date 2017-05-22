Title: Mp3のタグ情報をUTF-8にする
Date: 2017-05-19 14:25
Modified: 2017-05-19 14:25
Slug: mp3tag2UTF8
Tags: mp3, shell
Authors: yktsr
Summary: Windows時代に作成したレガシーmp3ファイルがLinux環境で文字化けする。コマンドラインだけでなんとか解決する

# シンプルに
mid3iconvをインストールして

	find `pwd` -type f -name "*.mp3" -print0 | xargs -0 mid3iconv -d -e CP932 --remove-v1
