---
layout: blog
date: 2021-05-03T01:53:20.443Z
title: Chrome for AndroidにResidentKeyサポートが来るかも？
---
Android FIDO2 API(com.google.android.gms:play-services-fido)を久しぶりにチェックしたところ、[AuthenticatorSelectionCriteria.BuilderにsetRequireResidentKeyメソッドが！](https://developers.google.com/android/reference/com/google/android/gms/fido/fido2/api/common/AuthenticatorSelectionCriteria.Builder#public-authenticatorselectioncriteria.builder-setrequireresidentkey-boolean-requireresidentkey)

これまで、Android FIDO2 APIはResidentKeyをサポートしていなかったので