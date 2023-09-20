---
title: (Unity) Animator Override Controller
author: LHH
date: 2023-09-20 22:00 GMT+0900
categories: [Study, Unity]
tags: [Unity, Animator]
---

`Animation`을 사용하다 보면 같은 캐릭터끼리 같은 애니메이션 패턴을 사용하는 경우가 생긴다.

예를 들어서 전사, 궁수, 마법사가 `Idle`, `Attack`, `Dead` 애니메이션을 사용한다고 가정한다.

그렇다면 이들은 `Attack` 애니메이션의 경우 서로 다른 애니메이션을 사용할 것이다.

이때 `AnimatorOverrideController`를 사용하면 같은 패턴안에 다른 애니메이션을 동작할 수 있다.

생성 방법은 `+` -> `Create` -> `Animator Override Controller`를 클릭하여 생성하면 된다.

![](https://github.com/LHuHyeon/LHuHyeon.github.io/assets/110723307/e0754b33-5bcb-499a-99d0-731dbb37682c)

생성되면 사진과 같이 나타나는 것을 볼 수 있고, Controller에 사용하고 싶은 애니메이션 컨트롤러를 넣고 사용하면 된다.