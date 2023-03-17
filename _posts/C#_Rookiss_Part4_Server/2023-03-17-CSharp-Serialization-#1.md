---
title: C# Serialization_1
author: LHH
date: 2023-03-17 14:10 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Serialization]
---

나중에 **직렬화** 관련 코드는 매우 많이 사용되기 때문에 자동화를 해주는 것이 좋다.

이번 글은 자동화를 만들기 위해 어떤식으로 구현할지 미리 인터페이스를 짜본다.

## 💻 코드
### 패킷 추가
테스트할 임시 패킷을 추가한다.
```cs
class Packet
{
    public ushort size;
    public ushort packetId;
}

// 유저 확인용으로 보내줄 패킷
class PlayerInfoReq : Packet
{
    public long playerId;
}

// 유저 확인될 시 보내줄 패킷
class PlayerInfoOk : Packet
{
    public int hp;
    public int attack;
}

// 패킷 ID
public enum PacketID
{
    PlayerInfoReq = 1,
    PlayerInfoOk = 2,
}
```
<br>

`Server`는 `ClientSession Class`를 만들어주고, `Client`는 `ServerSession`을 만들어준다.

각 Session은 서로의 도우미 역할로 어떤식으로 보내고 받는지 도와줄 것이다.

### Client
`BitConverter.TryWriteBytes`를 통해 버퍼안에 데이터를 가져올 수 있다.

`count`로 다음 위치 확인 및 사이즈를 체크해준다.

만약 버퍼안에 데이터가 없는 경우 `success`가 `false`를 뱉으므로 `Send`할 수 없다.
```cs
namespace DummyClient
{
    class ServerSession : Session
    {
        public override void OnConnected(EndPoint endPoint)
        {
            Console.WriteLine($"OnConnected : {endPoint}");

            PlayerInfoReq packet = new PlayerInfoReq(){
                size = 4, packetId = (ushort)PacketID.PlayerInfoReq, playerId = 1001
            };

            // 4096 버퍼 사용하기 (이름이 길어 segment -> s)
            ArraySegment<byte> s = SendBufferHelper.Open(4096);

            bool success = true;
            ushort count = 0;

            // [직렬화 시작]
            // size 공간 +
            count += 2;

            // packetId 공간 +
            success &= BitConverter.TryWriteBytes(new Span<byte>(s.Array, s.Offset + count, s.Count - count), packet.packetId);
            count += 2;

            // playerId 공간 +
            success &= BitConverter.TryWriteBytes(new Span<byte>(s.Array, s.Offset + count, s.Count - count), packet.playerId);
            count += 8;

            // 총 사이즈는 다 넣어야 알 수 있기 때문에 마지막에 넣기 
            success &= BitConverter.TryWriteBytes(new Span<byte>(s.Array, s.Offset, s.Count), count);

            // 얼마나 썼는지 알려주기
            ArraySegment<byte> sendBuff = SendBufferHelper.Close(count);

            // 조립에 성공하면 전송
            if (success)
                Send(sendBuff);
        }
    }
}
```

### Server
`size`와 `packetId`는 정해져 있으니 먼저 구현하고,

그 다음 버퍼에는 전달 받을 데이터가 들어있으니 `switch`를 사용해 확인한다.
```cs
namespace Server
{
    class ClientSession : PacketSession
    {
        public override void OnRecvPacket(ArraySegment<byte> buffer)
        {
            // [역직렬화 시작]
            int count = 0;
            ushort size = BitConverter.ToUInt16(buffer.Array, buffer.Offset);
            count += 2;
            ushort id = BitConverter.ToUInt16(buffer.Array, buffer.Offset + count);
            count += 2;

            switch ((PacketID)id)
            {
                case PacketID.PlayerInfoReq:
                    {
                        long playerId = BitConverter.ToInt64(buffer.Array, buffer.Offset + count);
                        count += 8;

                        Console.WriteLine($"PlayerInfoReq : {playerId}");
                    }
                    break;
            }
        }
    }
}
```

## `BitConverter.TryWriteBytes`가 선언이 안된다면
구글링을 좀 하다가 프레임워크를 추가 적용하는 방법을 몰라서 프로젝트를 새로 만들고 
1. `프로젝트 우클릭 -> 속성 -> 대상 프레임워크 확인`
2. `대상 프레임워크`가 `.Net Core`가 아닐 경우 바꿔주도록 한다.
3. 만약 `.Net Core`가 없다면 프로젝트를 다시 만들어서 프레임워크가 없는 콘솔창을 선택해 만들어준다.
4. 원래 사용하던 코드를 옮겨준다.

### 프레임워크 없는 콘솔창 선택
![프젝만들기](https://user-images.githubusercontent.com/110723307/225818255-998ca5f9-6f16-4d58-9104-6b5c79ba9042.PNG)

### 프레임워크 `.Net Core` 설정
![프레임워크](https://user-images.githubusercontent.com/110723307/225818259-79f3ead0-3d22-419a-bbd7-063c798474ce.PNG)

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}