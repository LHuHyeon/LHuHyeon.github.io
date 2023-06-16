---
title: (Unity) CapsuleCollider Center 변경법
author: LHH
date: 2023-06-05 16:00 GMT+0900
categories: [Error, Unity (Error)]
tags: [Unity, Error, CapsuleCollider Center, Set()]
---

`CapsuleCollider`를 사용하는 중 `center` 값을 변경하기 위해 구글링 하지 않고 바로 수정을 시도했는데 황당한 일이 벌어졌다.

다음과 같이 `center`를 수정했고 실행했을 땐 적용이 안됐다.
```cs
CapsuleCollider capsuleCollider;

capsuleCollider.center.Set(2f, 3f, 4f);
```
<br>

이상할게 없는데 적용이 안되자 다른 코드에 문제가 있나 검토했고 다른 코드엔 이상이 없었다..

다시 돌아와 `Set()`을 쓰지 않고 `Vector3` 값을 그대로 전달해주니 작동이 됐다.

```cs
capsuleCollider.center = new Vector3(0, 0, 0.4f);
```
<br>

이럴거면 왜 만들어 놓은건지 모르겠다..