---
title: Unity NetworkManager
author: LHH
date: 2023-04-02 13:20 GMT+0900
categories: [Study, Unity]
tags: [Unity, Network, API]
---

> **구현한 코드를 정리한 글입니다.**

웹서버와 협업을 한다면 꼭 접하는 내용이 `API`이다.

`API`는 서버와 통신할 수 있는 수단으로 서버에서 만들어준 `API`를 통해 `Data`를 주고 받을 수 있다.

나는 첫 협업을 통해 `Network`를 관리하는 `Manager`를 만들어 보았다.

## 💻 코드
### Data Class
`API`를 통해 통신하기 위해선 `Data`의 이름과 목록을 맞춰야한다.

간단하게 `id`와 `password`를 보내면 `status`를 뱉어주는 `API`가 있다 가정하겠다.

`status`는 `Data`의 성공유무를 알려준다.

클라이언트는 이에 맞게 클래스를 만들어줘야 한다. (클래스 이름은 자유)
```cs
[System.Serializable]
public class LoginData
{
    public string id;
    public string password;
}

[System.Serializable]
public class MembarData
{
    public string status;
}
```

### NetworkManager
본격적으로 `API`와 통신을 하려면 `API`의 `url`주소를 알아야한다.

해당 주소를 얻었다면 다음과 같이 진행한다.
```cs
public class NetworkManager
{
    public string baseURL = "https://<API 주소>";

    // 서버 송수신
    string ServerRequest<TRequest>(TRequest requestData)
    {
        // string -> byte[]로 변환
        string str = JsonUtility.ToJson(requestData);
        var bytes = System.Text.Encoding.UTF8.GetBytes(str);

        // 요청 보낼 주소와 세팅
        HttpWebRequest request = (HttpWebRequest)WebRequest.Create(baseURL);
        request.Method = "POST";
        request.ContentType = "application/json";
        request.ContentLength = bytes.Length;

        // Stream 형식으로 데이터를 보냄
        using(var stream = request.GetRequestStream()){
            stream.Write(bytes, 0, bytes.Length);
            stream.Flush();
            stream.Close();
        }

        // 응답 데이터를 StremaReader로 받음
        // HTTPWebResponse가 에러 뜨면 예외로 null 리턴
        string json = null;
        try
        {
            HttpWebResponse response = (HttpWebResponse)request.GetResponse();
            StreamReader reader = new StreamReader(response.GetResponseStream());
            json = reader.ReadToEnd();
        }
        catch
        {
            return json;
        }

        return json;
    }
}
```
`Data`의 종류와 이름은 다양하기 때문에 `<TRequest>` 제네릭을 선언하고 이곳에는 `LoginData`가 들어갈 것이다.

서버에 전송할때는 `Byte`로 주고 받아야되기 때문에 `Json`형태로 만든 후 `Byte`로 변환시켜준다.

`HttpWebRequest`로 보낼 주소와 어떤식으로 진행할지 세팅해준다.

세팅한 `request`를 `GetRequestStream`을 통해 서버에 전송한다.

`HttpWebResponse`로 서버로 부터 `data`를 받아준다.

혹시라도 `Error`가 뜨는 것에 대비해 `try catch`를 넣어 준다.

받은 값은 `json` 형태로 오기 때문에 `string`으로 리턴해준다.
<br>

### 사용하기 편리하게 구현
만들어진 `ServerRequest`를 사용해보도록 한다.

보내고 받을때의 데이터 구조는 각 상황마다 다르니 제네릭을 통해 클래스를 받아준다.

`TRequest`는 보내는 데이터 클래스, `TResponse`는 받는 데이터 클래스이다.

`ServerRequest`를 통해 json 값을 받으면 `JsonUtility`를 통해 `string` 값을 양식에 맞게 클래스에 넣어준다.
```cs
public static class NetworkManager
{
    public TResponse GetMemberData<TRequest, TResponse>(TRequest data) where TResponse : new()
    {
        string jsonData = ServerRequest<TRequest>(data);
        
        TResponse responseData;
        if (jsonData != null)
            responseData = GetJson<TResponse>(jsonData);
        else
            responseData = new TResponse();

        return responseData;
    }

    TResponse GetJson<TResponse>(string json)
    {
        // string 값을 JsonUtility로 커스텀 클래스
        TResponse info = JsonUtility.FromJson<TResponse>(json);

        return info;
    }
}
```

### 사용하기
`LoginData`를 생성하여 보낼 값을 넣어준다.

`NetworkManager`에서 만든 `GetMemeberData`를 사용하여 `detail`에 넣어준다.

`detail` 안에는 `status`가 있기 때문에 잘 전송 됐다면 서버에서 `true` 관련 문자열을 보내줄 것이다. (여기선 `Success`)
```cs
public class Login
{
    LoginData data = new LoginData(){ id = "testID", password = "test1234" };
    MembarData detail = NetworkManager.GetMemberData<LoginData, MembarData>(data);

    if (detail.status == "Success")
        Debug.Log("서버 전송 성공!");
    else
        Debug.Log("서버 전송 실패!");
}
```
<br>

## 💡 참고
- [서버와 통신 및 직렬화](https://kumgo1d.tistory.com/8){:target="_blank"}