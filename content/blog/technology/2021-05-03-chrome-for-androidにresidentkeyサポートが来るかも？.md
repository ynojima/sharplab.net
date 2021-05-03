---
layout: blog
date: 2021-05-03T01:53:20.443Z
title: Chrome for AndroidにResidentKeyサポートが来るかも？
---
Android FIDO2 API(com.google.android.gms:play-services-fido)のJavaDocを久しぶりにチェックしたところ、[AuthenticatorSelectionCriteria.BuilderにsetRequireResidentKeyメソッドが！](https://developers.google.com/android/reference/com/google/android/gms/fido/fido2/api/common/AuthenticatorSelectionCriteria.Builder#public-authenticatorselectioncriteria.builder-setrequireresidentkey-boolean-requireresidentkey)

どうも1/25にリリースされた、play-services-fidoの19.0.0-betaで追加されていたようです。

これまで、Android FIDO2 APIはResidentKeyをサポートしていなかったのでChrome for AndroidもResidentKeyのサポートをしておらず、credentialIdをlocalStorageに保存して無理矢理residentKeyっぽい動作を実現したりしていましたが、Android FIDO2 APIがResidentKeyをサポートすれば、そういうワークアラウンドも不要になりそうです。

Resident Credentialが生成されるようになったことで、これまで決して生成されることがなく、幻のAttestationStatementと化していたAndroid Key Attestation Statementが現れる日がくるかもしれません（Android Key Attestationは、Android Keystoreに鍵を保存した時のAttestationをそのまま使うAttestationフォーマットの為、Non-ResidentKeyしか生成できなかったこれまでは、生成されることがありませんでした）