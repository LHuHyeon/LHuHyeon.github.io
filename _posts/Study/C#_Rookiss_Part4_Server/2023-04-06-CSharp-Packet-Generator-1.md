---
title: (C#) Packet Generator [1]
author: LHH
date: 2023-04-06 20:40 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Packet]
---

패킷을 자동화하는 코드를 만들어본다.

## 💻 코드
### 코드 삭제 
패킷 클래스르 따로 만들어서 자동화시킬 것이기 때문에 다음 코드는 삭제하고 에러가 나는 부분은 적절히 수정해준다.
```cs
public abstract class Packet
{
    public ushort size;
    public ushort packetId;

    public abstract ArraySegment<byte> Wirte();
    public abstract void Read(ArraySegment<byte> s);
}

public PlayerInfoReq()
{
    this.packetId = (ushort)PacketID.PlayerInfoReq;
}
```

### PDL.xml 생성
공통적으로 사용할 패킷 변수명과 이름을 xml로 생성한다.
```xml
<PDL>
	<packet name="PlayerInfoReq">
		<long name="playerId"/>
		<string name="name"/>
		<list name="skill">
			<int name="id"/>
			<short name="level"/>
			<float name="duration"/>
		</list>
	</packet>
</PDL>
```

### 패킷 자동화 코드
`PacketGenerator` 클래스를 생성하고 xml을 잘 불러 오는지 테스트한다.
```cs
namespace PacketGenerator
{
    class Program
    {
        static void Main(string[] args)
        {
            XmlReaderSettings settings = new XmlReaderSettings()
            {
                IgnoreComments = true,      // 주석 무시
                IgnoreWhitespace = true     // 스페이스바 무시
            };

            // 에러가 날 경우 PDL.xml 파일을 현재 경로에서 \bin\Debug\net6.0으로 이동하여 복붙하기
            XmlReader r = XmlReader.Create("PDL.xml", settings);
            {
                // xml에 기본적으로 들어가는 값을 무시 
                r.MoveToContent();

                // 한줄씩 읽기
                while(r.Read())
                {
                    if (r.Depth == 1 && r.NodeType == XmlNodeType.Element)
                    {
                        ParsePacket(r);
                    }
                    //Console.WriteLine(r.Name + " " + r["name"]);
                }
            }
        }
    }
}
```

만약 경로관련 에러가 나온다면 `PDL.xml`을 현재 경로에서 `\bin\Debug\net6.0`으로 이동시켜 다시 실행해본다. 

#### Parse 코드
xml의 패킷을 체크하고 읽는 메소드이다. 
```cs
// xml 검사
public static void ParsePacket(XmlReader r)
{
    // 읽은 줄이 </> 인지 체크
    if (r.NodeType == XmlNodeType.EndElement)
        return;

    // 패킷인지 체크
    if (r.Name.ToLower() != "packet")
    {
        Console.WriteLine("Invalid packet node");
        return;
    }

    // 이름을 받고 null 체크
    string packetName = r["name"];
    if (string.IsNullOrEmpty(packetName))
    {
        Console.WriteLine("Packet without name");
        return;
    }

    // 검사 다했으면 읽기
    ParseMembers(r);
}
```
```cs
// 검사 다했으면 읽기
public static void ParseMembers(XmlReader r)
{
    string packetName = r["name"];

    int depth = r.Depth + 1;
    while (r.Read())
    {
        if (r.Depth != depth)
            break;

        string memberName = r["name"];
        if (string.IsNullOrEmpty(memberName))
        {
            Console.WriteLine("Member without name");
            return;
        }

        string memberType = r.Name.ToLower();
        switch (memberType)
        {
            case "bool":
            case "byte":
            case "short":
            case "ushort":
            case "int":
            case "long":
            case "double":
            case "string":
            case "list":
                break;
            default:
                break;
        }
    }
}
```

### PacketFormat
고정적으로 사용하는 코드를 `string`으로 만들어 `@""`에 넣어준다.
```cs
class PacketFormat
{
    // @""안에 공통적인 코드를 각각 넣어 만들어준다.
    public static string packetFormat = @"";
}
```

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}