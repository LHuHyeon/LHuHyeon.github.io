---
title: C# ReaderWriterLock 구현 연습
author: LHH
date: 2023-03-05 01:50 GMT+0900
categories: [C#, Server]
tags: [Rookiss 강의, C#, Server]
---

[ReaderWriterLock](/posts/CSharp-ReaderWriterLock)을 어떤식으로 동작하는지 구현해본다.

(주석 확인) <br>
먼저 Lock 클래스를 생성하고 다음과 같이 변수를 선언한다.
```cs
class Lock
{
    const int EMPTY_FLAG = 0x00000000;
    const int WRITE_MASK = 0x7FFF0000;  // 쓰기
    const int READ_MASK = 0x0000FFFF;   // 읽기
    const int MAX_SPIN_COUNT = 5000;    // Spin 최대 횟수

    // [Unused(1)] [WriteThreadId(15)] [ReadCount(16)]
    int _flag;

    // 쓰레드가 몇번의 WriteLock을 흭득하는지 확인
    int _writeCount = 0;
}
```
<br>

_flag는 32비트 안에서 각각 Unused(1), WriteThreadId(15), ReadCount(16)로 사용될 것이다.

WriteLock은 어떠한 스레드도 WriteLock과 ReadLock을 흭득하지 않았을 때 경합해서 소유권을 얻는다.

WriteLock은 현재 들어온 스레드의 ID를 가져와서 다른 스레드가 사용중이 아니라면 _flag에 넣는다.

또한 현재 사용중인 스레드가 또 사용한다면 _writeCount의 횟수를 증가시킨다.
```cs
public void WriteLock()
{
    // 동일 쓰레드가 WriteLock을 이미 흭득하고 있는지 확인
    int lockThreadID = (_flag & WRITE_MASK) >> 16;
    if (lockThreadID == Thread.CurrentThread.ManagedThreadId)
    {
        _writeCount++;
        return;
    }

    // desired : 스레드 id를 16진수로 변환 -> WRITE_MASK와 같은 위치에 값을 반환
    int desired = (Thread.CurrentThread.ManagedThreadId << 16) & WRITE_MASK;

    while (true)
    {
        // 최대 Spin 횟수만큼 돌기
        for(int i=0; i<MAX_SPIN_COUNT; i++)
        {
            // 다른 스레드가 사용중이 아니라면 _flag에 desired(ThreadID) 넣기
            if (Interlocked.CompareExchange(ref _flag, desired, EMPTY_FLAG) == EMPTY_FLAG)
            {
                _writeCount = 1;
                return;
            }
        }

        // 안되면 잠시 기다렸다 오기.
        Thread.Yield();
    }
}

public void WriteUnLock()
{
    // 현재 사용되는 스레드의 WriteLock 사용 끝내기 (횟수 감소)
    int lockCount = --_writeCount;

    if (lockCount == 0)
        Interlocked.Exchange(ref _flag, EMPTY_FLAG);
}
```
<br>

ReadLock 구현도 WriteLock과 비슷하다.

ReadLock은 독점권이 아닌 다른 스레드들끼리 동시에 사용 가능하기 때문에

_flag의 READ_MASK 자리를 가져와서 횟수만 +1 증가시킨다.

또한 WriteLock을 사용하는 스레드가 동시에 ReadLock을 사용할 수 있기 때문에

쓰레드 ID도 확인해주도록 한다.
```cs
public void ReadLock()
{
    while (true)
    {
        // 동일 쓰레드가 WriteLock을 이미 흭득하고 있는지 확인
        int lockThreadID = (_flag & WRITE_MASK) >> 16;
        if (lockThreadID == Thread.CurrentThread.ManagedThreadId)
        {
            Interlocked.Increment(ref _flag);
            return;
        }

        for (int i = 0; i < MAX_SPIN_COUNT; i++)
        {
            // _flag에 READ_MASK 자리 추출 후 사용.
            int expected = (_flag & READ_MASK);
            if (Interlocked.CompareExchange(ref _flag, expected+1, expected) == expected)
                return;
        }

        Thread.Yield();
    }
}

public void ReadUnLock()
{
    Interlocked.Decrement(ref _flag);
}
```
<br>

## 사용하기
```cs
static volatile int _count = 0;
static Lock _lock = new Lock();

static void Main(string[] args)
{
    // 메소드 구현이 귀찮으니 Delegate로 구현
    Task t1 = new Task(delegate ()
    {
        for(int i=0; i<1000000; i++)
        {
            _lock.WriteLock();
            _count++;
            _lock.WriteUnLock();
        }
    });

    Task t2 = new Task(delegate ()
    {
        for (int i = 0; i < 1000000; i++)
        {
            _lock.WriteLock();
            _count--;
            _lock.WriteUnLock();
        }
    });

    t1.Start();
    t2.Start();

    Task.WaitAll(t1, t2);

    Console.WriteLine(_count);
}
```
<br>

이렇게 ReaderWriterLock을 구현함으로써 구체적인 코드 구성을 이해할 수 있었다.

특히 `if (Interlocked.CompareExchange(ref _flag, desired, EMPTY_FLAG) == EMPTY_FLAG)`는 

이해가 어려울 수 있으므로 더욱 분석하여 공부하도록 한다.

### 전체 코드
<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
class Lock
{
    const int EMPTY_FLAG = 0x00000000;
    const int WRITE_MASK = 0x7FFF0000;
    const int READ_MASK = 0x0000FFFF;
    const int MAX_SPIN_COUNT = 5000;

    // [Unused(1)] [WriteThreadId(15)] [ReadCount(16)]
    int _flag;
    int _writeCount = 0;

    // 아무도 WriteLock or ReadLock을 흭득하지 않고 있을 때, 경합해서 소유권을 얻는다.
    public void WriteLock()
    {
        // 동일 쓰레드가 WriteLock을 이미 흭득하고 있는지 확인
        int lockThreadID = (_flag & WRITE_MASK) >> 16;
        if (lockThreadID == Thread.CurrentThread.ManagedThreadId)
        {
            _writeCount++;
            return;
        }

        // desired : 스레드 id를 16진수로 변환 -> WRITE_MASK와 같은 위치에 값을 반환
        int desired = (Thread.CurrentThread.ManagedThreadId << 16) & WRITE_MASK;

        while (true)
        {
            for(int i=0; i<MAX_SPIN_COUNT; i++)
            {
                if (Interlocked.CompareExchange(ref _flag, desired, EMPTY_FLAG) == EMPTY_FLAG)
                {
                    _writeCount = 1;
                    return;
                }
            }

            Thread.Yield();
        }
    }

    public void WriteUnLock()
    {
        int lockCount = --_writeCount;

        if (lockCount == 0)
            Interlocked.Exchange(ref _flag, EMPTY_FLAG);
    }

    public void ReadLock()
    {
        int lockThreadID = (_flag & WRITE_MASK) >> 16;
        if (lockThreadID == Thread.CurrentThread.ManagedThreadId)
        {
            Interlocked.Increment(ref _flag);
            return;
        }

        while (true)
        {
            for (int i = 0; i < MAX_SPIN_COUNT; i++)
            {
                int expected = (_flag & READ_MASK);
                if (Interlocked.CompareExchange(ref _flag, expected+1, expected) == expected)
                    return;
            }

            Thread.Yield();
        }
    }

    public void ReadUnLock()
    {
        Interlocked.Decrement(ref _flag);
    }
}

class Program
{
    static volatile int _count = 0;
    static Lock _lock = new Lock();

    static void Main(string[] args)
    {
        Task t1 = new Task(delegate ()
        {
            for(int i=0; i<1000000; i++)
            {
                _lock.WriteLock();
                _count++;
                _lock.WriteUnLock();
            }
        });

        Task t2 = new Task(delegate ()
        {
            for (int i = 0; i < 1000000; i++)
            {
                _lock.WriteLock();
                _count--;
                _lock.WriteUnLock();
            }
        });

        t1.Start();
        t2.Start();

        Task.WaitAll(t1, t2);

        Console.WriteLine(_count);
    }
}
```

</div>
</details>

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4)