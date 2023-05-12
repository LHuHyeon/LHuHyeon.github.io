---
title: Unity Slider NaN
author: LHH
date: 2023-05-12 17:14 GMT+0900
categories: [Error, Unity (Error)]
tags: [Unity, Error, Slider NaN, NaN]
---

UI 작업을 하는 중 `Slider`의 `value`를 코드에서 관리해주려는데 `Slider`가 작동을 하지 않는다.

원인은 `value`의 값이 `NaN`으로 되어있어 다른 값으로 변경 자체가 불가했다.

## NaN이란?
`NaN`은 `Not-a-Number`의 약자로 숫자로 표현할 수 없는 값을 뜻한다.

이런 이유 때문에 `NaN` 된다는건 숫자를 바꿀 수 없는 상황이니 사전에 차단을 해줘야한다.

## 💻 코드
`Update`를 통해 무한 반복이 될때 `slider.value`의 값이 `NaN`으로 들어가지 않도록 차단하는 코드이다.

코루틴을 통해 2초뒤에 `exp`값이 `slider.value`에 정상적으로 들어가게 된다.
```cs
class SliderTest : MonoBehaviour
{
    float exp = 0;
    float addExp = 20;

    Slider slider;
    
    void Start()
    {
        slider = GetComponent<Slider>();
        StartCoroutine(AddCoroutine());
    }

    void Update()
    {
        // [ 이 글의 핵심 코드 ]
        if (float.IsNaN(exp) == true)
            slider.value = 0;
        else
            slider.value = exp;
    }

    IEnumerator AddCoroutine()
    {
        yield return new WaitForSeconds(2f);

        exp += addExp;
    }
}
```

<br>

## 💡 참고
- [agnes0129: Unity 3D NaN](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=agnes0129&logNo=220245737778)