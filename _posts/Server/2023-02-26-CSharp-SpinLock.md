---
title: C# SpinLock
author: LHH
date: 2023-03-02 14:40 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server]
---

## SpinLock이란?
SpinLock은 만약 다른 스레드가 lock을 소유하고 있다면 그 lock이 반환될 때까지 계속 확인하며 기다린다.

## 💻 코드

<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
class SpinLock
{
    volatile bool _locked = false;

    // Enter
    public void Acquire()
    {
        // 잠김이 풀리기를 기다린다.
        while (_locked == true)
        {
            
        }

        // 내꺼!
        _locked = true;
    }

    // Exit
    public void Release()
    {
        _locked = false;
    }
}

class Program
{
    static int _num = 0;
    static SpinLock _lock = new SpinLock();

    static void Thread_1()
    {
        for(int i=0; i<100000; i++)
        {
            _lock.Acquire();
            _num++;
            _lock.Release();
        }
    }

    static void Thread_2()
    {
        for (int i = 0; i < 100000; i++)
        {
            _lock.Acquire();
            _num--;
            _lock.Release();
        }
    }

    static void Main(string[] args)
    {
        Task t1 = new Task(Thread_1);
        Task t2 = new Task(Thread_2);

        t1.Start();
        t2.Start();

        Task.WaitAll(t1, t2);

        Console.WriteLine(_num);
    }
}
```

</div>
</details>

다음 코드와 같이 실행하게 되면 _num의 값은 0이 아니라 아에 다른값이 나오는 것을 확인할 수 있다. <br>
이러한 문제를 해결하기 위해 두가지 방법이 있다.
- `Interlocked.Exchange(ref location, value);`
- `Interlocked.CompareExChange(ref location, desired, expected);`


## Interlocked.Exchange
`Interlocked.Exchange`를 코드로 풀어보면 아래와 같은 느낌이다. <br>
```cs
public void Acquire()
{
    while (true)
    {
        int original = _locked;
        _locked = 1;
        if (original == 0)
            break;
    }
}
```
하지만 이렇게 구현 한다면 두줄의 코드를 거쳐 실행되기 때문에 공동으로 사용되는 _locked가 다른 스레드에서 사용될 수도 있다.

이 때문에 한줄로 구현해줄 필요가 있으며 구현 방법은 다음과 같다.

<br>

```cs
public void Acquire()
{
    while (true)
    {
        int original = Interlocked.Exchange(ref _locked, 1);
        if (original == 0)
            break;
    }
}
```
`Interlocked.Exchange(ref _locked, 1)`가 실행되면 _locked에 1을 넣게된다.

`Interlocked.Exchange`의 반환값은 _locked에 1이 들어가기 전의 값을 반환해준다.

## Interlocked.CompareExchange
```cs
public void Acquire()
{
    while (true)
    {
        // CAS Compare-And-Swap
        // Interlocked.CompareExchange를 풀어보면 아래와 같은 느낌이다.
        // if (_locked == 0)
        //     _locked = 1;
        // 인자값을 넘겨줄 때 0, 1로 넘겨줘도 되지만 이해하기 슆게 expected, desired를 선언했다.
        int expected = 0;
        int desired = 1;
        if (Interlocked.CompareExchange(ref _locked, desired, expected) == expected)
            break;
    }
}
```
`Interlocked.CompareExChange`는 `Interlocked.Exchange`와 비슷하지만 조금 더 정교하다.

사용법은 _locked의 이전값이 expected라면 _locked에 desired 값을 반환한다.

리턴값은 Exchange와 똑같이 이전 값을 리턴한다.

<br>

## 기다린 후 재요청하는 3가지 방법.
1. Thread.Sleep(1); : 무조건 휴식 -> 무조건 1ms 정도 쉰다.
2. Thread.Sleep(0); : 조건부 양보 -> 나보다 우선순위가 낮은 애들한테는 양보 불가 -> 우선순위가 나보다 같거나 높은 쓰레드가 없으면 본인이 선택됨.
3. Thread.Yield(); : 관대한 양보 -> 관대하게 양보할테니, 지금 실행이 가능한 스레드가 있으면 실행 -> 실행 가능한 애가 없으면 남은 시간 소진.

```cs
volatile bool _locked = false;

// Enter
public void Acquire()
{
    while (true)
    {
        if (Interlocked.CompareExChange(ref _locked, 1, 0) == 0)
            break;

        // 상황에 맞게 사용.
        Thread.Sleep(1); 
        Thread.Sleep(0); 
        Thread.Yield();       
    }
}
```
기본적으로 OS 단에 요청하는 거의 모든 API (Console.Write, Sleep 등)은

CPU 사용권을 일단 반납하고 운영체제 쪽에서 다음 처리를 판별하게 되기 때문에 [Context Switching](/posts/CSharp-컨텍스트-스위칭과-Event/)이 일어난다.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}
- [Rookiss 문의 답변](https://www.inflearn.com/questions/36808/sleep%EA%B3%BC-context-switching%EC%97%90-%EB%8C%80%ED%95%B4){:target="_blank"}
- [불곰: SpinLock](https://brownbears.tistory.com/45){:target="_blank"}
- [rito15: SpinLock](https://rito15.github.io/posts/03-cs-spinlock/){:target="_blank"}