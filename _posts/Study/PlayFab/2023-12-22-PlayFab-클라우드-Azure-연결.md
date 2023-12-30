---
title: (PlayFab) CloudScript에 Azure 연결하기
author: LHH
date: 2023-12-23 14:00 GMT+0900
categories: [Study, PlayFab]
tags: [PlayFab, BaaS, Azure]
---

PlayFab에서 클라우드 스크립트는 자바스크립트만 작성할 수 있기 때문에 Unity를 하던 저에겐 부담스러운 일이었습니다.

그러나 C#을 사용하고 디버그까지 할 수 있는 쉬운 방법이 있어 정리하게 되었습니다.

### 기본 준비물
- PlayFab 계정 및 타이틀
- Visual Studio Code (글에서는 1.85.1 사용)
- Unity

### 목차
1. Azure 계정 등록 및 구독
2. Visual Studio Code 확장 및 설정
3. 테스트 코드 작성
4. Azure 함수 등록 및 PlayFab 연결
5. 테스트

<br>

## 1. Azure 계정 등록 및 구독
### Azure에 로그인 후 체험판 등록
1. [Microsoft Azure(클릭)](https://azure.microsoft.com/ko-kr/free/)에 접속합니다.

2. `무료로 시작하기` 클릭

3. 로그인 해줍니다.

4. 기본 정보 작성

5. 카드 정보 작성

### Azure 구독
1. 체험판을 등록하면 구독이 자동으로 생성되어 있을겁니다.

2. 만약 생성이 안됐다면 [여기를 클릭](https://learn.microsoft.com/ko-kr/azure/cost-management-billing/manage/create-subscription)하여 `구독 만들기`를 참고하고 생성해주세요.

<br>

## 2. Visual Studio Code 확장
### .Net 6 설치
1. [.Net 6](https://dotnet.microsoft.com/ko-kr/download/dotnet/6.0)에 접속하여 설치해줍니다.

2. `cmd`를 켜서 `dotnet`을 입력하여 작동되는지 확인

### VSCode 필수 확장
- [Azure Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack)

- [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)

- [PlayFab Explorer](https://marketplace.visualstudio.com/items?itemName=PlayFab.playfab-explorer)

### 로컬 프로젝트 생성
1. 먼저 원하는 위치에 폴더를 생성하고 `VSCode`로 열어줍니다.

2. 좌측 탭에서 `Azure`를 클릭합니다. (단축키 : Shift + Alt + A)

3. 하단 `WORKSPACE`에 커서를 가져가면 `번개모양 Icon`이 보이는데 그걸 클릭!

4. `Create New Project` 클릭 후 다음과 같이 진행해 주세요.

    - `폴더 선택`

    - `C#`

    - `.NET 6.0`

    - `HTTP trigger`

    - 기본으로 생성될 함수 이름인데 `HelloWorld`라고 지어줍시다.

    - C# namespace 이름입니다. 원하는대로 적어주세요.

    - 권한 부분입니다. 저는 `Anonymous`로 진행했습니다. ([권한의 자세한 내용[클릭]](https://learn.microsoft.com/ko-kr/azure/azure-functions/functions-bindings-http-webhook-trigger?tabs=python-v2%2Cisolated-process%2Cnodejs-v4%2Cfunctionsv2&pivots=programming-language-csharp#authorization-keys))

    - `WORKPACE`에 로컬 프로젝트가 생성되었는지 확인!

5. VSCode에 Terminal을 열고 `dotnet add package PlayFabAllSDK`을 실행해줍니다.

6. 마지막으로 CS2AFHelperClasses.cs[(CS2AFHelperClasses Container)](https://github.com/PlayFab/PlayFab-Samples/blob/master/Samples/CSharp/AzureFunctions/CS2AFHelperClasses.cs) 소스코드를 다운로드하여 폴더에 넣어줍니다.

### Azure 로그인 및 Function 생성
1. 좌측 탭에서 `Azure`를 클릭합니다. (단축키 : Shift + Alt + A)

2. 상단 `RESOURCES`에서 로그인을 진행합니다.

3. 구독(열쇠 모양)에 우클릭 후 `Create Resource` -> `Create Function App in Azure` 순서로 클릭!

4. 함수 이름 작성 -> `.Net 6` -> `Korea Central` 선택

5. 리소스 그룹과 함께 `Function`, `Storage`가 생성됐는지 확인!

<br>

## 3. 테스트 코드 작성
### 클라이언트(Unity) 측 구현
- 클라이언트에서 Azure 함수를 호출하는 코드를 구현합니다.

- PlayFab에 로그인한 사용자는 Azure의 HelloWorld 함수를 실행하여 inputValue 인수 이름에 Test 문자열을 넣어 함수를 호출합니다.

#### 클라이언트 코드
```cs
using PlayFab;
using PlayFab.ClientModels;
using PlayFab.CloudScriptModels;

using UnityEngine;

public void Login()
{
    // TODO : PlayFab 로그인 먼저 진행

    CallCSharpExecuteFunction(); 
}

// Azure 함수 호출
private void CallCSharpExecuteFunction()
{
    // Azure 함수 실행
    PlayFabCloudScriptAPI.ExecuteFunction(new ExecuteFunctionRequest()
    { 
        Entity = new PlayFab.CloudScriptModels.EntityKey()
        {
            // 로그인할 때부터 가져오기 ( Id, Type )
            Id = PlayFabSettings.staticPlayer.EntityId,
            Type = PlayFabSettings.staticPlayer.EntityType
        },
        FunctionName = "HelloWorld", // Azure 함수의 이름
        FunctionParameter = new Dictionary<string, object>() { { "inputValue", "Test" } },
        GeneratePlayStreamEvent = true
    },
    CallSuccess, CallError);
}

// 성공 시
private void CallSuccess(ExecuteFunctionResult result)
{
    if (result.FunctionResultTooLarge != null && (bool)result.FunctionResultTooLarge)
    {
        Debug.Log("This can happen if you exceed the limit that can be returned from an Azure Function," +
            " See PlayFab Limits Page for details.");
        return;
    }
    Debug.Log($"The {result.FunctionName} function took {result.ExecutionTimeMilliseconds} to complete");
    Debug.Log($"Result: {result.FunctionResult.ToString()}");
}

// 실패 시
private void CallError(PlayFabError error)
{
    Debug.Log($"Opps Something went wrong: {error.GenerateErrorReport()}");
}
```

<br>

### 서버(Azure Functions) 측 구현
- Azure 함수에 등록할 코드를 구현합니다.

- HelloWorld라는 Azure Function을 실행하고, 인수로
받은 inputValue를 기록하고, 플레이어 ID를 반환 값으로 반환합니다

#### 서버 코드
```cs
[FunctionName("HelloWorld")]
public static async Task<dynamic> Run(
[HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
        ILogger log)
{
    string body = await req.ReadAsStringAsync();
    var context = JsonConvert.DeserializeObject<FunctionExecutionContext<dynamic>>(body);
    var args = context.FunctionArgument;
    var message = $"Hello {context.CallerEntityProfile.Lineage.MasterPlayerAccountId}!";
    log.LogInformation(message);

    dynamic inputValue = null;
    if (args != null && args["inputValue"] != null)
    {
        inputValue = args["inputValue"];
    }
    log.LogDebug($"HelloWorld: {new { input = inputValue } }");

    return new { messageValue = message };
}
```

<br>

## 4. Azure 함수 등록 및 PlayFab 연결
### Azure에 HelloWorld 서버 코드 등록
1. 좌측 탭에서 `Azure`를 클릭합니다. (단축키 : Shift + Alt + A)

2. 하단 `WORKSPACE`에 방금 작성한 HelloWorld 서버 코드 파일이 존재하는지 확인합니다. 만약 없으면 폴더안에 파일이 있는지, 저장되어 있는지 확인하세요.

3. `WORKSPACE` 칸에 마우스 커서를 대고 `번개모양 Icon`을 누르고 `Deploy to Function App`를 클릭합니다.

4. 로딩이 완료되면 상단 `RESOURCES`의 `Functions`에 등록되는 것을 확인합니다.

### Azure 함수 PlayFab에 연결
1. 방금 등록한 함수에 우클릭하고 `Copy Function URL`을 클릭하여 복사합니다.

2. 좌측 탭에서 `PlayFab Explorer`를 클릭합니다.

3. 로그인 진행

4. 함수를 등록하고 싶은 타이틀을 우클릭하고 `Register HTTP function`을 클릭합니다.

5. 방금 복사한 URL을 붙여넣기 해주세요.

<br>

## 5. 테스트
- Unity를 실행하고 코드를 실행해줍니다.

- 다음과 같이 Debug되면 연결 성공!
    - The `<함수 이름>` function took `<시간>` to complete
    - Result: Hello `<플레이팹 Id>`

<br>

## ❓ 비용이 걱정된다면 
Azure를 사용하다보면 `Storage` 부분에서 비용이 발생하는 것을 알 수 있습니다.

12개월 체험판 제품 중 `Storage`가 포함되어 있어 비용이 발생하더라도 무료로 사용이 가능합니다.

12개월이 지나더라도 Microsoft에서 서비스를 계속 진행할 것인지 알려주기 때문에 기간이 지나면 딱 끝나고 추가 결제는 안된다고 합니다.

<br>

## 💡 참고
- [[Microsoft] 빠른 시작: Azure Functions를 사용하여 PlayFab CloudScript 작성](https://learn.microsoft.com/ko-kr/gaming/playfab/features/automation/cloudscript-af/quickstart){:target="_blank"}
- [[PlayFab 마스터] CloudScript에서 Azure Functions를 실행하는 방법](https://playfab-master.com/playfab-azure-cloudscript#rtoc-1){:target="_blank"}
- [[김우창 개발 블로그] Azure Functions - VSCode에서 함수 생성](https://nanenchanga.tistory.com/entry/Azure-Functions-VSCode%EC%97%90%EC%84%9C-%ED%95%A8%EC%88%98-%EC%83%9D%EC%84%B1){:target="_blank"}