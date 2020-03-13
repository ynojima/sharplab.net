---
layout: blog
date: 2020-03-08T12:21:13.859Z
title: Netlify CMS + DocsyでBlog構築
---
そろそろ0x20歳を迎えることや、最近ライフイベントが重なったこともあり、久しぶりにBlogを再開することにしました。前回Blogを構築したのは20歳になろうかという頃でしたので、0x20歳に差し掛かろうかという今新たにBlogを作るのも良いタイミングかなと考えてのことです。

前回Blogを構築した際は、WordPressを用いていましたが、最終的にオペレーションミスでデータを失ってしまい、そのまま閉鎖となってしまいました。今回はその反省を踏まえ、静的サイトジェネレータであるHugoをベースに、静的サイトジェネレータ向けCMSである"Netlify CMS"と、Hugoの技術サイト向けテンプレートである"Docsy"を組み合わせてBlogを構築することにしました。

## Hugo

Hugoは、Markdownなどで記述されたコンテンツをベースに、テーマを適用して静的なサイトを生成してくれる静的サイトジェネレータです。その名前から推測がつくように、Go言語で開発されています。コンテンツ執筆者はMarkdownなどで簡単にコンテンツを記述出来つつも、テーマを適用することでサイトを適切にスタイリング出来ます。従来のWordPressのような動的なCMSでは、DBや実行環境の維持管理をする必要がありましたが、Hugoのような静的サイトジェネレータを利用する場合は、最終的な成果物は静的なファイルとして出力されるので、Netlifyのような静的サイトのホスティングサービスにデプロイすることが出来、維持管理コストを削減することが出来ることが魅力です。

## Docsy

Docsyは、Google発のHugoのテーマで、オープンソースソフトウェアの公開ページなど、技術文書公開に適した機能を重点的に具備していることが特徴です。[Kubeflowの公式サイト](https://www.kubeflow.org/)もDocsyで構築されているようです。

## Netlify

NetlifyはGitHub等レポジトリサービスと連携し、レポジトリの更新を契機にコンテンツを取得し、ビルドして成果物を公開する継続的デリバリ機能と、無償プランから独自ドメイン持込でHTTPSが利用できることが特徴の静的サイトのホスティングサービスです。

## Netlify CMS

![](/img/netlify-cms.png "Netlify CMS")

Netlify CMSは、前述のNetlifyが提供しており、静的サイトジェネレータと組み合わせて使うことを前提とした、新しいコンセプトのCMSです。Netlify CMSとして提供される、管理画面を提供する2つのファイル（HTMLファイルとYAMLファイル）を組み込み先のサイトに対して追加した上でそのパスに対してアクセスすると、Netlify CMSは静的サイトのコンテンツが保管されているGitHub等のレポジトリに対して（Netlifyを経由して？）APIアクセスを行い、コンテンツの編集機能を提供します。

Netlify CMS上でコンテンツの入力を完了し、「保存」ボタンを押下すると、コンテンツの内容はGitHub等のレポジトリに対してコミットされ、そのコミットをトリガーにNetlifyはHugo等静的コンテンツジェネレータを実行し、ビルドして得られたコンテンツを表示する、という連携が動作します。

## 導入方法

本Blogを構築するにあたり必要な設定は全て[サイトのGitHubレポジトリ](https://github.com/sharplab/sharplab.net)に収まっているので、基本的にはそちらを参照して頂ければ追いかけることが出来るかと思いますが、自分が躓いたポイントを中心に導入方法を記載したいと思います。

### Hugoのインストール

Hugoは、Go言語で開発されているため、シングルバイナリで動作します。実行ファイルは公式のGitHubのリリースページにAssetsとして公開されている為、実行環境に合わせたバイナリをダウンロードし、パスの通ったディレクトリに配置しておきましょう。Hugoには、通常版とExtended版が存在しますが、DocsyはExtended版を必要とするため、Extended版をダウンロードしましょう。

<https://github.com/gohugoio/hugo/releases>

### ディレクトリ構造の生成

Hugoはコンテンツファイルや、テーマファイルがどのように配置されているべきかについて、期待するディレクトリ構造を持ちます。以下のコマンドで、この構造に従うミニマムなディレクトリ構成を生成することが出来ます。

```shell
hugo new site <site-name>
```

### Docsyテーマの導入

Docsyは以下のコマンドでHugoのサイトに対して導入することができます。

```
# docsyテーマが依存するユーティリティをnpmでインストール
sudo npm install -D --save autoprefixer
sudo npm install -D --save postcss-cli

# docsyテーマをsubmoduleとして導入
cd <site-name>
git init
git submodule add https://github.com/google/docsy.git themes/docsy
echo 'theme = "docsy"' >> config.toml
git submodule update --init --recursive
```

参考：<https://www.docsy.dev/docs/getting-started/#option-2-use-the-docsy-theme-in-your-own-site>

Docsyテーマの細かいカスタマイズについては、[公式ドキュメント](https://www.docsy.dev/docs/adding-content/lookandfeel/)を参照してください。

### サイトのビルド

サイトのビルドは、`hugo` コマンドを単体で呼び出すことで実行できます。

```
hugo
```

参考：<https://gohugo.io/getting-started/usage/#the-hugo-command>

### Netlifyの利用開始

では、サイトの基本的な構造ができたところで、Netlifyにデプロイしてみましょう。

まず、Netlifyで継続的デリバリを実行するのに必要なファイルを準備します。

#### Hugoのバイナリ

Netlifyの継続的デリバリ機能でサイトをビルドするために、HugoのLinux用バイナリをレポジトリにコミットしておきましょう。NetlifyもHugoのバイナリを提供してくれますが、Extended版ではないので、Docsyを使う為に[sharplab.net では、binディレクトリを作成し、hugoのバイナリをコミットしています](https://github.com/sharplab/sharplab.net/tree/master/bin)。

#### Netlifyの設定ファイル

Netlifyは、netlify.tomlというファイルを継続的デリバリ対象のレポジトリのルートディレクトリに配置しておくことで、そのファイルに記述した設定に従い、継続的デリバリを実行してくれます。

sharplab.netでは、以下のような設定を行い、必要な依存関係を取得した上で、コミットしたhugoのバイナリでビルドを行っています。

```
[build]
publish = "public"
command = "git submodule update -f --init --recursive; npm install; ./bin/hugo --gc --minify"
HUGO_ENV = "production"

[context.production.environment]
HUGO_ENABLEGITINFO = "true"

[context.deploy-preview]
command = "git submodule update -f --init --recursive; npm install; ./bin/hugo --gc --minify --buildFuture -b $DEPLOY_PRIME_URL"

[context.branch-deploy]
command = "ngit submodule update -f --init --recursive; pm install; ./bin/hugo --gc --minify -b $DEPLOY_PRIME_URL"

[context.next.environment]
HUGO_ENABLEGITINFO = "true"
```



### Netlify CMSの導入

続いては、Netlify CMSの導入です。Netlify CMSは、静的リソースとして公開されるディレクトリに所定のHTMLファイルを配置することで、導入が可能です。Hugoの場合、`<site root>/static` に配置されたファイルが、静的リソースとして公開される為、Netlify CMSの導入の為に、`<site root>/static/admin/index.html`を以下の内容で作成します。

```
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Content Manager</title>
</head>
<body>
  <!-- Include the script that builds the page and powers Netlify CMS -->
  <script src="https://unpkg.com/netlify-cms@^2.0.0/dist/netlify-cms.js"></script>
</body>
</html>
```

参考：<https://www.netlifycms.org/docs/add-to-your-site/#app-file-structure>

### Netlify CMSの設定

最後に、Netlify CMSの設定をしていきましょう。

Netlify CMSの設定は、Netlify CMSの`index.html`ファイルの隣に、`config.yml`というファイルを作成することで可能です。Netlify CMSで画像をアップロードする際のアップロード先ディレクトリや、CMSでエントリを作成する際の作成先ディレクトリ、コンテンツ保存バックエンド等を設定可能です。

詳しくは[Netlify CMSの公式ドキュメント](https://www.netlifycms.org/docs/add-to-your-site/#configuration)を参照してください。

## まとめ

以上のように、Hugo、Docsy、Netlify、Netlify CMSを組み合わせることで、アプリケーション実行環境やDBの管理に煩わされることなく、リッチなインタフェースでBlog等コンテンツを管理することが出来る環境を構築することが出来ました。