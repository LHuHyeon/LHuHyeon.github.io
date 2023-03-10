---
title: C# Memory Barrier
author: LHH
date: 2023-02-23 11:40 GMT+0900
categories: [C#, Server]
tags: [Rookiss 강의, C#, Server]
---

## Memory Barrier란?
컴파일을 진행하게 되면 우리 모르게 하드웨어에서도 최적화를 진행하게 된다.

그로 인해 코드 재배치가 일어나게 되고

이런 최적화 때문에 우리가 의도하지 않는 방향으로 코드가 흘러가게 된다.

우리의 의도대로 코드가 진행할 수 있도록 사용되는 것이 바로 메모리 베리어(Memory Barrier)이다.

### Store, Load란?
- Store : a = 1 &nbsp;<-&nbsp; 변수에 값을 넣는 것.
- Load  : b = a &nbsp;<-&nbsp; a라는 값의 변수를 불러오는 것.

### Memory Barrier 종류
- 1) `Full Memory Barrier` (ASM MFENCE, C# Thread.MemoryBarrier) : Store/Load 둘다 막는다.
- 2) `Store Memory Barrier` (ASM SFENCE) : Store만 막는다.
- 3) `Load Memory Barrier` (ASM LFENCE)  : Load만 막는다.

## 💻 코드
코드의 구조상 무한루프를 탈출할 수 없게 만들어진 코드이다. 하지만 실행을 해보면 탈출된 것을 알 수 있다. <br>
탈출된 이유는 하드웨어가 최적화를 진행했기 때문이다.
```cs
internal class Program
{
    static int x = 0;
    static int y = 0;
    static int r1 = 0;
    static int r2 = 0;

    static void Thread_1()
    {
        y = 1;      // Store y
        r1 = x;     // Load x
    }

    static void Thread_2()
    {
        x = 1;      // Store x
        r2 = y;     // Load y
    }

    static void Main(string[] args)
    {
        int count = 0;
        while (true)
        {
            count++;
            x = y = r1 = r2 = 0;

            Task t1 = new Task(Thread_1);
            Task t2 = new Task(Thread_2);
            t1.Start();
            t2.Start();

            // t1, t2 쓰레드가 끝날때까지 기다림.
            Task.WaitAll(t1, t2);

            if (r1 == 0 && r2 == 0)
                break;
        }
        Console.WriteLine($"{count}번만에 빠져나옴!");

        // 탈출되는 이유 : 하드웨어도 최적화를 진행함!
    }
}
```

하드웨어 최적화를 피할 수 있도록 아래의 코드처럼 스레드 메소드에 Memory Barrier 적용시킨다. <br>
실행을 해보면 무한루프에서 탈출하지 못한 것을 볼 수 있다.
```cs
// 메모리 베리어
// A) 코드 재배치 억제 ( Thread.MemoryBarrier(); )
// B) 가시성

static void Thread_1()
{
    y = 1;      // Store y

    Thread.MemoryBarrier(); // 위 아래 코드가 서로 뒤바뀔 수 없게 억제함.

    r1 = x;     // Load x
}

static void Thread_2()
{
    x = 1;      // Store x

    Thread.MemoryBarrier();

    r2 = y;     // Load y
}
```

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4)