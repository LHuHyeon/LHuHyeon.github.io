---
title: (C#) Packet Generator [3]
author: LHH
date: 2023-04-09 18:52 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Packet]
---

지난글([Packet Generator [2]](/posts/CSharp-Packet-Generator-2))에 이어서 개발을 진행한다.

## 💻 코드
### 실행에 필요한 구성 만들기
자동화 코드는 대부분 갖춰졌기 때문에 실행하기 위해서 나머지 필요한 부분을 넣어준다. <br>
( 라이브러리, PacketID 추가 )
```cs
// {0} 패킷 이름/번호 목록
// {1} 패킷 목록
public static string fileFormat =
@"using ServerCore;
using System;
using System.Collections.Generic;
using System.Net;
using System.Text;

public enum PacketID
{
    {0}
}

{1}
";
```

### 추가된 Format 내용 Main에 적용하기
- `Main`에서 `PacketFormat`에 추가한 내용을 추가한다.
- `ParsePacket`에는 `packetEnums`에 넣을 코드를 추가한다.

```cs
class Program
{
    static ushort packetId;
    static string packetEnums;

    static void Main(string[] args)
    {
        using (XmlReader r = XmlReader.Create("PDL.xml", settings))
        {
            // {0} 패킷 이름/번호 목록
            // {1} 패킷 목록
            string fileText = string.Format(PacketFormat.fileFormat, packetEnums, genPackets);
            File.WriteAllText("GenPackets.cs", fileText);
        }
    }

    public static void ParsePacket(XmlReader r)
    {
        Tuple<string, string, string> t = ParseMembers(r);
        genPackets += string.Format(PacketFormat.packetFormat, packetName, t.Item1, t.Item2, t.Item3);

        // Enum 패킷 아이디에 넣을 코드 (띄어쓰기 + 탭)
        packetEnums += string.Format(PacketFormat.packetEnumFormat, packetName, ++packetId) + Environment.NewLine + "\t";
    }
}
```

### Byte 자동화 추가
`Byte Format` 코드를 추가한다.
```cs
// {0} 변수 이름
// {1} 변수 형식
public static string readByteFormat =
@"this.{0} = ({1})segment.Array[segment.Offset + count];
count += sizeof({1});";

// {0} 변수 이름
// {1} 변수 형식
public static string writeByteFormat =
@"segment.Array[segment.Offset + count] = (byte)this.{0};
count += sizeof({1});";
```
<br>

`ParseMember`에 있는 `switch`에 적용해준다.
```cs
switch (memberType)
{
    case "byte":
    case "sbyte":
        memberCode += string.Format(PacketFormat.memberFormat, memberType, memberName);
        readCode += string.Format(PacketFormat.readByteFormat, memberName, memberType);
        writeCode += string.Format(PacketFormat.writeByteFormat, memberName, memberType);
        break;
}
```
<br>

강의에서 테스트를 한 결과 이중 리스트도 원활하게 자동화되는 것을 알 수 있었다.

대신 struct를 class로 바꿔줘야 한다.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}