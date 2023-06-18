---
title: (3D MMORPG) 아이템 DeepCopy (깊은 복사)
author: LHH
date: 2023-06-18 15:15 GMT+0900
categories: [Portfolio, (3D Unity) MY_MMORPG]
tags: [Unity, Portfolio]
---

> 구현 기간 : 06.18

이걸 뒤 늦게 구현한 이유는 `Dictionary`에서 꺼내올 때 내부에서 자동으로 `깊은 복사(Deep Copy)`를 진행할 것이라 생각했다.

`Dictionary`를 통해 `Item`을 저장하여 사용하고 있는데 강화하는 과정에서 이를 눈치챘다.

무기 아이템을 5강까지 강화했는데 똑같은 `id`를 가진 무기들이 모두 5강이 되어 있었다.

이를 통해 `얕은 복사`가 진행된 다는 것을 눈치챘고 `깊은 복사`로 수정하였다.

- `얕은 복사` : 새로운 클래스를 생성하지 않고 클래스의 주소만 복사해온다.
- `깊은 복사` : 새로운 클래스를 생성하여 복사한다.

## 🎬 구현 결과
![DeepCopy](https://github.com/LHuHyeon/MY_MMORPG/assets/110723307/49aa5baa-4f27-48e6-897a-5f4afc39d52f)
