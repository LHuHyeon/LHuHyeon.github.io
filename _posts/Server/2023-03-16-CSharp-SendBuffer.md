---
title: C# SendBuffer
author: LHH
date: 2023-03-16 13:40 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Send]
---

지난 글([RecvBuffer](/posts/CSharp-RecvBuffer))를 만들었으니 이에 대칭적인 `SendBuffer`를 만들어본다.

## 💻 코드
`Send`는 `Recv`와 다르게 버퍼 사용량만 체크하면 되기 때문에 `_usedSize`로 사용량을 체크한다.

+ `SendBuffer`
  - `SendBuffer` : 버퍼를 설정
  - `Open` : 사용할 버퍼 용량을 받는다.
  - `Close` : 사용한 버퍼 길이만큼 `_usedSize`에 더해 체크한다.

```cs
public class SendBuffer
{
    byte[] _buffer;
    int _usedSize = 0;  // 사용한 버퍼 크기

    // 받을 수 있는 공간 반환
    public int FreeSize { get { return _buffer.Length - _usedSize; } }

    // 버퍼 설정
    public SendBuffer(int chunkSize)
    {
        _buffer = new byte[chunkSize];
    }

    // 사용할 버퍼 용량 받기
    public ArraySegment<byte> Open(int reserveSize)
    {
        if (reserveSize > FreeSize)
            return new ArraySegment<byte>();

        return new ArraySegment<byte>(_buffer, _usedSize, reserveSize);
    }

    // 사용한 버퍼만큼 커서 이동
    public ArraySegment<byte> Close(int usedSize)
    {
        ArraySegment<byte> segment = new ArraySegment<byte>(_buffer, _usedSize, usedSize);
        _usedSize += usedSize;
        return segment;
    }
}
```
<br>

`SendBuffer`를 관리하기 위해 `SendBufferHelper`를 구현한다.

+ `SendBufferHelper`
  - `CurrentBuffer` : 현재 사용할 수 있는 거대한 버퍼 
  - `ChunkSize` : 버퍼 크기
  - `Open` : Open에 대한 관리
  - `Close` : Close에 대한 관리

`CurrentBuffer`는 여러 스레드가 접근하여 사용하기 때문에 경합문제가 발생하지 않도록 `ThreadLoacl`을 사용한다.
```cs
// SendBuffer 도우미
public class SendBufferHelper
{
    // ThreadLocal을 사용했기 때문에 스레드끼리 경합이 일어나지 않음.
    public static ThreadLocal<SendBuffer> CurrentBuffer = new ThreadLocal<SendBuffer>(() => { return null; });

    // 버퍼 크기
    public static int ChunkSize { get; set; } = 4096 * 100;

    // 버퍼 사용
    public static ArraySegment<byte> Open(int reserveSize)
    {
        // 현재 버퍼가 없다면 만들어주기
        if (CurrentBuffer.Value == null)
            CurrentBuffer.Value = new SendBuffer(ChunkSize);

        // 현재 버퍼보다 더 큰 버퍼라면 새롭게 만들기
        if (CurrentBuffer.Value.FreeSize < reserveSize)
            CurrentBuffer.Value = new SendBuffer(ChunkSize);

        return CurrentBuffer.Value.Open(reserveSize);
    }

    // 버퍼 사용이 끝나면
    public static ArraySegment<byte> Close(int usedSize)
    {
        return CurrentBuffer.Value.Close(usedSize);
    }
}
```
<br>

완성한 `SendBufferHelper`는 이제 `ArraySegment`를 반환하기 때문에

`Session`에서 `byte[]`를 사용하던 `Send`관련 메소드를 모두 `ArraySegment<byte>`로 수정해준다.
```cs
// public void Send(byte[] sendBuff) {}
public void Send(ArraySegment<byte> sendBuff) {}

void RegisterSend()
{
    while (_sendQueue.Count > 0)
    {
      //byte[] buff = new byte[1024];
        ArraySegment<byte> buff = _sendQueue.Dequeue();
        _pendingList.Add(buff);
    }
}
```
<br>

`SendBufferHelper` 사용해본다.

기사의 hp, attack를 `Send` 하는 과정이다.
```cs
class Knight
{
    public int hp;
    public int attack;
    public string name;
}

class GameSession : Session
{
    public override void OnConnected(EndPoint endPoint)
    {
        Console.WriteLine($"OnConnected : {endPoint}");

        Knight knight = new Knight();

        // 4096 버퍼 사용하기
        ArraySegment<byte> openSegment = SendBufferHelper.Open(4096);

        // 데이터 바이트 형식으로 가져오기
        byte[] hpBuffer = BitConverter.GetBytes(knight.hp);
        byte[] atkBuffer = BitConverter.GetBytes(knight.attack);

        // sendBuffer에 복사.
        Array.Copy(hpBuffer, 0, openSegment.Array, openSegment.Offset, hpBuffer.Length);
        Array.Copy(atkBuffer, 0, openSegment.Array, openSegment.Offset + hpBuffer.Length, atkBuffer.Length);

        // 얼마나 썼는지 알려주기
        ArraySegment<byte> sendBuff = SendBufferHelper.Close(hpBuffer.Length + atkBuffer.Length);

        Send(sendBuff);
        Thread.Sleep(1000);
        Disconnect();
    }
}
```

## 결과
![server 출력](https://user-images.githubusercontent.com/110723307/225594923-61cca1ce-d1f9-4095-9ade-b845557c43bb.PNG)

정상 작동이되고, `Transferred Bytes`를 확인해보면 int(4byte)를 2개 보냈기 때문에 8 출력값이 나온 것을 알 수 있다.

## 📝 참고 글
- `RecvBuffer _recvBuffer = new RecvBuffer(1024);` 처럼 Send도 Session 내부에 미리 버퍼를 생성하면 되지 않나?
  - 이렇게 구현하면 성능적으로 효율이 떨어지게된다.
  - Session 위치가 아닌 외부에서 버퍼를 생성하고 보내는 것이 더 효율적이다.
  - 만약 100명이라는 플레이어가 서로에게 여러 데이터를 보낸다면 `Session`안에 `sendbuffer`는 복사하며 사용될 것이다.
  - 하지만 `sendBuffer`를 외부적으로 구현해주면 한번만 생성하고 반복문 같은 곳에서 계속 사용되기 때문에 성능상 유리하다.

- 버퍼 사용 : 버퍼는 그때그때 정하여 사용하기 어렵기 때문에 미리 커다란 버퍼를 만들어 짤라쓰는게 좋다.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}