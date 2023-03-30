---
title: Unity Login 문자열 검사
author: LHH
date: 2023-03-30 18:00 GMT+0900
categories: [Study, Unity]
tags: [Unity, Login, Regex]
---

> **구현한 코드를 정리한 글입니다.**

## 💻 코드
### LoginManager
`Login`을 하기 위해서 `ID`와 `Password`의 규칙을 만들어줘야 된다.

- `ID` : 특수문자가 없는 영문 4~12자
- `PW` : 영문, 숫자, 특수문자가 각각 1자이상씩 들어간 8~18자

이에 규칙에 맞게 정규표현식을 사용하여 구현해준다.
```cs
public class LoginManager
{
    // 로그인 체크
    public bool LoginPassCheck(string id, string password)
    {
        if (IdPassCheck(id) == true && PasswordCheck(password) == true)
            return true;
        
        return false;
    }

    // ID 문자열 체크
    public bool IdPassCheck(string str)
    {
        if (str != null && str.Length <= Define.MAX_LOGIN_LENGTH && str.Length >= Define.MIN_LOGIN_LENGTH)
        {
            Regex regex = new Regex(@"^[0-9a-zA-Z]{4,12}$");
            if (regex.IsMatch(str))
            {
                Debug.Log("아이디 문자열 통과");
                return true;
            }

            Debug.Log("IDPassCheck : 특수문자는 금지입니다.");
        }

        return false;
    }

    // Password 문자열 체크
    public bool PasswordCheck(string str)
    {
        if (str != null && str.Length <= Define.MAX_PASSWORD_LENGTH && str.Length >= Define.MIN_PASSWORD_LENGTH)
        {
            Regex regex = new Regex(@"^(?=.*?[a-z])(?=.*?[0-9])(?=.*?[#?!@$%^&*-]).{8,18}$", RegexOptions.IgnorePatternWhitespace);
            if (regex.IsMatch(str))
            {
                Debug.Log("비밀번호 문자열 통과");
                return true;
            }

            Debug.Log("PasswordCheck : 영문, 숫자, 특수기호가 포함되어야 합니다.");
        }

        return false;
    }
}
```
<br>

### SigninPopup
로그인 팝업을 만들어주고 적용해준다.
```cs
public class UI_SigninPopup : MonoBehaviour
{
    public TMP_InputField _inputId;
    public TMP_InputField _inputPassword;

    void Start()
    {
        // InputField
        _inputId.onSubmit.AddListener(delegate{ _inputPassword.Select(); });
        _inputPassword.onSubmit.AddListener(delegate{ OnCheckButton(); });
    }

    // 로그인 확인
    void OnCheckButton()
    {
        Debug.Log("OnCheckButton");
        // 올바른 문자열 확인
        if (Managers.Login.LoginPassCheck(_inputId.text, _inputPassword.text) == false)
        {
            Debug.Log("올바른 문자열입니다!");
        }
        else
            Debug.Log("아이디나 비밀번호가 올바르지 않습니다.");
    }
}
```