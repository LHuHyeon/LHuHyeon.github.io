---
title: C# Thread Local Storage (TLS)
author: LHH
date: 2023-03-06 00:10 GMT+0900
categories: [C#, Server]
tags: [Rookiss 강의, C#, Server, TLS]
---

## TLS란?
TLS는 Thread Local Storage의 약자로 각 스레드마다 저장공간을 가지는 방법이다.

정적인 전역 변수가 존재할 때 여러 스레드가 그 변수를 사용한다면 독점권으로 인해

Lock을 걸었다 풀었다하기 때문에 싱글스레드보다 오히려 비효율적으로 사용되게 된다.

TLS는 그러한 변수를 각 스레드에게 자신만의 공간에서 다른 스레드의 간섭이 없도록 사용할 수 있게 해준다.

## 💻 코드
TLS 사용방법은 ThreadLocal 클래스를 사용하여 정적인 변수를 스레드가 사용했을 때 자신만의 공간에서 사용이 가능하다.

아래 코드는 WhoAmI를 여러 스레드가 동시에 접근하여 사용했을 때

각 스레드에서 threadName의 ID가 몇으로 나오는지 확인하는 코드이다.
```cs
static ThreadLocal<string> threadName = new ThreadLocal<string>();

static void WhoAmI()
{
    // ThreadLocal에 값을 넣을때는 .Value하여 쓰거나 읽을 수 있다.
    threadName.Value = $"My Name Is {Thread.CurrentThread.ManagedThreadId}";

    Thread.Sleep(1000);

    Console.WriteLine(threadName.Value);
}

static void Main(string[] args)
{
    // Task를 여러개 만들어서 한번에 호출하기 번거로우니 Parallel 사용.
    Parallel.Invoke(WhoAmI, WhoAmI, WhoAmI, WhoAmI, WhoAmI, WhoAmI, WhoAmI, WhoAmI, WhoAmI);
}
```

![](https://user-images.githubusercontent.com/110723307/222966552-db963498-0ec6-4174-97ba-1b6d29043c2a.PNG)

<br>

컴파일을 해보면 threadName을 사용하는 스레드 ID가 전부 다른것을 확인할 수 있다.

이는 스레드가 자신만의 공간에서 외부 간섭없이 threadName을 사용한 것이다.

### 🛠 ThreadLocal을 사용안했을 때
만약 ThreadLocal을 사용하지 않고 정적인 string 변수로 사용할 경우 결과는 다음과 같다.

```cs
// static ThreadLocal<string> threadName = new ThreadLocal<string>();
static string threadName;
```

![](https://user-images.githubusercontent.com/110723307/222967220-6bd3f5b9-f9ce-4114-970a-c4daa2f5917a.PNG)

<br>

컴파일 결과 스레드 ID가 모두 같은것을 확인할 수 있다.

### 🛠 스레드 개수를 정해줬을 때
만약 ThreadPool을 사용하여 스레드의 수를 제안했을 때 결과를 확인해본다.
```cs
ThreadPool.SetMinThreads(1, 1);
ThreadPool.SetMaxThreads(3, 3);
```

![](https://user-images.githubusercontent.com/110723307/222967952-9fda9adf-4990-44bb-a195-a7030fb03089.PNG)

<br>

결과를 확인해보면 스레드 개수를 제안했을 뿐인데 스레드 ID가 중복되어 출력된 것을 볼 수 있다.

하지만 스레드 ID 출력값이 같아 TLS가 제대로 작동안했나 할 수 있지만 TLS는 정상적으로 작동하였다.

다음 코드처럼 수정하여 다시 확인해보도록 한다.

```cs
// threadName을 스레드가 사용하면 ThreadLocal을 생성할 때 매개변수로 전달된 값이 Value에 저장된다.
static ThreadLocal<string> threadName = new ThreadLocal<string>(() => { return $"My Name Is {Thread.CurrentThread.ManagedThreadId}"; });

static void WhoAmI()
{
    // .IsValueCreadted를 통해 재사용 됐는지 확인
    bool repeat = threadName.IsValueCreated;

    if (repeat)
        Console.WriteLine(threadName.Value + "(repeat)");
    else
        Console.WriteLine(threadName.Value);
}

static void Main(string[] args)
{
    ThreadPool.SetMinThreads(1, 1);
    ThreadPool.SetMaxThreads(3, 3);
    Parallel.Invoke(WhoAmI, WhoAmI, WhoAmI, WhoAmI, WhoAmI, WhoAmI, WhoAmI, WhoAmI, WhoAmI);

    // threadName 값 초기화
    threadName.Dispose();
}
```

![](https://user-images.githubusercontent.com/110723307/222968322-f1d88497-963a-424d-9b42-da231382322f.PNG)

<br>

결과를 보면 같은 스레드ID를 재사용했을 때 `(repeat)`가 붙어있는 것을 확인할 수 있다.

## 결론
코드 실습을 통해 TLS 사용법을 알 수 있었고 각 스레드의 공간에서 정적인 변수를 사용할 수 있다는 것을 확인할 수 있었다.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4)
- [egloos: TLS (Thread Local Storage)](http://egloos.zum.com/sweeper/v/1985738)