---
layout: blog
date: 2020-04-18T01:56:23.563Z
title: Windows 10 WebAuthn APIの仕様注意点メモ
---
Windows 10 WebAuthn APIは、AttestationConveyancePreferenceを考慮せず、たとえnoneが設定されていてもAuthenticatorが返却したAttestationをそのままRPに返すので注意