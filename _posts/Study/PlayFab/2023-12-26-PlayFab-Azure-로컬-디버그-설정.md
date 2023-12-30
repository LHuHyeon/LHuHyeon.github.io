---
title: (PlayFab) Unity에서 Azure 함수를 로컬에서 디버그하기
author: LHH
date: 2023-12-26 12:00 GMT+0900
categories: [Study, PlayFab]
tags: [PlayFab, BaaS, Azure]
---

> [(PlayFab) CloudScript에 Azure 연결하기]() 이어서 진행합니다.

<br>

# `아직 글 작성 중! 참고만 하세요!`
여러 자료를 참고하며 시도하였지만 몇번을 새로하고 고쳐도 작동을 안하네요.. 해결되면 다시 작성하겠습니다.

---

<br>
<br>

PlayFab에서 Azure 함수를 사용하여 디버그하면 제대로 작동하는 것을 알았습니다.

이번에는 Azure 함수로 넘어가기 전 로컬 서버에서 디버그하고 실행결과를 확인하는 방법을 알아보겠습니다.

<br>

## 목차
1. Azure 함수를 로컬로 디버그하는 방법
2. 요약

<br>

## 1. Azure 함수를 로컬로 디버그하는 방법
### 로컬 디버깅 개념
Azure 함수를 호출하는 방법을 `일반 모드`, Azure 함수를 등록하기 전 로컬에서 서버로 디버그하는 방법을 `디버그 모드`라고 가정합니다.

이런 `일반 모드`와 `디버그 모드`에는 한가지 차이점이 있습니다.

<br>

### 일반 모드와 디버그 모드의 차이점
- 일반 모드
    - https://test.azurewebsites.net/api/HelloWorld
    - 함수의 이름에 따라 URL이 변경됩니다.

- 디버그 모드
    - http://localhost:7071/api/CloudScript/ExecuteFunction
    - 함수 이름을 변경하더라도 항상 똑같은 URL이 됩니다.

이렇게 디버그 모드에서 실행하고 싶어도 고정된 URL이 발급되기 때문에 오류가 발생합니다. 

이제 방법을 알아보겠습니다.

<br>

### Azure Functions Core Tools 설치
[여기 사이트](https://learn.microsoft.com/ko-kr/azure/azure-functions/functions-run-local?tabs=windows%2Cisolated-process%2Cnode-v4%2Cpython-v2%2Chttp-trigger%2Ccontainer-apps&pivots=programming-language-csharp#install-the-azure-functions-core-tools)에 접속하셔서 `Azure Functions 핵심 도구 설치` 단락에서 설치를 진행해줍니다.

<br>

### ExecuteFunction 함수 준비
[여기 사이트](https://github.com/PlayFab/pf-af-devfuncs/blob/master/csharp/ExecuteFunction.cs)에 접속하셔서 `ExecuteFunction 샘플`을 다운로드합니다.

해당 샘플을 살펴보면 가장먼저 눈에 띄는 것은 다음 세줄인데요.
```cs
private const string DEV_SECRET_KEY = "PLAYFAB_DEV_SECRET_KEY";
private const string TITLE_ID = "PLAYFAB_TITLE_ID";
private const string CLOUD_NAME = "PLAYFAB_CLOUD_NAME";
```

PlayFab의 정보를 입력하는 것 같은데.. 알고보니 환경변수를 추출하기 위한 string 변수였습니다.

이 때문에 해당 string에 자신의 값을 넣는 것이 아니라 환경변수로 등록하고 다음과 같은 코드처럼 사용되어야 합니다.
```cs
string title = Environment.GetEnvironmentVariable(TITLE_ID, EnvironmentVariableTarget.Process);
var secretKey = Environment.GetEnvironmentVariable(DEV_SECRET_KEY, EnvironmentVariableTarget.Process);
```

<br>

### 환경변수 설정
간단하니 다음 블로그에서 번역하시고 따라하시면 됩니다.

- [Azure Functions에서 환경 변수를 설정하는 방법[서버 및 로컬에 필요]](https://playfab-master.com/azure-environment-variable)