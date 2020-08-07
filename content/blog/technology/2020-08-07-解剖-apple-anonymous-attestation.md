---
layout: blog
date: 2020-08-07T13:53:56.791Z
title: 解剖 Apple Anonymous Attestation
thumbnail: ""
---
OS Public Beta4と共にSafariがアップデートされ、SafariのWebAuthnサポートでTouch ID/Face IDを利用した時にAttestationを返却するようになりました。AppleはPlatform Authenticatorが返却するAttestationとして、独自のApple Anonymous Attestation Statementを新たに定めており、その内容を紐解いてみたので共有です。

## Auth0 WebAuthn Debuggerでの調査

新しいAuthenticatorやClient Platformが出てきた時、その動作を検証する最も簡単な方法は、Auth0の提供するWebAuthn Debuggerというツールを使うことです。WebAuthnのAPIを呼び出すパラメータを自由に調整するGUIと、WebAuthnのAPIのレスポンスを読みやすい形に整形したうえで出力してくれるフォーマッタを備えた非常に便利なツールです。iOS 14 Public Beta4をインストールしたiPad miniで、AttestationパラメータにDirectを指定してAttestationがレスポンスに含まれるようにした状態で呼び出した結果が以下の通りです。

![Auth0 WebAuthn Debugger](/img/webauthn-debugger.png)

フォーマットとして、”apple”が指定されたAttestation Statementが返却されています。この新しいAttestation Statementを、Appleは、”Apple Anonymous Attestation”と名付けたようです。

## Apple Anonymous Attestationのフォーマット

”Apple Anonymous Attestation”の中身を見てみると、algパラメータにCOSEのアルゴリズムが、x5cパラメータにX.509証明書のリストが含まれているようです。ここまではPacked Attestation Statementや、Android Key Attestation Statementとあまり差はありませんね。

Apple Anonymous Attestationで特徴的なのは、sigパラメータが存在しない点です。他のPackedやTPM、Android KeyといったAttestationでは、sigというパラメータにAuthenticatorDataとClientDataHashを連結したデータに対する署名が含まれており、その署名をAttestation証明書の公開鍵で検証することで、AuthenticatoDataやClientDataHashのデータの真正性を検証できたのですが、Apple Anonymous Attestationにsigがないとはどういうことなのでしょうか。Apple Anonymous Attestationは、公開鍵の真正性のみしか担保しないのでしょうか。実は、Apple Anonymous AttestationにもAuthenticatorDataとClientDataHashの真正性を担保するデータが含まれており、後述します。

## X.509証明書の中身

x5cメンバに含まれるX.509証明書について見ていきましょう。Auth0 WebAuthn Debuggerには、x5c要素の部分に、"Download PEM“というボタンがあり、Attestation証明書をpem形式でダウンロードすることが可能です。 こちらをダウンロードして、手元のOpenSSLコマンドで表示してみた結果が以下の通りです。

```
$ openssl x509 -text -fingerprint -noout -in ./attestation.pem 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 1596599632997 (0x173bcc10465)
        Signature Algorithm: ecdsa-with-SHA256
        Issuer: CN = Apple WebAuthn CA 1, O = Apple Inc., ST = California
        Validity
            Not Before: Aug  5 03:43:52 2020 GMT
            Not After : Aug  6 03:53:52 2020 GMT
        Subject: CN = b26908ed8b3775f55eea9766fa1e666e4a869801c63798693054c8b23add73e7, OU = AAA Certification, O = Apple Inc., ST = California
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:78:32:99:37:c4:1e:60:82:cd:68:70:12:e8:ef:
                    cb:3a:ad:b6:4b:53:e1:4d:aa:9a:10:f0:41:e9:64:
                    fb:2d:6a:a9:da:e8:70:e3:5d:eb:84:c4:24:ce:73:
                    9c:e0:e0:b6:7c:5e:6f:be:82:ba:34:f6:9e:10:cf:
                    f2:65:c8:d5:37
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Key Usage: critical
                Digital Signature, Non Repudiation, Key Encipherment, Data Encipherment
            1.2.840.113635.100.8.2:
                0$.". ....T7cmt@.A...9.".1..X.<..|.M..
    Signature Algorithm: ecdsa-with-SHA256
         30:65:02:30:78:d5:60:2b:a2:39:37:1e:dd:66:d0:74:4e:a0:
         59:b1:80:f2:a9:13:32:56:29:18:4e:c2:d8:82:8e:f9:1f:57:
         42:d3:76:d9:b9:7f:d2:f0:ec:86:e3:a9:d1:89:cd:90:02:31:
         00:d1:41:9a:07:d3:ba:e7:80:a1:f2:9a:c6:e3:39:1d:3e:c5:
         07:c9:40:00:5c:07:2f:cd:ee:63:db:31:92:6a:c7:33:b8:d8:
         72:78:99:78:5e:bd:3e:50:2d:95:f9:66:51
SHA1 Fingerprint=50:D5:11:B0:F3:A0:7C:DF:AD:8F:37:8D:18:7F:31:77:77:39:5C:51
```

一つ特徴的なのが、Validityが非常に短いこと。一日しかありません。USBドングルタイプのセキュリティキーの場合、Attestation証明書が焼きこまれている為か、Attestation証明書のValidityは数十年先とかに設定されていることが多いですが、Apple Anonymous Attestationの場合は、デバイスの中でAttestation証明書は生成しており、一つチェーンした先の"Apple WebAutn CA 1“という中間認証局の秘密鍵がデバイス側にあるようにみえます。

## OID “1.2.840.113635.100.8.2“とApp Attest API

このAttestation証明書でもう一つ特徴的なのが、 “1.2.840.113635.100.8.2“というOIDを持った拡張の存在です。調べてみると、"1.2.840.113635"がAppleに割り当てられたプライベートなOIDで、OID "1.2.840.113635.100.8.2"の定義を調べていると、8月4日に発表された、[iOS 14以降で新たに提供された](https://developer.apple.com/jp/news/?id=2sngpulc)、[App Attest API](https://developer.apple.com/documentation/devicecheck/establishing_your_app_s_integrity)というものに辿り着きました。iOSでのアプリ開発については、全く土地勘を持たないので、斜め読みしただけなのですが、iOS14からは、デバイスがルート化されていたり侵害されていないかをチェックするためのAPIが提供されるようです。Androidでいうところの、Android SafetyNet APIに似ていますね。[そのAPIはデバイスの真正性を保証するデータとして、Attestation Statementというデータを返却するそうです。](https://developer.apple.com/documentation/devicecheck/validating_apps_that_connect_to_your_server)まるでWebAuthnみたいですね？実際、こちらのAttestation Statementのドキュメントを読んでみると、Web Authenticationというキーワードが出てきており、WebAuthnの仕様を参考に設計されたAPIのようです。

ただ一方でこちらのドキュメントで説明されているAttestation Statementのスキーマは以下の通りです。

```
$$attStmtType //= (
                      fmt: "apple-appattest",
                      attStmt: StmtFormat
                  )

StmtFormat =      {
                      x5c: [ credCert: bytes, * (caCert: bytes) ],
                      receipt: bytes,
                  }
```

フォーマットとしてiOS Safariが先ほど返してきた”apple”ではなく、”apple-appatest”が指定されており、Attestation Statementの中には、algメンバが無い代わりに、receiptというメンバが含まれています。どうも、”apple-appatest”は、iOSアプリ向けの[App Attest API](https://developer.apple.com/documentation/devicecheck/establishing_your_app_s_integrity)の為のAttestation であり、WebAuthn向けの”apple” Attestationとは少し異なるようです。

話がOID “1.2.840.113635.100.8.2“から逸れてしまいました。OID “1.2.840.113635.100.8.2“は、[”apple-appatest”の検証手順を解説したドキュメント](https://developer.apple.com/documentation/devicecheck/validating_apps_that_connect_to_your_server)の中で、AuthenticatorDataとClientDataHashを連結したデータのSHA-256ハッシュから得られるnonceと一致するか検証する為に用いられることが定められています。Apple Anonymous Attestationにはsigメンバが存在せず、AuthenticatorDataとClientDataの真正性の保証がどうやって行われるのか、謎でしたが、Attestation証明書の中にAuthenticatorDataとClientDataHashを連結したデータのSHA-256ハッシュ（nonce）を含めることで真正性を検証可能にしていたのですね。

## 実装してみた

という訳で、Apple Anonymous Attestationを検証するValidatorをwebauthn4j-spring-securityのサンプルアプリに組み込む形で実装してみたのがこちらです：<https://github.com/webauthn4j/webauthn4j-spring-security/pull/399/files#diff-e23631eb615506ce7a6680400cb40191>

あとは[Apple Private PKI](https://www.apple.com/certificateauthority/private/)で公開されている[Apple WebAuthn Root CA](https://www.apple.com/certificateauthority/Apple_WebAuthn_Root_CA.pem)を組み込めば、iOS safariのTouch ID/Face IDによるWebAuthnのAttestationの検証は完成と言って良さそうですね。正式にAppleからWebAuthnのAttestationの検証手順が公開されたら、WebAuthn4J本体にも取り込んでいきたいと思います。

## まとめ

AppleはiOS Public Beta4と共にSafariをアップデートし、Touch ID/Face IDを使ってiOSデバイスをPlatform Authenticatorとして利用した際に、WebAuthn APIはAttestationを返却するようになりました。Apple Anonymous Attestationという新しい形式のAttestationが導入されており、同じく8/4に新たに公開された[App Attest API](https://developer.apple.com/documentation/devicecheck/establishing_your_app_s_integrity)で用いられているAttestationと形式が類似しています（同一ではありません）。Apple Anonymous Attestationは、AuthenticatorDataとClientDataの真正性の保証を、Attestation証明書に、“1.2.840.113635.100.8.2“というOIDの拡張を含めることで実現しています。

iOS SafariでTouch ID/Face IDが利用できるようになったことにより、PC、Mac、Android、iPhone、主要デバイス全てでWebAuthnを利用する下地が出来たと言えます。特に、Touch ID/Face IDでのWebAuthnは、モバイルデバイスでのログインのユーザーエクスペリエンスを大きく向上させるものであり、WebAuthnの普及に繋がることを期待したいですね。