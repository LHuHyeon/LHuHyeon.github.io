---
title: C# Connector
author: LHH
date: 2023-03-14 11:50 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Socket, Connector]
---

당연한 소리지만 서버 통신에 있어서 서버만 구현하면 안되고, 클라이언트 부분에서도 서버로 연결요청을 해줘야한다.

이또한 사용되는 `Connect`라는 명령어를 비동기식으로 만들어야되기 때문에 구현이 필요하다.

## 💻 코드
`ServerCore`에 `Connector` 클래스를 생성한다.

구현 방식은 `Listener`와 매우 흡사하므로 추가된 코드는 주석에 적어놨다.
```cs
public class Connector
{
    // Socket _socket; <- 이것을 멤버로 사용안한 이유
    // Listener가 계속 돌아가며 여러 클라이언트를 받는 것처럼
    // _socket을 여러 클라이언트가 접근할 것이기 때문에
    // 인자로 넘겨주며 사용하도록 한다.

    Func<Session> _sessionFactory;

    public void Connect(IPEndPoint endPoint, Func<Session> sessionFactory)
    {
        Socket socket = new Socket(endPoint.AddressFamily, SocketType.Stream, ProtocolType.Tcp);
        _sessionFactory = sessionFactory;

        SocketAsyncEventArgs args = new SocketAsyncEventArgs();
        args.Completed += OnConnectCompleted;   // 성공한다면
        args.RemoteEndPoint = endPoint;         // 보낼 주소 설정
        args.UserToken = socket;                // 저장할 토큰

        RegisterConnect(args);
    }

    void RegisterConnect(SocketAsyncEventArgs args)
    {
        // 소켓 체크
        Socket socket = args.UserToken as Socket;
        if (socket == null)
            return;

        // 전송이 바로 됐다면
        bool pending = socket.ConnectAsync(args);
        if (pending == false)
            OnConnectCompleted(null, args);
    }

    void OnConnectCompleted(object sender, SocketAsyncEventArgs args)
    {
        if (args.SocketError == SocketError.Success)
        {
            // 클라가 만들어놓은 Session 콜백 진행
            Session session = _sessionFactory.Invoke();
            session.Start(args.ConnectSocket);
            session.OnConnected(args.RemoteEndPoint);
        }
        else
        {
            Console.WriteLine($"OnConnectCompleted Failed {args.SocketError}");
        }
    }
}
```
<br>

## ServerCore 클래스 라이브러리화
`ServerCore`는 말그대로 라이브러리처럼 사용될 것이기 때문에 라이브러리 설정을 해주고, 다른 Client, Server에서 사용할 수 있도록 참조해준다.

- ServerCore 클래스 우클릭 -> 설정
- 출력 형식 클릭 -> 클래스 라이브러리 선택 -> 저장
![classLib](https://user-images.githubusercontent.com/110723307/224724065-d28319d8-e345-42b4-8455-7677d94ba4f6.PNG)

<br>

- 참조할 클래스 우클릭 -> 추가 -> 참조
- 프로젝트 클릭 -> ServerCore 선택 -> 확인
![참조](https://user-images.githubusercontent.com/110723307/224724054-072abb8b-0b7f-4a24-bb68-d8ef08addce4.PNG)

<br>

- 참조가 됐다면 사진처럼 참조부분에 `ServerCore`가 생성됐음을 확인할 수 있다.
![check](https://user-images.githubusercontent.com/110723307/224724040-99ecb79c-3447-4a54-8807-5cdb27c64ad1.PNG)

<br>

## Server / Client 코드 구현
이제 참조받은 `ServerCore`를 라이브러리 처럼 사용할 수 있다. 이를 활용해 구현하도록 한다.

### Server
임시로 사용하던 `ServerCore`의 `Main`쪽 코드를 모두 긁어와 `Server`에 넣어준다.

코드는 이미 완성된 것이니 그대로 사용할 것이다.

<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
namespace Server
{
    class GameSession : Session
    {
        public override void OnConnected(EndPoint endPoint)
        {
            Console.WriteLine($"OnConnected : {endPoint}");

            byte[] sendBuff = Encoding.UTF8.GetBytes("Welcome to LHH Server !!");
            Send(sendBuff);
            Thread.Sleep(1000);
            Disconnect();
        }

        public override void OnDisConnected(EndPoint endPoint)
        {
            Console.WriteLine($"OnDisConnected : {endPoint}");
        }

        public override void OnRecv(ArraySegment<byte> buffer)
        {
            string recvData = Encoding.UTF8.GetString(buffer.Array, buffer.Offset, buffer.Count);
            Console.WriteLine($"[From Client] {recvData}");
        }

        public override void OnSend(int numOfBytes)
        {
            Console.WriteLine($"Transferred Bytes : {numOfBytes}");
        }
    }

    class Program
    {
        static Listener _listener = new Listener();

        static void Main(string[] args)
        {
            // DNS (Domain Name System)
            string host = Dns.GetHostName();
            IPHostEntry ipHost = Dns.GetHostEntry(host);
            IPAddress ipAddress = ipHost.AddressList[0];
            IPEndPoint endPoint = new IPEndPoint(ipAddress, 5000);

            _listener.Init(endPoint, () => { return new GameSession(); });
            Console.WriteLine("Listening...");

            while (true)
            {
                ;
            }
        }
    }
}
```

</div>
</details>

<br>

### Client
클라이언트도 서버와 구현이 거의 똑같다.

<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
namespace DummyClient
{
    class GameSession : Session
    {
        public override void OnConnected(EndPoint endPoint)
        {
            Console.WriteLine($"OnConnected : {endPoint}");

            // 보낸다
            for (int i = 0; i < 5; i++)
            {
                byte[] sendBuff = Encoding.UTF8.GetBytes($"Hello World!! {i + 1} ");
                Send(sendBuff);
            }
        }

        public override void OnDisConnected(EndPoint endPoint)
        {
            Console.WriteLine($"OnDisConnected : {endPoint}");
        }

        public override void OnRecv(ArraySegment<byte> buffer)
        {
            string recvData = Encoding.UTF8.GetString(buffer.Array, buffer.Offset, buffer.Count);
            Console.WriteLine($"[From Server] {recvData}");
        }

        public override void OnSend(int numOfBytes)
        {
            Console.WriteLine($"Transferred Bytes : {numOfBytes}");
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // DNS (Domain Name System)
            string host = Dns.GetHostName();
            IPHostEntry ipHost = Dns.GetHostEntry(host);
            IPAddress ipAddress = ipHost.AddressList[0];
            IPEndPoint endPoint = new IPEndPoint(ipAddress, 5000);

            Connector connector = new Connector();
            connector.Connect(endPoint, () => { return new GameSession(); });

            while (true)
            {
                try
                {
                }
                catch (Exception ex)
                {
                    Console.WriteLine(ex.ToString());
                }

                Thread.Sleep(100);
            }
        }
    }
}
```

</div>
</details>

<br>

## 결과
![결과](https://user-images.githubusercontent.com/110723307/224743082-1463601d-304c-4a47-bd47-8d51cb934573.PNG)

이렇게 클라이언트도 서버에게 `connect`할때 블록킹되는 것을 지양해야 하므로 비동기식으로 구현하였다.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}