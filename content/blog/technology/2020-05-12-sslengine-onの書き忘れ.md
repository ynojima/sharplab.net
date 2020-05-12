---
layout: blog
date: 2020-05-12T15:09:56.092Z
title: SSLEngine onの書き忘れ
---
AparcheのVirtualHostでHTTPSを使う際、<VirtualHost hoge.example.com:443></VirtualHost>の中に

\`SSLEngine on\` というディレクティブを記載する必要があるが、これを忘れたVirtualHostが存在すると、他のVirtualHostが正しく設定されていても \`SSLEngine on\` を書き忘れたVirtualHostに引きずられてHTTPSではなく、生のHTTP通信として受け付けようとしてしまうので注意。1日無駄にした。。