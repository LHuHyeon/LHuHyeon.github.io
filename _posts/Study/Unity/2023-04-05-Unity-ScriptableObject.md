---
title: Unity ScriptableObject
author: LHH
date: 2023-04-05 14:45 GMT+0900
categories: [Study, Unity]
tags: [Unity, ScriptableObject]
---

스크립터블 오브젝트(`ScriptableObject`)는 평소 우리가 사용하는 스크립트와 다르게 대량의 데이터를 저장하는데 사용할 수 있는 데이터 컨테이너이고,

값의 사본이 생성되는 것을 방지하여 프로젝트의 메모리 사용을 줄일 수 있다.

`ScriptableObject`를 사용하면 `MonoBehaviour` 스크립트에 변경되지 않는 중복된 데이터를 저장하는 프리팹이 있는 프로젝트의 경우 유용하다.

## ScriptableObject 사용방법
- 만들고 싶은 객체의 관련된 데이터 코드를 작성한다.
- 미리 상속된 `MonoBehaviour` 대신 `ScriptableObject`를 상속한다.
- 클래스 선언부분 위에 [CreateAssetMenu()] 속성을 사용한다. ***(선택)***

## 💻 코드
```cs
using UnityEngine;
[CreateAssetMenu(fileName = "New Item", menuName = "New Item/item")]
public class Item : ScriptableObject
{
    enum ItemType
    {
        Equipment,      // 장비
        Food,           // 음식
        Ingredient,     // 재료
        Used,           // 사용가능
        ETC             // 기타
    }
    public string itemName;     // 아이템 이름
    public Sprite itemImage;    // 아이템 이미지
    public ItemType itemType;   // 아이템 타입
    public int value;           // 가격
}
```

<br>

`※ [CreateAssetMenu()]란?`<br>
- 해당 스크립트를 편리하게 가져와서 사용할 수 있다.

`※ [CreateAssetMenu()] 사용방법`<br>
- CreateAssetManu(fileName = "", menuName = "", order = int값);
- `fileName` : 새롭게 만들게 되면 해당 이름으로 임시 생성된다.
- `menuName` : Assets -> Create에서 표기될 이름 설정
- `order` : 메뉴에서 보여질 순서

<br>

## 📝 ScriptableObject에 대한 의견
- 소규모의 데이터를 편하게 관리할 수 있는 장점이 있지만, 대용량의 데이터를 관리하기엔 어려움이 있을 것으로 예상된다. 대규모의 데이터 관리방법은 엑셀로 하는 것이 편하기 때문이다.

- 외부 데이터를 파싱해서 사용하는 번거로움을 피하고, 데이터의 관리 부하가 크지 않다면 `ScriptableObject`를 사용하는 것이 **편리하고 최적화**에 도움될 것으로 생각한다. 

<br>

내가 기존에 알던 방식은 npc 정보, item 정보 등 스크립트를 작성하고 해당 객체 컴포넌트에 할당하여 사용하였다.

이번에 알아낸 `ScriptableObject`를 통해 중복되는 데이터를 다룰 때 메모리를 효율적으로 사용할 수 있을 것 같다.

<br>

## 💡 참고
- [UnityBeginner 블로그 : ScriptableObject를 활용한 오브젝트 생성](https://unitybeginner.tistory.com/86){:target="_blank"}
- [EveryDay.DevUp 블로그 : ScriptableObject](https://everyday-devup.tistory.com/53){:target="_blank"}
- [Unity Documentation 유니티 공식 홈페이지](https://docs.unity3d.com/kr/2021.3/Manual/class-ScriptableObject.html){:target="_blank"}