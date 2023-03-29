---
title: Unity InputField 입력 이벤트
author: LHH
date: 2023-03-29 13:15 GMT+0900
categories: [Study, Unity]
tags: [Unity, InputField]
---

`InputField`를 사용해 로그인 비밀번호 입력을 구현하는 과정에서 입력 중이거나 `Enter`를 쳤을 때

다음 입력 구간으로 넘어가도록 구현하는 방법을 찾아보았다.

## 💻 코드
### onSubmit
`onSubmit`는 입력이 끝나고 마무리로 `Enter`를 눌렀을 때 호출되는 이벤트 메소드이다.

다음 코드는 `id` 입력 후 `Enter`를 누르면 `password`가 선택되어 입력 가능한 상태가 되고,

`password`에서 `Enter`를 누르면 `OnPrint()`가 실행되는 코드이다.
```cs
TMP_InputField _inputId;
TMP_InputField _inputPassword;

void Start()
{
    _inputId.onSubmit.AddListener(delegate{_inputPassword.Select();});
    _inputPassword.onSubmit.AddListener(delegate{OnPrint();});
}

// 출력
void OnPrint()
{
    Debug.Log($"id : {_inputId.text}, password : {_inputPassword.text}");
}
```
<br>

InputField를 다른 방법으로 사용한 일이 생기면 글을 추가 수정할 예정이다.

## 💡 참고
- [NJSUNG BLOG: Unity - InputField 입력 이벤트 구현하기](https://naakjii.tistory.com/83){:target="_blank"}