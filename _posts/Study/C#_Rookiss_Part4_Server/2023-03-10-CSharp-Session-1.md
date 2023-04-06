---
title: C# Session [1]
author: LHH
date: 2023-03-10 00:20 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Session, Recv]
---

지난 글([Listener](/posts/CSharp-Listener))에서 Listener를 비동기식으로 실행되도록 구현하였다.

이번에는 서버에서 클라이언트로 데이터를 보낼 때 비동기식으로 구현한다.

## 💻 코드
코드 구성은 `Listener Class`에서 구현한 것과 비슷하다.

이번에는 [소켓 프로그래밍 기초](/posts/CSharp-소켓-프로그래밍-기초)에서 구현했던 Recv를 비동기식으로 수정할 것이다.

먼저 `Session Class`를 생성하고 다음과 같이 만들어준다. (Rookiss 강사께서 Session으로 쓰는걸 좋아함!)
```cs
class Session
{
    Socket _socket;

    public void Start()
    {
    }

    // Recv 등록
    void RegisterRecv()
    {

    }

    // Recv 성공
    void OnRecvCompleted()
    {

    }
}
```
<br>

Socket을 통해 데이터를 주고 받기 때문에 `Start`에 인자로 Socket을 받아주도록 한다.

그리고 연결이 성공됐을 때 `EventHandelr`로 메소드를 호출할 수 있도록 등록해준다.

`EventHandelr`에 맞게 호출될 메소드(`OnRecvCompleted()`)의 인자를 수정해준다.

클라이언트로부터 받을 데이터 버퍼를 생성해준다.
```cs
public void Start(Socket socket)
{
    _socket = socket;

    SocketAsyncEventArgs args = new SocketAsyncEventArgs();
    args.Completed += new EventHandler<SocketAsyncEventArgs>(OnRecvCompleted);

    // args 버퍼 생성
    args.SetBuffer(new byte[1024], 0, 1024);

    // Recv 등록
    RegisterRecv(args);
}

void OnRecvCompleted(object sender, SocketAsyncEventArgs args)
{
}
```
<br>

`RegisterRecv()`는 `Listener Class`와 구현이 같기 때문에 넘어간다.

<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
void RegisterRecv(SocketAsyncEventArgs args)
{
    bool pending = _socket.ReceiveAsync(args);
    if (pending == false)
        OnRecvCompleted(null, args);
}
```

</div>
</details>

<br>

다음으로 클라이언트 연결이 성공됐을 때의 메소드를 구현한다.

가끔씩 클라이언트가 연결을 끊어버려 버퍼값이 `0`으로 올 때도 있다.

그러므로 버퍼를 받았을 때 `0` 이상의 값을 받을 수 있도록 조건문을 짜준다.

UTF8로 인코딩할때는 인자값을 다음과 같이 넣어준다.
- 전체 버퍼 : args.Buffer
- 시작 버퍼 : args.Offset
- 읽을 버퍼 : args.BytesTransferred
```cs
// Recv 성공
void OnRecvCompleted(object sender, SocketAsyncEventArgs args)
{
    // Byte가 0으로 올때도 있기 때문에 0 이상으로 받기
    if (args.BytesTransferred > 0 && args.SocketError == SocketError.Success)
    {
        try
        {
            string recvData = Encoding.UTF8.GetString(args.Buffer, args.Offset, args.BytesTransferred);
            Console.WriteLine($"[From Client] {recvData}");
            RegisterRecv(args);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"OnRecvCompleted Failed : {ex}");
        }
    }
    else
    {
        // TODO DisConnect
        Disconnect();
    }
}
```
<br>

여기까지 구현 했다면 비동기식 Recv는 완성이 된것이다.

Session에는 Recv, Send를 관리해줄 것이니 Send도 간단하게 구현해주도록 한다.
```cs
int _disConnected = 0;  // 소켓이 연속으로 빠져나가는걸 방지

// 보내기
public void Send(byte[] sendBuff)
{
    _socket.Send(sendBuff);
}

// 연결 종료
public void Disconnect()
{
    // 혹시 클라이언트 2명 이상이 동시에 종료됐을 때를 방지
    if (Interlocked.Exchange(ref _disConnected, 1) == 1)
        return;

    _socket.Shutdown(SocketShutdown.Both);
    _socket.Close();
}
```
<br>

클라이언트 연결이 성공됐을때 호출되는 `OnAcceptHandler()`를 수정해준다.
```cs
 static void OnAcceptHandler(Socket clientSocket)
{
    try
    {
        Session session = new Session();

        session.Start(clientSocket);

        byte[] sendBuff = Encoding.UTF8.GetBytes("Welcome to LHH Server !!");
        session.Send(sendBuff);

        Thread.Sleep(1000);

        // 연속 두번 빠져나갈 때의 오류 체크 (Session Disconnect에서 해결)
        session.Disconnect();
        session.Disconnect();
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
    }
}
```
<br>

`Listener`와 구현방식이 매우 비슷하기 때문에 이해하기에 있어서 어렵지 않았다.

아직 어색한 코드들이 많지만 계속 이해해 보도록 노력해야겠다.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}