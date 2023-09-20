---
title: (Unity) Animator Controller 코드 변경
author: LHH
date: 2023-09-20 22:00 GMT+0900
categories: [Study, Unity]
tags: [Unity, Animator]
---

애니메이션 컨트롤러를 변경하는 방법이다.

```cs

public class AnimTest : MonoBehaviour
{
    [SerializeField]
    private RuntimeAnimatorController animatorContoller;    // Animator Controller

    private Animator _anim;

    void Start()
    {
        _anim = GetComponent<Animator>();
        
        // 컨트롤러 변경
        _anim.runtimeAnimatorController = animatorContoller;

        // or Resources에서 불러오기
        _anim.runtimeAnimatorController = Resources.Load<RuntimeAnimatorController>(animatorControllerPath);
    }
}

```