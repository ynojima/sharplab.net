---
layout: blog
date: 2020-06-21T11:55:55.675Z
title: EPubをDeepL APIを用いて機械翻訳で対訳
thumbnail: ""
---
EPub形式の電子書籍を、翻訳精度が良いことで最近話題のDeepLのAPIを用いて機械翻訳で対訳するCLIツールを作ってみました。

<https://github.com/sharplab/epub-translator>

CLIツールを作るなら、折角だからとネイティブビルドが出来、起動速度が速いことが特徴の[Quarkus](https://quarkus.io/)を使っています。

![](/img/epub-translator.png)

このCLIツールにEPubファイルを食べさせると、上記のように、ブロック毎に原文の下に訳文を挿入した対訳形式のEPubファイルを出力します。

## 使用方法

### 事前準備

epub-translatorの実行には、Java11以降が必要です。適宜ダウンロードしてインストールして下さい。

epub-translatorは、[GitHubのリリースページ](https://github.com/sharplab/epub-translator/releases)で配布しています。epub-translator-runner.jarというファイルをダウンロードしてください。

また、epub-translatorは設定ファイルが必要です。epub-translator-runner.jarを配置したディレクトリに、configというサブディレクトリを作成し、その中にapplication.ymlというファイルを作成し、以下のフォーマットで、DeepLのAPI鍵を記載してください。

```yaml
ePubTranslator:
  deepL:
    apiKey: <put your api key here>
  language:
    source: en        # default source language
    destination: ja   # default destination language
```

### 実行

以下のコマンドで実行できます。

```shell
java -jar <path-to-epub-translator-dir>/epub-translator-runner.jar --src <path-to-epub-file>
```

### Linux用バイナリ

なお、epub-translatorはQuarkusを用いていることから、ネイティブビルドにも対応しています。
Linuxで利用できるバイナリもGitHubのリリースページでepub-translator-runnerという名前で配布しています。

WSL2環境含むLinux環境で使えますので、こちらもお試し下さい。なお、Linuxバイナリの場合は、Java11のインストールは不要です。

```shell
<path-to-epub-translator-dir>/epub-translator-runner.jar --src <path-to-epub-file>
```

電子書籍や、技術文書を読む助けになれば幸いです。