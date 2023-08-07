---
title: (Unity) 디자인패턴 Event Bus
author: LHH
date: 2023-08-07 17:05 GMT+0900
categories: [Study, Unity]
tags: [Unity, 디자인패턴, Event Bus]
---

> 유니티로 배우는 게임 디자인 패턴 책을 정리한 글입니다.

`Event Bus`는 객체가 구독하거나 게시할 수 있는 특정한 전역 이벤트의 목록을 관리하는 중앙 허브 역할을 한다.

이벤트 관리와 관련된 가장 간단한 패턴으로 한 줄의 코드로 객체에 구독자 혹은 게시자의 역할을 할당하는 과정을 줄인다.

## Event Bus 장단점
### 장점
- 분리 : 핵심은 오브젝트의 분리이다. 오브젝트는 직접 서로를 참조하는 대신 이벤트로 통신할 수 있다.
- 단순성 : 이벤트의 구독 혹은 게시 메커니즘을 추상화하여 단순성을 제공한다.

### 단점
- 성능 : 모든 이벤트 시스템의 내부에는 오브젝트 간 메시지를 관리하는 저수준 메커니즘이 있다. 이 때문에 약간의 성능 비용이 발생할 수 있다.

## Event Bus 코드
```cs
using UnityEngine.Events;
using System.Collections.Generic;

public enum FunctionType { START, RESTART, PAUSE, STOP, FINISH, QUIT }

public class EventBus
{
    // 각 타입마다 Event를 저장
    private static readonly Dictionary<FunctionType, UnityEvent> Events = new Dictionary<FunctionType, UnityEvent>();

    // 해당 타입에 listener 추가
    public static void Subscribe(FunctionType eventType, UnityAction listener)
    {
        UnityEvent thisEvent;

        if (Events.TryGetValue(eventType, out thisEvent))
        {
            thisEvent.AddListener(listener);
            return;
        }

        thisEvent = new UnityEvent();
        thisEvent.AddListener(listener);
        Events.Add(eventType, thisEvent);
    }

    // 해당 타입에서 listener 삭제
    public static void Unsubscribe(FunctionType type, UnityAction listener) 
    {
        UnityEvent thisEvent;

        if (Events.TryGetValue(type, out thisEvent))
            thisEvent.RemoveListener(listener);
    }

    // 해당 타입안에 들어있는 Event 실행
    public static void Publish(FunctionType type)
    {
        UnityEvent thisEvent;

        if (Events.TryGetValue(type, out thisEvent))
            thisEvent.Invoke();
    }
}
```

- `Subscribe()` : listener를 Event에 추가한다.
- `Unsubscribe()` : listener를 Event에서 삭제시킨다.
- `Publish()` : Event를 실행한다.

<br>

**간단 요약** : `Event Bus`는 어떤 상황에서 동시에 실행되어야할 기능이 여러개라면 `Event Bus`를 통해 단순하게 구현할 수 있다.