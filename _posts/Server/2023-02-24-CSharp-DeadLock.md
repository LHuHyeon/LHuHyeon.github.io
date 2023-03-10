---
title: C# DeadLock
author: LHH
date: 2023-02-24 20:50 GMT+0900
categories: [C#, Server]
tags: [Rookiss 강의, C#, Server]
---

## lock 2개일 때 데드락 현상
만약 A와 B라는 스레드와 lock_1과 lock_2가 존재할 때

A는 lock_1 -> lock_2 순서로 Enter를 하고

B는 lock_2 -> lock_1 순서로 Enter를 동시에 실행한다 가정하면

A는 lock_2를 기다리고 B는 lock_1을 기다리는 데드락 현상이 생기게 된다.

## 💻 코드
```cs
class SessionManager
{
    static object _lock = new object();

    public static void TestSesstion()
    {
        lock (_lock)
        {

        }
    }

    public static void Test()
    {
        lock (_lock)
        {
            UserManager.TestUser();
        }
    }
}
```

```cs
class UserManager
{
    static object _lock = new object();

    public static void TestUser()
    {
        lock (_lock)
        {

        }
    }

    public static void Test()
    {
        lock (_lock )
        {
            SessionManager.TestSesstion();
        }
    }
}
```

```cs
class Program
{
    static void Thread_1()
    {
        for(int i=0; i<10000; i++)
        {
            SessionManager.Test();
        }
    }

    static void Thread_2()
    {
        for (int i = 0; i < 10000; i++)
        {
            UserManager.Test();
        }
    }
}
```

## 코드 설명
위와 같이 코드가 동시에 진행되면 다음과 같이 진행되고, 데드락에 걸릴 확률이 높아진다.
- SessionManager.Test() -> lock -> UserManager.TestUser() -> lock;
- UserManager.Test() -> lock -> SessionManager.TestSession() -> lock;
<br>

하지만 데드락은 스레드끼리 시간만 어긋난다면 데드락현상은 일어나지 않는다.

이를 방지하기 위해 try, catch 등 여러 대응 방법이 있지만

데드락을 예상하여 계속 피하게 되면 코드는 더욱 어수선해질 뿐이다.

## 결론
데드락 현상은 까다롭기 때문에 사실상 완저히 막을 방법은 없다.

데드락이 일어날거 같아 계속 코드를 길게 대응하기보다

차라리 데드락이 일어나고 고치는게 더 쉬운 방법일 것이다.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4)