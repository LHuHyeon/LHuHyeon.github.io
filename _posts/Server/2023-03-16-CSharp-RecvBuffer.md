---
title: C# RecvBuffer
author: LHH
date: 2023-03-16 13:40 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Recv]
---

클라이언트에게 버퍼를 받아서 관리해줄 `RecvBuffer Class`를 구현할 것이다.

만약 패킷을 받았을 때 버퍼 100중에서 80 정도만 왔다면 이것을 읽으면 안된다.

`RecvBuffer Class`는 이를 확인하기 위해 100 전체가 올때까지 기다리고 읽게 만들 것이다.

## 💻 코드
`RecvBuffer Class`를 만들고 변수를 선언해준다.

`_writePos`는 클라이언트로 받은 버퍼의 크기만큼 움직인다.

`_readPos`는 `_writePos`가 이동한 위치가 받은 버퍼의 끝자리이기 때문에 그 안에서의 버퍼를 확인하고 읽는다.
```cs
ArraySegment<byte> _buffer;
int _readPos;   // 읽을 버퍼의 시작 위치
int _writePos;  // 받은 버퍼의 마지막 위치
```

예를 들어서 10 버퍼 공간이 존재한다면 시작은 다음과같다. (r : Read, w : Write)

> `[rw][][][][][][][][][]`

만약 클라이언트에게 5라는 버퍼를 받는다면 w는 5만큼 움직일 것이다.

> `[r][][][][][w][][][][]`

r은 읽을 수 있는 범위가 w만큼 있기 때문에 이 안에서 버퍼를 읽을 수 있는지 확인한다.

만약 w까지 읽을 수 있다면 r은 읽고 자리를 이동한다.

> `[][][][][][rw][][][][]`

<br>

첫 버퍼를 세팅할 때 클래스 생성에서 버퍼를 전달 받을 수 있게 할 것이다.

다음 코드와 같이 필요한 메소드를 구현한다.
```cs
// 버퍼 사이즈 저장
public RecvBuffer(int bufferSize)
{
    _buffer = new ArraySegment<byte>(new byte[bufferSize], 0, bufferSize);
}

// 읽을 버퍼 크기 구하기
public int DataSize { get { return _writePos - _readPos; } }

// 받을 수 있는 버퍼 크기 구하기
public int FreeSize { get { return _buffer.Count - _writePos; } }

// 어디부터 읽으면 되는지? 전달 ( 버퍼 전달 )
public ArraySegment<byte> ReadSegment
{
    get { return new ArraySegment<byte>(_buffer.Array, _buffer.Offset + _readPos, DataSize); }
}

// 읽을 때
public bool OnRead(int numOfBytes)
{
    if (numOfBytes > DataSize)
        return false;

    _readPos += numOfBytes;
    return true;
}

// 받을 때
public bool OnWrite(int numOfBytes)
{
    if (numOfBytes > FreeSize)
        return false;

    _writePos += numOfBytes;
    return true;
}
```
<br>

버퍼를 읽을 때 세팅한 최대 길이를 초과하는 일이 없도록 위치 리셋시키는 메소드를 구현한다.
```cs
// 커서 위치 리셋 ( read, write의 위치가 계속 올라가면 버퍼 크기를 초과하기 때문 )
public void Clean()
{
    int dataSize = DataSize;
    if (dataSize == 0)
    {
        // 남은 데이터가 없으면 복사하지 않고 커서 위치만 리셋
        _readPos = _writePos = 0;
    }
    else
    {
        // 남은 데이터가 있으면 시작 위치로 복사
        // Array.Copy(복사할 버퍼, 복사할 시작 위치, 붙여넣기할 버퍼, 붙여넣기할 시작 위치, 복붙할 크기);
        Array.Copy(_buffer.Array, _buffer.Offset + _readPos, _buffer.Array, _buffer.Offset, dataSize);
        _readPos = 0;
        _writePos = dataSize;
    }
}
```
<br>

이제 완성된 `RecvBuffer Class`를 `Session`에 적용시킨다.

`RegisterRecv`는 먼저 버퍼를 리셋해주고, 받을 수 있는 버퍼를 가져와 버퍼를 세팅한다.

`OnRecvCompleted`는 성공했으니 `OnWrite` -> 컨텐츠 진행(`OnRecv`) -> `OnRead` 순서로 진행해준다.

각각 실행할 때 에러도 확인해야 되기 때문에 if문으로 false일 경우 쫓아내도록 한다.
```cs
// 1024만큼 버퍼 생성
RecvBuffer _recvBuffer = new RecvBuffer(1024);

// Recv 등록
void RegisterRecv()
{
    // 리셋 후 버퍼 설정 진행
    _recvBuffer.Clean();
    ArraySegment<byte> segment = _recvBuffer.WriteSegment;
    _recvArgs.SetBuffer(segment.Array, segment.Offset, segment.Count);

    bool pending = _socket.ReceiveAsync(_recvArgs);
    if (pending == false)
        OnRecvCompleted(null, _recvArgs);
}

// Recv 성공
void OnRecvCompleted(object sender, SocketAsyncEventArgs args)
{
    // Byte가 0으로 올때도 있기 때문에 0 이상으로 받기
    if (args.BytesTransferred > 0 && args.SocketError == SocketError.Success)
    {
        try
        {
            // Write 커서 이동
            if (_recvBuffer.OnWrite(args.BytesTransferred) == false)
            {
                Disconnect();
                return;
            }

            // 컨텐츠 쪽으로 데이터를 넘겨주고 얼마나 처리했는지 받는다
            int processLen = OnRecv(_recvBuffer.ReadSegment);
            if (processLen < 0 || processLen > _recvBuffer.DataSize)
            {
                Disconnect();
                return;
            }

            // Read 커서 이동
            if (_recvBuffer.OnRead(processLen) == false)
            {
                Disconnect();
                return;
            }

            RegisterRecv();
        }
        catch (Exception ex)
        {
            Console.WriteLine($"OnRecvCompleted Failed : {ex}");
        }
    }
    else
    {
        Disconnect();
    }
}
```
<br>

`RecvBuffer Class`를 구현하여 부분적으로 데이터를 처리하는 기능을 만들어 보았다.

<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
public class RecvBuffer
{
    // [][][][][][][][][][]
    ArraySegment<byte> _buffer;
    int _readPos;   // 읽을 버퍼의 시작 위치
    int _writePos;  // 받은 버퍼의 마지막 위치

    // 버퍼 사이즈 저장
    public RecvBuffer(int bufferSize)
    {
        _buffer = new ArraySegment<byte>(new byte[bufferSize], 0, bufferSize);
    }

    // 읽을 버퍼 크기 구하기
    public int DataSize { get { return _writePos - _readPos; } }

    // 받을 수 있는 버퍼 크기 구하기
    public int FreeSize { get { return _buffer.Count - _writePos; } }

    // 어디부터 읽으면 되는지? 전달 ( 버퍼 전달 )
    public ArraySegment<byte> ReadSegment
    {
        get { return new ArraySegment<byte>(_buffer.Array, _buffer.Offset + _readPos, DataSize); }
    }

    // 버퍼자리가 얼마나 남았는지? 전달 ( 버퍼 전달 )
    public ArraySegment<byte> WriteSegment
    {
        get { return new ArraySegment<byte>(_buffer.Array, _buffer.Offset + _writePos, FreeSize); }
    }

    // 커서 위치 리셋 ( read, write의 위치가 계속 올라가면 버퍼 크기를 초과하기 때문 )
    public void Clean()
    {
        int dataSize = DataSize;
        if (dataSize == 0)
        {
            // 남은 데이터가 없으면 복사하지 않고 커서 위치만 리셋
            _readPos = _writePos = 0;
        }
        else
        {
            // 남은 데이터가 있으면 시작 위치로 복사
            // Array.Copy(복사할 버퍼, 복사할 시작 위치, 붙여넣기할 버퍼, 붙여넣기할 시작 위치, 복붙할 크기);
            Array.Copy(_buffer.Array, _buffer.Offset + _readPos, _buffer.Array, _buffer.Offset, dataSize);
            _readPos = 0;
            _writePos = dataSize;
        }
    }

    // 읽을 때
    public bool OnRead(int numOfBytes)
    {
        if (numOfBytes > DataSize)
            return false;

        _readPos += numOfBytes;
        return true;
    }

    // 받을 때
    public bool OnWrite(int numOfBytes)
    {
        if (numOfBytes > FreeSize)
            return false;

        _writePos += numOfBytes;
        return true;
    }
}
```

</div>
</details>

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}