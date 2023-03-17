---
title: C# Session_#4
author: LHH
date: 2023-03-13 15:25 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Session]
---

지난 글([Session #3](/posts/CSharp-Session-3))에 이어서 완성된 Session을 사용해 보도록 한다.

## 💻 코드
비동기식으로 만들어진 Send, Recv 등을 사용하기 위해 추상메소드로 만들어준다.
```cs
abstract class Session
{
    public abstract void OnConnected(EndPoint endPoint);
    public abstract void OnRecv(ArraySegment<byte> buffer);
    public abstract void OnSend(int numOfBytes);
    public abstract void OnDisConnected(EndPoint endPoint);
}
```
<br>

추상클래스 `Session`을 사용하기 위해 `Main`에서 새로 `GameSession`을 구현한다.
- 🛠 이러한 `Session`의 경우 상속받아 사용하는게 더 편리하다.

```cs
class GameSession : Session
{
    public override void OnConnected(EndPoint endPoint)
    {
    }

    public override void OnDisConnected(EndPoint endPoint)
    {
    }

    public override void OnRecv(ArraySegment<byte> buffer)
    {
    }

    public override void OnSend(int numOfBytes)
    {
    }
}
```
<br>

만들어낸 추상메소드를 각 기능에 맞는 코어에 넣어준다.

<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
// OnDisConnected
public void Disconnect()
{
    // 혹시 클라이언트 2명 이상이 동시에 종료됐을 때를 방지
    if (Interlocked.Exchange(ref _disConnected, 1) == 1)
        return;

    // ※ 추상메소드 이곳에 넣기 ※
    OnDisConnected(_socket.RemoteEndPoint);

    _socket.Shutdown(SocketShutdown.Both);
    _socket.Close();
}

// OnSend
void OnSendCompleted(object sender, SocketAsyncEventArgs args)
{
    lock (_lock)
    {
        if (args.BytesTransferred > 0 && args.SocketError == SocketError.Success)
        {
            try
            {
                _sendArgs.BufferList = null;
                _pendingList.Clear();

                // ※ 추상메소드 이곳에 넣기 ※
                OnSend(_sendArgs.BytesTransferred);

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

// OnRecv
void OnRecvCompleted(object sender, SocketAsyncEventArgs args)
{
    // Byte가 0으로 올때도 있기 때문에 0 이상으로 받기
    if (args.BytesTransferred > 0 && args.SocketError == SocketError.Success)
    {
        try
        {
            // ※ 추상메소드 이곳에 넣기 ※
            OnRecv(new ArraySegment<byte>(args.Buffer, args.Offset, args.BytesTransferred));
            
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

당장 구현할 기능이 없으니 코어에서 임시로 사용하던 기능을 `GameSession`에 넣어준다.
```cs
class GameSession : Session
{
    public override void OnConnected(EndPoint endPoint)
    {
        Console.WriteLine($"OnConnected : {endPoint}");
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
```

</div>
</details>

<br>

지금 상황에서 `OnConnected`를 넣기에는 너무 애매하기 때문에 코드를 수정하도록 한다.

`OnConnected`가 실행될 위치를 생각해보면 `Listener`에서 클라이언트의 접속이 성공했을 때 실행되는 위치에 넣으면 될것이다.

그리고 이러한 중요한 코드(`Start, Connected`)는 내부 코어에서 실행할 수 있도록 하는 것이 좋다.

하지만 이러한 `Session`을 `Listener`에서 생성하는 것은 조금 말이 안맞기 때문에

그때그때 다양한 `Session`을 받아 사용할 수 있도록 `CallBack`을 적용한다.
```cs
class Listener
{
    // Action<Socket> _onAcceptHandler;
    Func<Session> _sessionFactory;

    // public void Init(IPEndPoint endPoint, Action<Socket> onAcceptHandler)
    public void Init(IPEndPoint endPoint, Func<Session> sessionFactory)
    {
        // _onAcceptHandler += onAcceptHandler;
        _sessionFactory += sessionFactory;
    }
}
```
```cs
class Listener
{
    void OnAcceptCompleted(object sender, SocketAsyncEventArgs args)
    {
        if (args.SocketError == SocketError.Success)
        {
            // 클라이언트와 통신 시작
            Session session = _sessionFactory.Invoke();

            session.Start(args.AcceptSocket);
            session.OnConnected(args.AcceptSocket.RemoteEndPoint);
        }
        else
            Console.WriteLine(args.SocketError.ToString());

        // 다시 낚시대 던지기
        RegisterAccept(args);
    }
}
```
<br>

`Main`에서 `Listener`를 호출하던 코드도 수정해준다.

예시로 사용하기 때문에 람다식을 사용해 `GameSession`을 생성하여 보내준다.

그리고 기존에 콜백으로 사용하던 `OnAcceptHandler`는 사용하지 않으므로

기능은 `OnConnected`에 보내주고 삭제한다.
```cs
public override void OnConnected(EndPoint endPoint)
{
    Console.WriteLine($"OnConnected : {endPoint}");

    byte[] sendBuff = Encoding.UTF8.GetBytes("Welcome to LHH Server !!");
    Send(sendBuff);
    Thread.Sleep(1000);
    Disconnect();
}

static void Main(string[] args)
{
    _listener.Init(endPoint, () => { return new GameSession(); });
}
```
<br>

## 🔍 조심해야될 부분
클라이언트가 입장하고 서버와 통신이 시작되는 부분에서

`session.Start` -> `session.OnConnected`로 넘어가는 과정에 클라이언트가 연결을 끊는다면

`session.OnConnected`에서는 에러가날 수 밖에 없다.

이러한 부분에서 미리 공부하는 것도 좋지만, 직접 경험하여 몸에 쌓는 것이 더 기억에 남을 것이다.
```cs
class Listener
{
    void OnAcceptCompleted(object sender, SocketAsyncEventArgs args)
    {
        if (args.SocketError == SocketError.Success)
        {
            // 클라이언트와 통신 시작
            Session session = _sessionFactory.Invoke();

            session.Start(args.AcceptSocket);
            session.OnConnected(args.AcceptSocket.RemoteEndPoint);
        }
        else
            Console.WriteLine(args.SocketError.ToString());

        // 다시 낚시대 던지기
        RegisterAccept(args);
    }
}
```
<br>

이렇게 구현된 Session이 어느정도 틀이 완성됐다.

<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
namespace ServerCore
{
    abstract class Session
    {
        Socket _socket;
        int _disConnected = 0;  // 소켓이 연속으로 빠져나가는걸 방지

        object _lock = new object();

        Queue<byte[]> _sendQueue= new Queue<byte[]>();
        List<ArraySegment<byte>> _pendingList = new List<ArraySegment<byte>>();
        SocketAsyncEventArgs _sendArgs = new SocketAsyncEventArgs();
        SocketAsyncEventArgs _recvArgs = new SocketAsyncEventArgs();

        public abstract void OnConnected(EndPoint endPoint);
        public abstract void OnRecv(ArraySegment<byte> buffer);
        public abstract void OnSend(int numOfBytes);
        public abstract void OnDisConnected(EndPoint endPoint);

        public void Start(Socket socket)
        {
            _socket = socket;


            _recvArgs.Completed += new EventHandler<SocketAsyncEventArgs>(OnRecvCompleted);

            // args 버퍼 생성
            _recvArgs.SetBuffer(new byte[1024], 0, 1024);

            _sendArgs.Completed += new EventHandler<SocketAsyncEventArgs>(OnSendCompleted);

            // Recv 등록
            RegisterRecv();
        }

        // 보내기
        public void Send(byte[] sendBuff)
        {
            // 계속 바이트를 만들어 주기에는 부담이 너무 크다.
            // 그러므로 큐로 보내줄 바이트들을 받아준 다음
            // 보낼준비가 되면 그때 한번에 보낸다.
            lock (_lock)
            {
                _sendQueue.Enqueue(sendBuff);
                if (_pendingList.Count == 0)
                    RegisterSend();
            }
        }

        // 연결 종료
        public void Disconnect()
        {
            // 혹시 클라이언트 2명 이상이 동시에 종료됐을 때를 방지
            if (Interlocked.Exchange(ref _disConnected, 1) == 1)
                return;

            OnDisConnected(_socket.RemoteEndPoint);
            _socket.Shutdown(SocketShutdown.Both);
            _socket.Close();
        }

        #region 네트워크 통신

        void RegisterSend()
        {
            // 알려진바는 없지만 이렇게 구현해야됨.
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

        void OnSendCompleted(object sender, SocketAsyncEventArgs args)
        {
            lock (_lock)
            {
                if (args.BytesTransferred > 0 && args.SocketError == SocketError.Success)
                {
                    try
                    {
                        _sendArgs.BufferList = null;
                        _pendingList.Clear();

                        OnSend(_sendArgs.BytesTransferred);

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

        // Recv 등록
        void RegisterRecv()
        {
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
                    OnRecv(new ArraySegment<byte>(args.Buffer, args.Offset, args.BytesTransferred));
                    
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

        #endregion
    }

    class Listener
    {
        Socket _listeneSocket;
        Func<Session> _sessionFactory;

        public void Init(IPEndPoint endPoint, Func<Session> sessionFactory)
        {
            // 문지기 ( SocketType.Stream, ProtocolType.Tcp 는 거의 묶음으로 사용한다. )
            _listeneSocket = new Socket(endPoint.AddressFamily, SocketType.Stream, ProtocolType.Tcp);
            _sessionFactory += sessionFactory;

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

        void OnAcceptCompleted(object sender, SocketAsyncEventArgs args)
        {
            if (args.SocketError == SocketError.Success)
            {
                // 받기, 보내기 시작
                Session session = _sessionFactory.Invoke();

                session.Start(args.AcceptSocket);
                session.OnConnected(args.AcceptSocket.RemoteEndPoint);
            }
            else
                Console.WriteLine(args.SocketError.ToString());

            // 다시 낚시대 던지기
            RegisterAccept(args);
        }
    }

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

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}