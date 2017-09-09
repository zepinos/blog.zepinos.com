---
layout: post
title: Apps2Fire 을 통해 Fire TV Stick 에 KODI 을 설치할 때 INSTALL_FAILED_NO_MATCHING_ABIS 가 발생하는 문제
comments: true
category: etc
tags: 취미생활 FireTV AFTV
---

LG TV 에서 NAS 의 동영상 보는 것은 도저히 못해먹을 짓이라는 걸 드디어 인정하고(USB 로 퍼다 날라도 안되는건 안되는 것임) Amazon Fire TV Stick 을 네이버페이에 입점한 구매대행을 통해 새걸로 구입했습니다. 좀 비싸도 그냥 신경 끄고 있으면 되니까...

물건이 부족해서 오늘...주문 후 16일만에 받았는데...KODI 을 Apps2Fire 라는 앱으로 쉽게 옮길 수 있다고 하는데, INSTALL_FAILED_NO_MATCHING_ABIS 오류가 발생했습니다. 실험적으로 DS Video 을 옮겨봤는데 잘 되었습니다.

이리 저리 방법을 찾다 보니 adbLink 라는 프로그램을 알게되었고, 여전히 오류가 발생해서 INSTALL_FAILED_NO_MATCHING_ABIS 오류를 검색해보니(화면에 나온걸 다 적기 어려워서 adbLink 오류 났을 때 로그 파일 내에서 문구를 찾아냄) CPU 가 휴대전화와 조금 달라서 그런 걸로 보여졌습니다. x64 용으로 기본 설치가 됐는데, Fire TV Stick 은 x86 만 지원되는 것으로 보이네요.

kodi-17.0-Krypton-arm64-v8a.apk 대신에 kodi-17.0-Krypton-armeabi-v7a.apk 을 adbLink 프로그램으로 설치해주니 잘 되네요. 
