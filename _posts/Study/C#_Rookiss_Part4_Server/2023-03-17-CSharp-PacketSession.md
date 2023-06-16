---
title: (C#) PacketSession
author: LHH
date: 2023-03-17 09:15 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Session]
---

`Session`의 틀이 완성됐으니 이것을 상속받아 패킷을 관리하는 `PacketSession`을 구현해본다.

## 💻 코드
### PacketSession
서버는 클라이언트로 부터 패킷을 받았을 때 패킷이 온전하게 도착했는지 확인해야 한다.

무한루프안에서 패킷을 모두 확인할 때까지 `패킷 검사 -> 컨텐츠 확인 -> 다음 패킷 이동`을 진행한다.

`PacketSession`에 `Session`을 상속받고 `OnRecv`는 여기서만 `override`할 수 있도록 `sealed`를 넣어준다.
```cs
public abstract class PacketSession : Session
{
    public static readonly int HeaderSize = 2;

    // sealed가 붙으면 OnRecv 함수는 다른데서 override를 못한다. (OnRecv 봉인)
    public sealed override int OnRecv(ArraySegment<byte> buffer)
    {
        int processLen = 0;

        while (true)
        {
            // 패킷을 받을 수 있는 조건을 통과해야지 조립 가능
            // 1. 최소한 헤더는 파싱할 수 있는지 확인 [size(2)]
            if (buffer.Count < HeaderSize)
                break;

            // 2. 패킷이 완전체로 도착했는지 확인 [size(2)][packetId(2)][ ... ]
            ushort dataSize = BitConverter.ToUInt16(buffer.Array, buffer.Offset);
            if (buffer.Count < dataSize)
                break;

            // 여기까지 왔으면 패킷 조립 가능
            OnRecvPacket(new ArraySegment<byte>(buffer.Array, buffer.Offset, dataSize));

            // 얼만큼 받았는지 더해주기
            processLen += dataSize;

            // 다음 패킷을 읽을 위치 이동
            buffer = new ArraySegment<byte>(buffer.Array, buffer.Offset + dataSize, buffer.Count - dataSize);
        }

        return processLen;
    }

    // 컨텐츠에서 사용할 메소드
    public abstract void OnRecvPacket(ArraySegment<byte> buffer);
}
```
<br>

### 패킷 구조
실제로 대부분의 게임이 패킷을 설계할 때 첫번째는 `size`, 두번째는 `packetID`로 설계하는 경우가 많다.

또한 패킷을 보낼 때 `int` 대신 `ushort`로 보내면 2바이트를 아낄 수 있으므로 `ushort`를 사용해주는 것이 좋다.

간단한 테스트를 위해 서버와 클라이언트는 다음과 같이 패킷 클래스를 구현해준다.
```cs
class Packet
{
    public ushort size;
    public ushort packetID;
}
```
<br>

### 사용하기
#### Server
패킷을 전달받기 위해 `PacketSession`을 상속 받는다.

기존에 사용하던 `OnRecv`는 지우고 `OnRecvPacket`을 사용하도록 한다.

패킷은 size와 id를 받을 것이므로 사이즈에 맞게 추출해주는 코드를 구현한다.
```cs
namespace Server
{
    class GameSession : PacketSession
    {
        public override void OnConnected(EndPoint endPoint)
        {
            Console.WriteLine($"OnConnected : {endPoint}");

            Thread.Sleep(3000);
            Disconnect();
        }

        public override void OnRecvPacket(ArraySegment<byte> buffer)
        {
            ushort size = BitConverter.ToUInt16(buffer.Array, buffer.Offset);
            ushort id = BitConverter.ToUInt16(buffer.Array, buffer.Offset + 2);
            Console.WriteLine($"Packet ID : {id}, Size : {size}");
        }
    }
}
```
<br>

#### Client
패킷 전송을 위해 패킷을 생성한다. 생성과 동시에 값도 정해준다.

기존에 사용하던 코드를 복사해 패킷을 보내주도록 값을 변경해준다.

패킷은 `size -> id` 순서로 보내야하기 때문에 패킷 버퍼를 저장할땐 패킷구조를 잘 생각하며 넣어주도록 한다.
```cs
public override void OnConnected(EndPoint endPoint)
{
    Console.WriteLine($"OnConnected : {endPoint}");

    Packet packet = new Packet() { size = 4, packetId = 77 };

    // 보낸다
    for (int i = 0; i < 5; i++)
    {
        // 4096 버퍼 사용하기
        ArraySegment<byte> openSegment = SendBufferHelper.Open(4096);

        // 데이터 바이트 형식으로 가져오기
        byte[] sizeBuffer = BitConverter.GetBytes(packet.size);
        byte[] packetIdBuffer = BitConverter.GetBytes(packet.packetId);

        // sendBuffer에 복사.
        Array.Copy(sizeBuffer, 0, openSegment.Array, openSegment.Offset, sizeBuffer.Length);
        Array.Copy(packetIdBuffer, 0, openSegment.Array, openSegment.Offset + sizeBuffer.Length, packetIdBuffer.Length);

        // 얼마나 썼는지 알려주기
        ArraySegment<byte> sendBuff = SendBufferHelper.Close(packet.size);

        Send(sendBuff);
    }
}
```
<br>

### 결과
![](https://user-images.githubusercontent.com/110723307/225779501-fa112b41-3299-473e-945a-ba584258fb50.PNG)

클라이언트에서 설정해준 `size`, `packetId`가 서버에게 전송하여 온전히 잘 받은것을 알 수 있다.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}