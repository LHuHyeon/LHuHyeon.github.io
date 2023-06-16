---
title: (Unity) FixedUpdate GetKey
author: LHH
date: 2023-06-05 16:20 GMT+0900
categories: [Error, Unity (Error)]
tags: [Unity, Error, FixedUpdate, Input, GetKey]
---

조금이라도 최적화를 하기 위해 사용자의 키 입력을 받을 땐 `Update()`를 사용하고,

키 입력이 필요 없는 곳은 `FixedUpdate()`를 사용해줬다.

어느날 실수로 `FixedUpdate()`에서 `Input.GetKeyDown`을 실행했는데

한번 누를 때 1번 실행되는 것이 아닌 2~4번 정도 작동이 됐다.

이상함을 느끼고 `Update()`로 바꿔주자 정상적으로 1번 작동 됐다.

<br>

물론 움직이거나 다른 상황이 있다면 `FixedUpdate()`에서도 키 입력을 받아도 된다.

하지만 1번 누를때 딱 1번 실행을 원한다면 `Update()`에서 사용해야 된다.

<br>

컴퓨터마다 다를수는 있지만 매 프레임마다 실행되는 `Update()`와 달리

`FixedUpdate()`는 설정된 일정 값에 따라 딜레이 되며 호출되기 때문에

여러번 눌릴 가능성도 있을 것 같다.