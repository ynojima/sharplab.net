---
layout: blog
date: 2020-06-06T13:00:34.144Z
title: WebAuthn WebDriver Extension
---
<!--StartFragment-->

WebAuthnを認証に使用するサービスを開発したとして、テストを自動化しようとすると、Authenticatorをどう用意するかが問題となります。WebAuthn4Jでは、単体テストに組み込んで使うAuthenticatorとして、[WebAuthn仕様に定義されたModel Authenticatorのエミュレータ](https://github.com/webauthn4j/webauthn4j/tree/master/webauthn4j-test/src/main/java/com/webauthn4j/test/authenticator/webauthn)や、CTAP AuthenticatorをJavaで実装したりしてきました。

一方、ブラウザを含めたE2Eテストを実現しようとすると、ブラウザに接続し、テストコードから操作できるAuthenticatorを用意する必要があります。Chromeでは、この課題に対して、Web Authentication Testing APIとしう名前で、Chromeに対して仮想的なAuthenticatorを追加するフラグが用意されていました。何故過去形かというと、最近のStableリリースからは、このフラグが削除され、WebAuthn Level2仕様のドラフトで策定が進められている、WebAuthn WebDriver Extensionという、WebDriverの拡張という形のAPIに改められた為です。

WebDriverは、E2Eテストなどでブラウザの操作の自動化を行う為のAPIで、各ブラウザ毎にDriverと呼ばれるソフトウェアが、JSONベースのREST APIの形式で、ブラウザの操作を自動化するAPIを公開しています。SeleniumなどのE2Eテストフレームワークは、このWebDriver APIを呼び出すクライアントという構成です。そして、このWebAuthn WebDriver Extensionは、WebAuthn関係の操作自動化を行う為に、WebDriverに追加のAPIを定義するものです。

WebAuthn WebDriver Extensionでは、Authenticatorの追加削除、Credentialの追加削除、UVフラグの設定等、実現出来る操作の自由度が増しており、WebAuthnを認証に使用するサービスとして実現したい操作をよくカバーしています。

selenium-javaの4.0.0系は、2020年6月現在アルファ版ですが、[WebAuthn WebDriver Extensionのサポートが既に追加](https://github.com/SeleniumHQ/selenium/issues/7753)されており、[WebAuthn4j Spring Securityのテストもそちらに切り替えてみたところ、上手く動作することが確認できました。](https://github.com/webauthn4j/webauthn4j-spring-security/blob/368681da1d4c2c23b97c6f1e80bebfb7ed0df3ca/samples/spa/src/test/java/e2e/RegistrationAndAuthenticationE2ETest.java#L74-L75)

WebAuthnを認証に取り入れたサービスを開発する場合は、WebAuthn WebDriver Extensionを用いてE2Eテストを積極的に実装していきましょう。

<!--EndFragment-->