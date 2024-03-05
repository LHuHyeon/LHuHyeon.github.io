---
title: (Unity) Fake Null 정리
author: LHH
date: 2024-03-05 17:00 GMT+0900
categories: [Study, Unity]
tags: [Unity, FakeNull]
---

유니티의 기능은 C#으로 작성되지만 실제로는 `C++ Native Object`를 감싸 C#으로 `Wrapping`하여 동작합니다.
  
오브젝트를 `Destroy` 하면 `C++ Native Object`가 삭제되고 `C# UnityEngine Object`는 메모리에 그대로 존재하게 됩니다.

이러한 `C# UnityEngine Object`는 가비지 컬렉션(GC)가 동작할 때 메모리에서 해제되어 삭제됩니다.

## Fake Null이란?
+ `C++ Native Object`가 삭제되고 GC가 동작하기 전 `C# UnityEngine Object`가 살아있는 상태

+ `Fake Null` 상태의 null을 출력하면 "null"로 처리된 것을 확인 가능합니다.
  
+ `public`이나 `[SerializeField]`로 선언된 오브젝트는 할당하지 않으면 `Fake Null`이 발생하니 주의합시다.

<br>

## 해결 방법
+ 오브젝트를 `Destroy`한 후 해당 오브젝트를 명시적으로 null을 넣어줍니다. *(완전한 Null 상태)*

+ `==, !=` vs `ReferenceEquals`
  + 오브젝트의 null 체크 연산 비용은 비싸다고 합니다. 그러므로 다음과 같이 사용해주도록 합시다.
  
  + `==, !=` Null 검사
    + 자주 바뀌고 삭제되는 객체나 컴포넌트
  
  + `System.Object인 ReferenceEquals` Null 검사
    + 생성된 이후 파괴되지 않는 객체
    + 단순한 캐싱 또는 싱글톤 객체

<br>

## 💻 코드
+ 제가 평소에 사용하고 있는 `Fake Null`과 `ReferenceEuqals` 검사 방법입니다.

```cs
// System.Object null 체크
public static bool IsNull(this UnityEngine.Object go)   { return ReferenceEquals(go, null); }
public static bool IsNull(this System.Object go)        { return ReferenceEquals(go, null); }

// Fake Null 체크
public static bool IsFakeNull(this UnityEngine.Object go) { return (go.IsNull() == false && go == true) == false; }
```

+ `IsFakeNull()`
  + Fake Null 검사가 필요한 상황에서 사용하면 됩니다.
    
  + `System.Object`와 `UnityEngine.Object`가 둘다 존재하면 `False`를 반환

  + 반대로 둘중 하나라도 존재하지 않는다 하면 `True`를 반환

<br>

## 💡 참고
+ [[공부하는 식빵맘] Unity C# > 유니티 오브젝트의 fake null](https://ansohxxn.github.io/unitydocs/fakenull/){:target="_blank"}

+ [[짱승_ 게임공장] 유니티 null 체크 유의사항](https://heinleinsgame.tistory.com/38){:target="_blank"}