---
title: C# 컨텍스트 스위칭과 Event
author: LHH
date: 2023-03-02 15:30 GMT+0900
categories: [C#, Server]
tags: [Rookiss 강의, C#, Server]
---

## 컨텍스트 스위칭
컨텍스트 스위칭은 여러 프로세스가 실행되고 있을 때 기존에 실행되던 프로세스를 중단하고

다른 프로세스로 옮겨지는 중에 생기는 현상을 말한다.

실행 중인 프로세스가 변경되면 CPU의 레지스터 값을 변경하는데,

변경되기 전에 이전 프로세스가 지니던 데이터를 메모리에 저장시켜줘야 한다.

`실행되는 프로세스의 변경 과정에서 발생하는 컨텍스트 스위칭은 시스템에 부담을 준다.`

## 💻 Event 코드
### Auto Reset Event (톨게이트 개념)
<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
class Lock
{
    // bool <- 커널
    AutoResetEvent _available = new AutoResetEvent(true);    // 애를 쓰는게 더 현명함!

    public void Acquire()
    {
        // 톨게이트를 통과하면 문이 자동으로 닫히는 개념
        _available.WaitOne();   // 입장 시도 ( _available.Reset(); 포함됨.)
    }

    public void Release()
    {
        _available.Set();       // 문을 열어준다.
    }
}
```

</div>
</details>

### Manual Reset Event (방에 있는 문 개념)
<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
class Lock
{
    ManualResetEvent _available = new ManualResetEvent(true);

    public void Acquire()
    {
        // 방문에 들어가면 수동으로 닫는 개념
        _available.WaitOne();   // 입장 시도
        _available.Reset();     // 문을 닫는다.
    }

    public void Release()
    {
        _available.Set();       // 문을 열어준다.
    }
}
```

</div>
</details>

<br>

## AutoResetEvent와 ManualResetEvent 차이점
`AutoResetEvent`는 .WaitOne() 실행 시 문도 같이 닫아주는 반면

`ManualResetEvent`는 .Reset()도 작성해줘야 한다.

이로 인해 `AutoResetEvent`는 결과가 정상적으로 나오지만

`ManualResetEvent`는 두줄의 코드로 입장과 문을 닫기 때문에 다른 결과가 나온다.

그러므로 `AutoResetEvent`를 사용하는 것이 더 현명하다.

## Mutex
<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
static int _num = 0;
static Mutex _lock = new Mutex();

static void Thread_1()
{
    for(int i=0; i<1000000; i++)
    {
        _lock.WaitOne();
        _num++;
        _lock.ReleaseMutex();
    }
}

static void Thread_2()
{
    for(int i=0; i<1000000; i++)
    {
        _lock.WaitOne();
        _num--;
        _lock.ReleaseMutex();
    }
}
```

</div>
</details>

### Mutex 사용 이유
`Mutex`는 앞서 말한 Event와 기능면에서 다를 바가 없지만

int, bool, ThreadID 등 여러 정보를 제공한다.

이렇기 때문에 비용면에서 부담이 많이되기 때문에

이런게 있다 참고하고 `AutoResetEvent`를 사용하는게 더 좋다.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}
- [은비로운 개발일기: Context Switching](https://agh2o.tistory.com/12){:target="_blank"}