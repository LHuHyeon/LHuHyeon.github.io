---
title: C# ReaderWriterLock
author: LHH
date: 2023-03-03 20:30 GMT+0900
categories: [C#, Server]
tags: [Rookiss 강의, C#, Server]
---

## ReaderWriterLock
---
여러 스레드를 사용하다 보면 서로 공유되는 데이터가 존재한다. 

스레드들의 각 상황마다 읽거나 쓰는 일이 발생되고, 

대체적으로 읽는 상황이 더 많고 쓰는 상황은 빈번하게 일어난다.

<br>

`SpinLock`을 사용하는 예제에서 읽는 스레드와 쓰기 스레드가 대기를 한다 가정하면

읽기 스레드가 더 많이 수행하는 반면 

쓰기 스레드는 계속 대기하는 상황이 발생하게 되어 성능적으로 떨어지게 된다.

<br>

`ReaderWriterLock`은 데이터를 읽기 위해서 락을 설정하지 않고

쓰기 위해 접근할 때는 락을 설정하는 비대칭적인 락을 구현함으로써

계속 대기하는 `SpinLock` 상황보다 성능을 높일 수 있다.

## 💻 코드
---
간단하게 보상을 받을 때와 보상이 추가될 때의 코드 상황이다.
```cs
static ReaderWriterLockSlim _lock = new ReaderWriterLockSlim();

class Reward {}

// 보상 받기 ( 많이 쓰이는 곳 )
static Reward GetRewardBuId(int id)
{
    _lock4.EnterReadLock();

    _lock4.ExitReadLock();

    return null;
}

// 추가 보상 ( 정말 가끔 쓰일 곳 )
static void AddReward()
{
    _lock4.EnterWriteLock();

    _lock4.ExitWriteLock();
}
```

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4)
- [Rito15: ReaderWriterLock](https://rito15.github.io/posts/06-cs-reader-writer-lock/)