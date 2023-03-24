---
title: C# Serialization_2
author: LHH
date: 2023-03-23 14:40 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Serialization]
---

서버는 클라이언트로부터 패킷을 받으면 거짓말을 하고 있다 가정하며 코드를 구현해야 한다.

지난 글([Serialization #1](/posts/CSharp-Serialization-1))에 이어서 자동화에 더 가깝게 수정하고, 패킷 사이즈를 검사할 것이다.

## 💻 코드
### Packet
`Packet`의 `쓰기(Write)`, `읽기(Read)`를 추가하고 `PlayerInfoReq`를 생성하여 상속 받아준다.

기존에 `Client`와 `Server`가 송수신하던 코드를 `PlayerInfoReq`에 옮겨주었다.

패킷을 받았을 때 사이즈를 변조하여 보내면 그대로 받는 문제점이 있었다.

이를 해결하기 위해 `Read`에서 `new ReadOnlySpan<byte>()`을 사용해 범위를 지정하여 받도록 수정되었다.
```cs
public abstract class Packet
{
    public ushort size;
    public ushort packetId;

    public abstract ArraySegment<byte> Write();
    public abstract void Read(ArraySegment<byte> s);
}

class PlayerInfoReq : Packet
{
    public long playerId;

    public PlayerInfoReq()
    {
        this.packetId = (ushort)PacketID.PlayerInfoReq;
    }

    public override void Read(ArraySegment<byte> s)
    {
        ushort count = 0;
        //ushort size = BitConverter.ToUInt16(s.Array, s.Offset);
        count += 2;
        //ushort id = BitConverter.ToUInt16(s.Array, s.Offset + count);
        count += 2;

     // long playerId = BitConverter.ToInt64(s.Array, s.Offset + count);
        this.playerId = BitConverter.ToInt64(new ReadOnlySpan<byte>(s.Array, s.Offset + count, s.Count - count));
        count += 8;
    }

    public override ArraySegment<byte> Write()
    {
        // 4096 버퍼 사용하기 (이름이 길어 segment -> s)
        ArraySegment<byte> s = SendBufferHelper.Open(4096);

        bool success = true;
        ushort count = 0;

        // size 공간 +
        count += 2;

        // packetId 공간 +
        success &= BitConverter.TryWriteBytes(new Span<byte>(s.Array, s.Offset + count, s.Count - count), this.packetId);
        count += 2;

        // playerId 공간 +
        success &= BitConverter.TryWriteBytes(new Span<byte>(s.Array, s.Offset + count, s.Count - count), this.playerId);
        count += 8;

        // 총 사이즈는 다 넣어야 알 수 있기 때문에 마지막에 넣기 
        success &= BitConverter.TryWriteBytes(new Span<byte>(s.Array, s.Offset, s.Count), count);

        if (success == false)
            return null;

        // 얼마나 썼는지 알려주기
        return SendBufferHelper.Close(count);
    }
}

public enum PacketID
{
    PlayerInfoReq = 1,
    PlayerInfoOk = 2,
}
```
<br>

### Client
<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
namespace DummyClient
{
    public abstract class Packet
    {
        public ushort size;
        public ushort packetId;

        public abstract ArraySegment<byte> Write();
        public abstract void Read(ArraySegment<byte> s);
    }

    class PlayerInfoReq : Packet
    {
        public long playerId;

        public PlayerInfoReq()
        {
            this.packetId = (ushort)PacketID.PlayerInfoReq;
        }

        public override void Read(ArraySegment<byte> s)
        {
            ushort count = 0;
            //ushort size = BitConverter.ToUInt16(s.Array, s.Offset);
            count += 2;
            //ushort id = BitConverter.ToUInt16(s.Array, s.Offset + count);
            count += 2;

            this.playerId = BitConverter.ToInt64(new ReadOnlySpan<byte>(s.Array, s.Offset + count, s.Count - count));
            count += 8;
        }

        public override ArraySegment<byte> Write()
        {
            // 4096 버퍼 사용하기 (이름이 길어 segment -> s)
            ArraySegment<byte> s = SendBufferHelper.Open(4096);

            bool success = true;
            ushort count = 0;

            // size 공간 +
            count += 2;

            // packetId 공간 +
            success &= BitConverter.TryWriteBytes(new Span<byte>(s.Array, s.Offset + count, s.Count - count), this.packetId);
            count += 2;

            // playerId 공간 +
            success &= BitConverter.TryWriteBytes(new Span<byte>(s.Array, s.Offset + count, s.Count - count), this.playerId);
            count += 8;

            // 총 사이즈는 다 넣어야 알 수 있기 때문에 마지막에 넣기 
            success &= BitConverter.TryWriteBytes(new Span<byte>(s.Array, s.Offset, s.Count), count);

            if (success == false)
                return null;

            // 얼마나 썼는지 알려주기
            return SendBufferHelper.Close(count);
        }
    }

    public enum PacketID
    {
        PlayerInfoReq = 1,
        PlayerInfoOk = 2,
    }

    class ServerSession : Session
    {
        public override void OnConnected(EndPoint endPoint)
        {
            Console.WriteLine($"OnConnected : {endPoint}");

            PlayerInfoReq packet = new PlayerInfoReq(){ playerId = 1001 };

            // 보낸다
            //for (int i = 0; i < 5; i++)
            {
                ArraySegment<byte> s = packet.Write();

                if (s != null)
                    Send(s);
            }
        }

        public override void OnDisConnected(EndPoint endPoint)
        {
            Console.WriteLine($"OnDisConnected : {endPoint}");
        }

        public override int OnRecv(ArraySegment<byte> buffer)
        {
            string recvData = Encoding.UTF8.GetString(buffer.Array, buffer.Offset, buffer.Count);
            Console.WriteLine($"[From Server] {recvData}");

            return buffer.Count;
        }

        public override void OnSend(int numOfBytes)
        {
            Console.WriteLine($"Transferred Bytes : {numOfBytes}");
        }
    }
}
```

</div>
</details>

<br>

### Server
<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
namespace Server
{
    public abstract class Packet
    {
        public ushort size;
        public ushort packetId;

        public abstract ArraySegment<byte> Write();
        public abstract void Read(ArraySegment<byte> s);
    }

    class PlayerInfoReq : Packet
    {
        public long playerId;

        public PlayerInfoReq()
        {
            this.packetId = (ushort)PacketID.PlayerInfoReq;
        }

        public override void Read(ArraySegment<byte> s)
        {
            ushort count = 0;
            //ushort size = BitConverter.ToUInt16(s.Array, s.Offset);
            count += 2;
            //ushort id = BitConverter.ToUInt16(s.Array, s.Offset + count);
            count += 2;

            this.playerId = BitConverter.ToInt64(new ReadOnlySpan<byte>(s.Array, s.Offset + count, s.Count - count));
            count += 8;
        }

        public override ArraySegment<byte> Write()
        {
            // 4096 버퍼 사용하기 (이름이 길어 segment -> s)
            ArraySegment<byte> s = SendBufferHelper.Open(4096);

            bool success = true;
            ushort count = 0;

            // size 공간 +
            count += 2;

            // packetId 공간 +
            success &= BitConverter.TryWriteBytes(new Span<byte>(s.Array, s.Offset + count, s.Count - count), this.packetId);
            count += 2;

            // playerId 공간 +
            success &= BitConverter.TryWriteBytes(new Span<byte>(s.Array, s.Offset + count, s.Count - count), this.playerId);
            count += 8;

            // 총 사이즈는 다 넣어야 알 수 있기 때문에 마지막에 넣기 
            success &= BitConverter.TryWriteBytes(new Span<byte>(s.Array, s.Offset, s.Count), count);

            if (success == false)
                return null;

            // 얼마나 썼는지 알려주기
            return SendBufferHelper.Close(count);
        }
    }

    public enum PacketID
    {
        PlayerInfoReq = 1,
        PlayerInfoOk = 2,
    }

    class ClientSession : PacketSession
    {
        public override void OnConnected(EndPoint endPoint)
        {
            Console.WriteLine($"OnConnected : {endPoint}");

            /*Packet packet = new Packet() { size = 100, packetId = 10 };

            // 4096 버퍼 사용하기
            ArraySegment<byte> openSegment = SendBufferHelper.Open(4096);

            // 데이터 바이트 형식으로 가져오기
            byte[] hpBuffer = BitConverter.GetBytes(packet.size);
            byte[] atkBuffer = BitConverter.GetBytes(packet.packetId);

            // sendBuffer에 복사.
            Array.Copy(hpBuffer, 0, openSegment.Array, openSegment.Offset, hpBuffer.Length);
            Array.Copy(atkBuffer, 0, openSegment.Array, openSegment.Offset + hpBuffer.Length, atkBuffer.Length);

            // 얼마나 썼는지 알려주기
            ArraySegment<byte> sendBuff = SendBufferHelper.Close(hpBuffer.Length + atkBuffer.Length);

            Send(sendBuff);*/
            Thread.Sleep(3000);
            Disconnect();
        }

        public override void OnRecvPacket(ArraySegment<byte> buffer)
        {
            int count = 0;
            ushort size = BitConverter.ToUInt16(buffer.Array, buffer.Offset);
            count += 2;
            ushort id = BitConverter.ToUInt16(buffer.Array, buffer.Offset + count);
            count += 2;

            switch ((PacketID)id)
            {
                case PacketID.PlayerInfoReq:
                    {
                        PlayerInfoReq p = new PlayerInfoReq();
                        p.Read(buffer);

                        Console.WriteLine($"PlayerInfoReq : {p.playerId}, size : {size}, count : {count}");
                    }
                    break;
            }
        }

        public override void OnDisConnected(EndPoint endPoint)
        {
            Console.WriteLine($"OnDisConnected : {endPoint}");
        }

        public override void OnSend(int numOfBytes)
        {
            Console.WriteLine($"Transferred Bytes : {numOfBytes}");
        }
    }
}
```

</div>
</details>

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}