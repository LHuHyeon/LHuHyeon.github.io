---
title: C# Session_#2
author: LHH
date: 2023-03-12 16:50 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Session, Send]
---

지난 글([Session #1](/posts/CSharp-Session-1))에 이어서 이번에는 Send를 비동기로 만들어준다.

## 💻 코드
Send는 여러 클라이언트가 `_socket.SendAsync`를 접근했을 때 느리고 부하가생긴다.

이 때문에 생성 후 사용보다 미리 만들어 놓고 재사용하는 것이 더 좋을 것이다.

- 여러 클라이언트들이 접근하므로 쓰레드 사용을 위해 `_lock`을 생성해준다.
- 버퍼 값을 저장해 한번에 보내줘야 부하가 덜하기 때문에 이를 관리해줄 Queue도 생성해준다.
- args도 재사용을 위해 멤버변수로 선언해준다.

```cs
object _lock = new object();

Queue<byte[]> _sendQueue= new Queue<byte[]>();
bool _pending = false;
SocketAsyncEventArgs _sendArgs = new SocketAsyncEventArgs();

public void Start()
{
    _sendArgs.Completed += new EventHandler<SocketAsyncEventArgs>(OnSendCompleted);
}

// Send 등록
void RegisterSend()
{

}

// Send 성공
void OnSendCompleted(object sender, SocketAsyncEventArgs args)
{

}
```
<br>

메시지를 전송하기 위해 제일 첫번째로 오는 메소드다.

만약 이곳에서 버퍼를 생성하고(`args.SetBuffer()`) 한다면

`버퍼 생성 -> 등록`을 반복하기 때문에 이것만으로 여러 클라이언트가 몰린다면 부하가 많이 생길수 있다.

그러므로 받은 버퍼를 `Queue`에 담아주고 한번 등록할 때 한꺼번에 처리하는 방식으로 진행한다.

`Send()`는 여러 클라이언트가 접근하기 때문에 lock을 걸어주도록 한다.
```cs
public void Send(byte[] sendBuff)
{
    // 계속 바이트를 만들어 주기에는 부담이 너무 크다.
    // 그러므로 큐로 보내줄 바이트들을 받아준 다음
    // 보낼준비가 되면 그때 한번에 보낸다.
    lock (_lock)
    {
        _sendQueue.Enqueue(sendBuff);
        if (_pending == false)
            RegisterSend();
    }
}
```
<br>

Send를 등록한다면 다음 Send가 못들어오도록 `_pending`을 `true`로 전환해준다.

이곳에서 `Queue`로 받은 버퍼를 `Dequeue`하여 버퍼를 뽑고 생성 후 `SendAsync`를 진행한다.
```cs
void RegisterSend()
{
    _pending = true;
    byte[] buff = _sendQueue.Dequeue();
    _sendArgs.SetBuffer(buff, 0, buff.Length);

    bool pending = _socket.SendAsync(_sendArgs);
    if (pending == false)
        OnSendCompleted(null, _sendArgs);
}
```
<br>

Send는 Recv와 다르게 보내기에 성공했을 때 추가로 처리해줄만한 것이 없다.

그러므로 만약 보내기에 성공하고 그 사이에 다른 클라이언트가 `Send()` 했다면

다시 `ReigsterSend()`를 호출하고 그게 아니라면 `_pending`을 `false`로 전환시켜준다.

이곳도 마찬가지로 `RegisterSend()`에서 접근할뿐만 아니라 `EventHandler`로

성공한다면 접근하는 곳이기 때문에 `lock`을 걸어주도록 한다.
```cs
void OnSendCompleted(object sender, SocketAsyncEventArgs args)
{
    lock (_lock)
    {
        if (args.BytesTransferred > 0 && args.SocketError == SocketError.Success)
        {
            try
            {
                // 끝내기 전 Send할게 있다면 이곳에서 처리해주기.
                if (_sendQueue.Count > 0)
                    RegisterSend();
                else
                    _pending = false;
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

여러 클라이언트의 접근을 생각하여 구현하니 부하에 관련해서 도움이 되었다.

1000명 이상의 클라이언트가 접근한다 생각하며 구현하는 습관을 가져야겠다.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}