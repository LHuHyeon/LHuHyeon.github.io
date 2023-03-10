---
title: C# Listener
author: LHH
date: 2023-03-09 12:00 GMT+0900
categories: [C#, Server]
tags: [Rookiss 강의, C#, Server, Socket, Listener]
published: true
---
<!-- 
## 동기/비동기화
소켓을 사용할때 서버에서 Accept()를 사용하면 기본적으로 동기화가 되어 코드 블로킹이 진행되다.

이 때문에 클라이언트가 접속하기 전까지는 다음코드를 진행하지 않는다.

서버는 다른 작업도 해야하는 상황에 이렇게 무한정 기다리는 것은 비효율적이다.

그러므로 클라이언트를 기다리지 않고 다음 작업으로 바로 넘어갈 수 있도록 비동기화 작업이 필요하다.

하지만 클라이언트를 받았을 때의 관련코드가 클라이언트가 접속을 안했는데 실행이 되면 이것 또한 문제가 된다.

이렇게 비동기화 상황일 때 클라이언트도 받고 코드도 원활하게 이루어지도록 코드를 구현해보도록 한다.

## 💻 코드
[소켓 프로그래밍 기초](/posts/CSharp-소켓-프로그래밍-기초) 글에서 구현한 코드를 비동기화식으로 수정할 것이다.

Server 프로젝트에서 새로운 Listener 클래스를 생성해준다.

이 곳에서는 비동기식 Accept, Init에 필요한 코드를 넣어줄 것이다.

먼저 Socket 변수를 선언하고 `Init`, `RegisterAccept`, `OnAcceptCompleted` 메소드를 만들어준다.
```cs
class Listener
{
    Socket _listeneSocket;

    public void Init()
    {
    }

    // Accept 등록
    void RegisterAccept()
    {

    }

    // Accept 성공
    void OnAcceptCompleted()
    {

    }
}
```
<br>

Init 메소드에 Server 코드에 있던 Socket 생성과 Bind, Listen 등을 Listener 클래스로 옮겨준다.

또한 Socket을 생성해야하기 때문에 `endPoint`를 인자로 받아준다.
```cs
public void Init(IPEndPoint endPoint)
{
     // 문지기 ( SocketType.Stream, ProtocolType.Tcp 는 거의 묶음으로 사용한다. )
    _listeneSocket = new Socket(endPoint.AddressFamily, SocketType.Stream, ProtocolType.Tcp);

    // 문지기 교육
    _listeneSocket.Bind(endPoint);

    // 영업 시작
    // backlog = 최대 대기수
    _listeneSocket.Listen(10);
}
``` 
<br>

`RegisterAccept()`는 클라이언트의 등록을 관리해줄 것이다.

비동기식으로 만들어야되기 때문에 `Accept()`가 아닌 `AcceptAsync()`를 사용해준다.

`AcceptAsync()`는 `SocketAsyncEventArgs`를 인자로 받기 때문에 `RegisterAccept()` 메소드를 호출할 때 받아주도록 한다.

혹시라도 Accept를 진행했을 때 클라이언트가 바로 접속할 수도 있으므로 pending 여부를 확인해준다.
```cs
void RegisterAccept(SocketAsyncEventArgs args)
{
    // (낚시대를 던지자 마자 물고기 입질이 잡힌거)
    // 유저를 기다리는데 바로 유저가 들어왔다면 false 반환
    bool pending = _listeneSocket.AcceptAsync(args);
    if (pending == false)
        OnAcceptCompleted(null, args);
}
```
<br>

`SocketAsyncEventArgs`는 계속계속 가지고와서 사용할 수 있는 장점이 있다.

이를 관리하기 위해서 `Init`에서 선언해주고 `args.Completed`로 클라이언트의 접속 여부를

확인할 수 있기 때문에 성공시 `OnAcceptCompleted` 메소르를 호출하도록 `EventHandler`로 관리해준다.

EventHandler를 등록할 때 인자로 `object sender, SocketAsyncEventArgs args`를 받아야하는 규칙이 있으므로

`OnAcceptCompleted`에 알맞게 구현해주도록 한다.
```cs
public void Init(IPEndPoint endPoint)
{
    SocketAsyncEventArgs args = new SocketAsyncEventArgs();

    // 유저가 접속하면 OnAcceptCompleted()를 실행
    args.Completed += new EventHandler<SocketAsyncEventArgs>(OnAcceptCompleted);

    // 클라이언트 등록 기다리기
    RegisterAccept(args);
}

void OnAcceptCompleted(object sender, SocketAsyncEventArgs args)
{
}
```
<br>

이제 클라이언트가 등록되어 접속에 성공한다면 작업을 진행해야한다.

기존에 사용되는 전송, 받기 관련 코드를 실행하기 위해 Action으로 받아 실행해줄 것이다.

`Program` 클래스로 가서 `OnAcceptHandler` 메소드를 만들고 기존 코드를 여기에 옮겨준다.

그리고 `Listener` 클래스에서 Action을 등록해야되기 때문에 Init할 때 인자로 넘겨주도록 한다.
```cs
class Program
{
    static void OnAcceptHandler(Socket clientSocket)
    {
        try
        {
            // 받는다
            byte[] recvBuff = new byte[1024];
            int recvBytes = clientSocket.Receive(recvBuff);
            string recvData = Encoding.UTF8.GetString(recvBuff, 0, recvBytes);    // 버퍼, 받을 데이터 시작 위치, 몇 만큼 받을지
            Console.WriteLine($"[From Client] {recvData}");

            // 보낸다
            byte[] sendBuff = Encoding.UTF8.GetBytes("Welcome to LHH Server !!");
            clientSocket.Send(sendBuff);

            // 쫓아낸다
            clientSocket.Shutdown(SocketShutdown.Both);
            clientSocket.Close();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }
}
```

```cs
class Listener
{
    Action<Socket> _onAcceptHandler;

    public void Init(IPEndPoint endPoint, Action<Socket> onAcceptHandler)
    {
        _onAcceptHandler += onAcceptHandler;
    }
}
```
<br>

Action 등록인 완료 됐으면 이제 마지막으로 `OnAcceptCompleted()` 코드만 구현해주면 된다.

`args.SockError`를 통해 에러를 확인하고 `_onAcceptHandler`를 Invoke하여 호출시킨다.

그리고 다음 클라이언트를 받아줘야하기 때문에 `RegisterAccept()`를 불러주도록 한다.
```cs
void OnAcceptCompleted(object sender, SocketAsyncEventArgs args)
{
    if (args.SocketError == SocketError.Success)
        _onAcceptHandler.Invoke(args.AcceptSocket);
    else
        Console.WriteLine(args.SocketError.ToString());

    // 다시 낚시대 던지기
    RegisterAccept(args);
}
```
<br>

하지만 이대로 실행한다면 Socket 중복으로 버그나 에러가 날것이다.

그렇기 때문에 `RegisterAccept()`를 실행할 때 Socket을 초기화해주는 작업을 거쳐야 한다.
```cs
void RegisterAccept(SocketAsyncEventArgs args)
{
    // args 소켓이 초기화하지 않으면 pending은 계속 false가 나옴.
    args.AcceptSocket = null;
}
```
<br>

마지막으로 Main을 수정해준다.

`while(true)`를 마지막에 넣어준 이유는 서버가 종료되기 때문에

무한루프를 돌 수 있도록 해놓은 것이다.
```cs
static void Main(string[] args)
{
    // DNS (Domain Name System)
    string host = Dns.GetHostName();
    IPHostEntry ipHost = Dns.GetHostEntry(host);
    IPAddress ipAddress = ipHost.AddressList[0];
    IPEndPoint endPoint = new IPEndPoint(ipAddress, 5000);

    _listener.Init(endPoint, OnAcceptHandler);
    Console.WriteLine("Listening...");

    while (true)
    {
        ;
    }
}
```

## 결론
컴파일 결과는 달라진게 없지만 비동기식으로 코드를 구현했기 때문에

이제 클라이언트를 대기하느라 못했던 작업을 진행할 수 있게됐다.
<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
class Listener
{
    Socket _listeneSocket;
    Action<Socket> _onAcceptHandler;

    public void Init(IPEndPoint endPoint, Action<Socket> onAcceptHandler)
    {
        // 문지기 ( SocketType.Stream, ProtocolType.Tcp 는 거의 묶음으로 사용한다. )
        _listeneSocket = new Socket(endPoint.AddressFamily, SocketType.Stream, ProtocolType.Tcp);
        _onAcceptHandler += onAcceptHandler;

        // 문지기 교육
        _listeneSocket.Bind(endPoint);

        // 영업 시작
        // backlog = 최대 대기수
        _listeneSocket.Listen(10);

        SocketAsyncEventArgs args = new SocketAsyncEventArgs();

        // 유저가 접속하면 OnAcceptCompleted()를 실행
        args.Completed += new EventHandler<SocketAsyncEventArgs>(OnAcceptCompleted);
        RegisterAccept(args);
    }

    // 등록
    void RegisterAccept(SocketAsyncEventArgs args)
    {
        // args 소켓이 초기화하지 않으면 pending은 계속 false가 나옴.
        args.AcceptSocket = null;

        // (낚시대를 던지자 마자 물고기 입질이 잡힌거)
        // 유저를 기다리는데 바로 유저가 들어왔다면 false 반환
        bool pending = _listeneSocket.AcceptAsync(args);
        if (pending == false)
            OnAcceptCompleted(null, args);
    }

    // 성공
    void OnAcceptCompleted(object sender, SocketAsyncEventArgs args)
    {
        if (args.SocketError == SocketError.Success)
            _onAcceptHandler.Invoke(args.AcceptSocket);
        else
            Console.WriteLine(args.SocketError.ToString());

        // 다시 낚시대 던지기
        RegisterAccept(args);
    }
}

class Program
{
    static Listener _listener = new Listener();

    static void OnAcceptHandler(Socket clientSocket)
    {
        try
        {
            // 받는다
            byte[] recvBuff = new byte[1024];
            int recvBytes = clientSocket.Receive(recvBuff);
            string recvData = Encoding.UTF8.GetString(recvBuff, 0, recvBytes);    // 버퍼, 받을 데이터 시작 위치, 몇 만큼 받을지
            Console.WriteLine($"[From Client] {recvData}");

            // 보낸다
            byte[] sendBuff = Encoding.UTF8.GetBytes("Welcome to LHH Server !!");
            clientSocket.Send(sendBuff);

            // 쫓아낸다
            clientSocket.Shutdown(SocketShutdown.Both);
            clientSocket.Close();
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.ToString());
        }
    }

    static void Main(string[] args)
    {
        // DNS (Domain Name System)
        string host = Dns.GetHostName();
        IPHostEntry ipHost = Dns.GetHostEntry(host);
        IPAddress ipAddress = ipHost.AddressList[0];
        IPEndPoint endPoint = new IPEndPoint(ipAddress, 5000);

        _listener.Init(endPoint, OnAcceptHandler);
        Console.WriteLine("Listening...");

        while (true)
        {
            ;
        }
    }
}
```

</div>
</details>

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"} -->