---
title: C# Session_#3
author: LHH
date: 2023-03-13 14:00 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Socket, Session]
---

지난 글([Session #2](/posts/CSharp-Session-2))에 이어서 이번에는 보낼 버퍼들을 등록할 때

한꺼번에 `list`에 넣어 `SendAsync`하도록 수정하여 `Send`를 더욱 효율적이게 만든다.

## 💻 코드
이제 `bool _pending` 대신 `list`로 `byte`를 담아 `pending` 여부를 확인할 것이다.
```cs
List<ArraySegment<byte>> _pendingList = new List<ArraySegment<byte>>();

public void Send(byte[] sendBuff)
{
    lock (_lock)
    {
        _sendQueue.Enqueue(sendBuff);
        if (_pendingList.Count == 0)
            RegisterSend();
    }
}
```
<br>

`_sendQueue.Count`가 0보다 클 동안에 `Dequeue` 하여 `_pendingList`에 저장한다.

참고로 `_sendArgs.BufferList`에 넣어야하는데 `new ArraySegment<byte>` 타입으로 받기 때문에

생성하고 값을 넣어 `List`에 저장시켜주도록 한다.

그러게 모인 패킷들을 `SendAsync`하여 전송을 시작하고 전송이 바로 완료되면 `pending`은 `false`를 반환한다.
```cs
void RegisterSend()
{
    // 패킷을 한번에 모아서 보내주기
    while (_sendQueue.Count > 0)
    {
        byte[] buff = _sendQueue.Dequeue();
        _pendingList.Add(new ArraySegment<byte>(buff, 0, buff.Length));
    }

    _sendArgs.BufferList = _pendingList;

    // 모아놓은 패킷을 보낸다.
    // 만약 바로 보내진다면 pending은 false 나옴.
    bool pending = _socket.SendAsync(_sendArgs);
    if (pending == false)
        OnSendCompleted(null, _sendArgs);
}
```
<br>

혹시라도 다음 코드처럼 구현해도 되는데 왜 `List`를 생성해서 넣어주는지 이해가 안될 수 있다.

이렇게 하는 이유는 코드상에 문제는 없지만 아래 코드처럼 했을때 이상하게 작동할 수 있기 때문이다.

이유는 모르겠지만 `_sendArgs.BufferList`는 `.Add`하면 안되고 `=`으로 넘겨 받아야 한다.
```cs
void RegisterSend()
{
    // 이렇게 구현하면 적동이 이상할 수 있음.
    while (_sendQueue.Count > 0)
    {
        byte[] buff = _sendQueue.Dequeue();
        _sendArgs.BufferList.Add(new ArraySegment<byte>(buff, 0, buff.Length));
    }
}
```
<br>

`Send`가 성공했을 때도 수정해주도록 한다.

`BufferList`와 `Queue`를 초기화 시켜주고, 궁금하니 성공했을때의 보낸 메시지 byte 값도 출력해본다.

그리고 메시지를 보내는 과정에서 다른 클라이언트가 `Send()` 했다면 `_sendQueue`가 증가할테니 확인하여 등록시켜준다.
```cs
void OnSendCompleted(object sender, SocketAsyncEventArgs args)
{
    lock (_lock)
    {
        if (args.BytesTransferred > 0 && args.SocketError == SocketError.Success)
        {
            try
            {
                _sendArgs.BufferList = null;
                _sendQueue.Clear();

                Console.WriteLine($"Transferred Bytes : {_sendArgs.BytesTransferred}");

                // 끝내기 전 Send할게 있다면 이곳에서 처리해주기.
                // 혹시 보내는 과정에서 다른 클라이언트가 Send했다면
                if (_sendQueue.Count > 0)
                    RegisterSend();
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
}
```
<br>

하나하나씩 버퍼를 추가하여 등록하기보다 보내야할 메시지들을 모아 보내는 것이 효과적인 것을 알았다.

만약 몇천명의 유저가 움직이고, 공격, 스킬을 사용했을때 서로에게 보내지는 Send를 하나하나씩 보낸다면 정말 비효율적이고 느릴 것이다.

이렇게 보내질 Send의 버퍼들을 커다란 버퍼에 담아 한꺼번에 처리한다면 좀더 성능향상에 도움이 될것이다.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}