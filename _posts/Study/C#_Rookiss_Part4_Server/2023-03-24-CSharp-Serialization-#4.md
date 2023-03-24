---
title: C# Serialization_#4
author: LHH
date: 2023-03-24 17:32 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Serialization]
---

지난 글([Serialization #3](/posts/CSharp-Serialization-3))에 이어서 List를 관리하도록 구현한다.

## 💻 코드
### Skill List
`SkillInfo` 구조체를 만들어 `List`를 생성해준다.
```cs
class PlayerInfoReq : Packet
{
    public long playerId;
    public string name;

    public struct SkillInfo
    {
        public int id;
        public short level;
        public float duration;
    }

    public List<SkillInfo> skills = new List<SkillInfo>();
}
```

### SkillInfo에 Write(), Read() 생성
구조체 같이 여러 변수를 한번에 가지고 있는 상황에는 그 안에 `Read()`, `Wrtie()`를 만들어 관리한다.

이때 각 변수마다 사용되는 바이트 용량이 다르기 때문에 작성할 때 실수가 없도록 한다.
```cs
public struct SkillInfo
{
    // 한번에 쓰기
    public bool Write(Span<byte> s, ref ushort count)
    {
        bool success = true;

        // id
        success &= BitConverter.TryWriteBytes(s.Slice(count, s.Length - count), id);
        count += sizeof(int);

        // level
        success &= BitConverter.TryWriteBytes(s.Slice(count, s.Length - count), level);
        count += sizeof(short);

        // duration
        success &= BitConverter.TryWriteBytes(s.Slice(count, s.Length - count), duration);
        count += sizeof(float);

        return success;
    }

    // 한번에 읽기
    public void Read(ReadOnlySpan<byte> s, ref ushort count)
    {
        // id
        id = BitConverter.ToInt32(s.Slice(count, s.Length - count));
        count += sizeof(int);

        // level
        level = BitConverter.ToInt16(s.Slice(count, s.Length - count));
        count += sizeof(short);

        // duration (float는 ToSingle 사용)
        duration = BitConverter.ToSingle(s.Slice(count, s.Length - count));
        count += sizeof(float);
    }
}
```

### 적용하기
완성된 구조체를 패킷으로 사용할 수 있도록 적용해준다.
```cs
public override ArraySegment<byte> Wirte()
{
    // skills 구조체 크기 공간 + 
    success &= BitConverter.TryWriteBytes(s.Slice(count, s.Length - count), (ushort)skills.Count);
    count += sizeof(ushort);

    // 스킬 구조 쓰기
    foreach(SkillInfo skill in skills)
        success &= skill.Write(s, ref count);
}

public override void Read(ArraySegment<byte> segment)
{
    // Skill 크기 받기
    skills.Clear();
    ushort skillLen = BitConverter.ToUInt16(s.Slice(count, s.Length - count));
    count += sizeof(ushort);

    // Skill 정보 받기
    for (int i = 0; i < skillLen; i++)
    {
        SkillInfo skill = new SkillInfo();
        skill.Read(s, ref count);
        skills.Add(skill);
    }
}
```

### 사용하기
각 세션에 만들어준 패킷을 생성하여 정보를 입력 후 송수신을 진행한다.
```cs
// Client
class ServerSession : Session
{
    public override void OnConnected(EndPoint endPoint)
    {
        Console.WriteLine($"OnConnected : {endPoint}");

        PlayerInfoReq packet = new PlayerInfoReq(){ playerId = 1001, name = "ABCDE" };
        packet.skills.Add(new PlayerInfoReq.SkillInfo() { id = 10, level = 3, duration = 777 });
        packet.skills.Add(new PlayerInfoReq.SkillInfo() { id = 11, level = 4, duration = 230 });
        packet.skills.Add(new PlayerInfoReq.SkillInfo() { id = 12, level = 5, duration = 982 });

        ArraySegment<byte> s = packet.Wirte();

        if (s != null)
            Send(s);
    }
}
```
```cs
// Server
class ClientSession : PacketSession
{
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

                    // 받은 playerId, name 출력
                    Console.WriteLine($"PlayerInfoReq : {p.playerId}, {p.name}");

                    // 받은 스킬 정보 출력
                    foreach(PlayerInfoReq.SkillInfo skill in p.skills)
                    {
                        Console.WriteLine($"ID : {skill.id}, Level : {skill.level}, Duration : {skill.duration}");
                    }
                }
                break;
        }

        Console.WriteLine($"RecvPacketId : {id}, Size : {size}");
    }
}
```

### 결과
사이즈도 모두 동일하고, 미리 입력한 `Skills` 값도 정상적으로 송수신이 완료됐다.
![](https://user-images.githubusercontent.com/110723307/227466058-3728570b-bbed-4016-92f3-7c3415865d06.PNG)

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}