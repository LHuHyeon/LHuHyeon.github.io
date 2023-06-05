---
title: Unity foreach Error
author: LHH
date: 2023-06-05 16:10 GMT+0900
categories: [Error, Unity (Error)]
tags: [Unity, Error, foreach]
---

> `Error` : InvalidOperationException: Collection was modified; enumeration operation may not execute.

팝업창들을 관리하기 위해 전체삭제를 구현하는 중 `foreach`에서 에러가 났다.

이유는 간단했다.
```cs
List<GameObject> popupList = new List<GameObject>();

void ClosePopup(GameObject go)
{
    popupList.Remove(go);
    Destroy(go);
}

void CloseAllPopup()
{
    foreach(GameObject go in popupList)
    {
        ClosePopup(go);
    }
}
```
<br>

다음 코드에서 에러가 나는 이유는 `foreach`에서 `popupList`안에 있는 요소들을 전체 불러오는데

반복문을 실행하는 중 `popupList`의 요소가 삭제됐기 때문에 에러가 일어난 것이다.

그래서 다음과 같이 코드를 수정해줬다.
```cs
List<GameObject> popupList = new List<GameObject>();

void ClosePopup(GameObject go)
{
    Destroy(go);
}

void CloseAllPopup()
{
    foreach(GameObject go in popupList)
    {
        ClosePopup(go);
    }

    popupList.Clear();
}
```
<br>

## 💡 참고
[루덴의 지식 블로그: foreach 에러](https://ruen346.tistory.com/109){:target="_blank"}