---
layout: blog
date: 2020-03-22T12:08:24.500Z
title: Concourse CIをCentOS8に構築しようとしてfirewalldでハマった
---
Concourse CIをCentOS8に構築しようとしたところ、firewalldで嵌りました。

Concouse CIはDocker Composeで起動するのが、公式のドキュメントのQuick Startでも紹介されている簡単な方法のようですが、今回は、手でSystemdのUnitファイルを記述して立ち上げてみることにしました。

しかし、一点嵌りまして、Concourse CIは、各種ビルド・テスト処理をコンテナの中で実施するのですが、Concourse CIが立ち上げるコンテナから外部に対する通信が、CentOS8のfirewalldに落とされてしましました。

Concouse CIが示すエラーログが以下の通り。名前解決が出来ないというエラー。ホスト側のCentOSでは、問題なく名前解決が出来ているので、コンテナ側の問題のようです。

```
resource script '/opt/resource/check []' failed: exit status 128

stderr:
Cloning into '/tmp/git-resource-repo-cache'...
fatal: unable to access 'https://github.com/webauthn4j/webauthn4j/': Could not resolve host: github.com

```

以下のコマンドで、firewalldによる通信制限のログを見てみると、

```
sudo firewall-cmd --set-log-denied all
```

以下のようなエラーメッセージが残っていました。

```
Mar 22 19:29:24 feedback kernel: FINAL_REJECT: IN=wbrdg-0afe0000 OUT=enp1s0 MAC=92:d2:a6:07:97:94:72:f8:15:8b:5b:e3:08:00 SRC=10.254.0.2 DST=8.8.8.8 LEN=56 TOS=0x00 PREC=0x00 TTL=63 ID=17552 DF PROTO=UDP SPT=46304 DPT=53 LEN=36 

```