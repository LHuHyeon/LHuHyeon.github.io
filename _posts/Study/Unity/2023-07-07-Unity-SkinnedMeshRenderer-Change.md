---
title: (Unity) SkinnedMeshRenderer 파츠 변경
author: LHH
date: 2023-07-07 09:40 GMT+0900
categories: [Study, Unity]
tags: [Unity, SkinnedMeshRenderer]
---

간단한 캐릭터 커스텀을 만들기 위해 장비 장착 에셋을 구매하여 사용하고 있다.

에셋 안에 캐릭터는 `SkinnedMeshRenderer`의 컴포넌트를 사용하고 있었고 이를 수정하기 위한 코드를 알아보았다.

처음에 당황했던 것은 `Mesh`를 다른 옷으로 변경해줬는데 `Scene`과 `Game`에서는 해당 부위가 보이지 않았다.

그래서 다시 원본 `Mesh`를 넣어줬더니 그래도 안보여서 너무 당황했다;; `( 결국 Ctrl + Z )`

그래도 구글링을 통해 파츠를 교체에 대한 정보를 알 수 있었다.

### 필요 정보
내가 생각한 했을땐 `SkinnedMeshRenderer`는 기본적으로 4가지의 정보가 필요하다.

1. sharedMesh ( Mesh )
2. bounds ( 위치 )
3. bones ( 뼈대 )
4. rootBone ( 대표 뼈대 )

이 중에 하나라도 문제가 생기면 옷이 찢어지거나, 사라지는 등 에러가 생긴다.

### 💻 코드
바꾸고 싶은 `SkinnedMeshRenderer`를 `public`으로 받고 4가지 정보를 넘겨주도록 한다.
```cs
public class CharacterCustom
{
    public SkinnedMeshRenderer changeSkinned;   // 바꾸고 싶은 SkinnedMeshRenderer
    public Transform rootBone;  // 대표 뼈대

    void Start()
    {
        SkinnedMeshRenderer objSkinned = gameobject.GetComponent<SkinnedMeshRenderer>();
        
        objSkinned.sharedMesh = changeSkinned.sharedMesh;
        objSkinned.localBounds = changeSkinned.localBounds;
        objSkinned.bones = (Transform[])changeSkinned.bones.Clone();
        objSkinned.rootBone = rootBone;
    }
}
```

### Scene 변경 시 주의!!
나는 캐릭터를 커스텀해줄 `Custom Scene`과 게임을 진행하는 `Game Scene`을 따로 만들어줬기 때문에 윗 코드를 그대로 쓰면 해결이 안된다.

`Mesh, Bounds` 경우 잘 가져오지만 `Transform` 같은 경우 `Scene`이 넘어가는 과정에서 사라지기 때문에 조치가 필요하다.

내가 사용한 방법은 `string`으로 넘겨줘서 해당 캐릭터안에 뼈대들을 찾아 넣어줬다.

> 기본적인 내용이지만 당연하듯 넘겨지지 않아 이틀을 해맸다,,

<br>

### 💡 참고
- [Unity Community Forums](https://forum.unity.com/threads/how-to-move-a-skinned-mesh-renderer-over-to-another-model.921854/){:target="_blank"}
- [[Unity] 캐릭터 파츠 교체하기 / 파츠 구조 변경 후 애니메이션 적용시키기](https://mingyu0403.tistory.com/248){:target="_blank"}