---
title: (C#) Lock과 Monitor Class
author: LHH
date: 2023-02-24 15:05 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server]
---

## Monitor 클래스 
기존 원자성을 보장하는 방법에서 Interlocked를 사용하였다.

하지만 이 것을 사용하기엔 +/- 밖에 사용을 못하는 단점이 있다.

Monitor 클래스의 사용법은 Monitor.Enter가 실행되면 다른 Monitor.Enter를 사용하는 스레드는 Exit가 실행될 때까지 기다리게 된다.

이렇게 되면 Enter와 Exit 사이는 싱글 스레드가 보장되는 셈이다. 

하지만 Monitor 클래스는 관리하기 어렵다는 단점이 있다.

만약 Moniter.Enter를 실행하고 Exit하기 전에 return을 해버린다면

다른 Enter를 기다리는 스레드들은 무한 대기에 빠지는 `데드락(DeadLock)`에 걸리게 된다.

이 때문에 Monitor 클래스를 사용하려면 코드를 더욱 신중하게 짜야한다.

```cs
// 예) 화장실문에 들어갔다 나오는 예를 생각하면 됨!
Monitor.Enter(_obj);    // 문을 잠구는 행위

// 이 안에서는 싱글 스레드가 보장됨!
number++;

Monitor.Exit(_obj);     // 잠금을 풀어준다.
```

```cs
// 데드락을 피하면서 코드를 짜면 지저분해지기 때문에 이를 방지하기 위해서 try, finally 사용
try
{
    Monitor.Enter(_obj);
    number++;

    return;
}
finally // try가 끝나면 무조건 실행함.
{
    Monitor.Exit(_obj);
}
```

## Lock이란?
데드락을 피하며 Monitor 클래스를 쓰기엔  코드가 지저분해진다.

대부분 Monitor 클래스보다 Lock을 사용하며 데드락을 조금이라도 줄일 수 있는 방법이다.

Lock을 사용하는 비중은 60~70%가 된다.

```cs
// [ lock의 간단한 원리 ]
// lock에 들어가면 Monitor.Enter가 실행되고
// 안에 있는 코드들이 다 실행되면
// Monitor.Exit를 실행한다.

static int number = 0;
static object _obj = new object();

static void Thread_1()
{
    for(int i=0; i<1000000; i++)
    {
        lock (_obj)
        {
            number++;
        }
    }
}

static void Thread_2()
{
    for (int i = 0; i < 1000000; i++)
    {
        lock(_obj)
        {
            number--;
        }
    }
}
```

## Lock 구현의 선택 방향
화장실에 사람이 들어가 있을 때 3가지 행동을 선택할 수 있다. 
1. 무작정 기다린다. <br> (너무 오래기달리면 비효율적임.)
2. 다시 자리로 돌아가 몇분 뒤 화장실로 간다. <br> (랜덤성이 강함.)
3. 나는 소중하니 직원에게 부탁 (갑질 메타). <br> (만약 100명의 사람들이 직원에게 부탁하면 상당히 짜증나고 힘들 것임) <br>

위 예시를 스레드에 적용시키면
1. 언제 끝날지 모르는 무한 체크를 계속 한다. <br> (스레드의 cpu의 점유율이 확 상승.)
2. 스레드의 소유권을 포기 후 몇초 후 재호출. <br> (이 덕분에 cpu 코어는 다른 스레드 사용 가능)
3. 스레드의 소유권을 다른 터널에게 맡김으로써 Exit가 되면 스레드에게 통보해주는 방법.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}