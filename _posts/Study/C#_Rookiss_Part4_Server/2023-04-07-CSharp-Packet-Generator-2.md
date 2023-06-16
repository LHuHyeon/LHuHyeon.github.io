---
title: (C#) Packet Generator [2]
author: LHH
date: 2023-04-07 17:10 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Packet]
---

지난글([Packet Generator [1]](/posts/CSharp-Packet-Generator-1))에 이어서 개발을 진행한다.

## 💻 코드
### GenPacket 생성
패킷 자동화 스크립트를 파일로 만들기 위해 Main의 using안에 구현해준다.
```cs
static string genPackets;
class Program
{
    using (XmlReader r = XmlReader.Create("PDL.xml", settings))
    {
        File.WriteAllText("GenPackets.cs", genPackets);
    }
}
```

### 문자열 -> 코드.cs 포맷 작업
`ParsePacket`안에서 검증이 끝나면 `Tuple`을 통해 `memberCode`, `readCode`, `writeCode`를 받도록 생성한다.

생성된 `Tuple`은 `string.Format`을 통해 문자열을 코드화시킬 `PacketForamt`을 정하고

그 다음으로 `PacketFormat`에 정해진 변수를 넣어준다. ( {0}, {1}, .. )
```cs

public static void ParsePacket(XmlReader r)
{
    Tuple<string, string, string> t = ParseMembers(r);

    // {0} 클래스 이름
    // {1} 멤버 변수들
    // {2} 멤버 변수 Read
    // {3} 멤버 변수 Write
    genPackets += string.Format(PacketFormat.packetFormat, packetName, t.Item1, t.Item2, t.Item3);
}
```

### ParseMembers 메소드
`ParseMemeber` 메소드의 반환값을 `Tuple`로 수정한다.

`Tuple`안에 넣을 `memberCode`, `readCode`, `writeCode`를 선언해준다.

`while`안에서 띄어쓰기 체크 후 `switch`안에서 각 포맷에 맞는 코드화를 진행한다.

코드화가 완료되면 가독성을 위해 정렬해준다.

> 코드에 주석 참고

<details>
<summary> [ Source Code (Click) ] </summary>
<div markdown="1">

```cs
// {0} 멤버 변수들
// {1} 멤버 변수 Read
// {2} 멤버 변수 Write
public static Tuple<string, string, string> ParseMembers(XmlReader r)
{
    string packetName = r["name"];

    string memberCode = "";
    string readCode = "";
    string writeCode = "";

    int depth = r.Depth + 1;
    while (r.Read())
    {
        if (r.Depth != depth)
            break;

        // null 체크
        string memberName = r["name"];
        if (string.IsNullOrEmpty(memberName))
        {
            Console.WriteLine("Member without name");
            return null;
        }

        // 띄어쓰기 용도
        if (string.IsNullOrEmpty(memberCode) == false)
            memberCode += Environment.NewLine;
        if (string.IsNullOrEmpty(readCode) == false)
            readCode += Environment.NewLine;
        if (string.IsNullOrEmpty(writeCode) == false)
            writeCode += Environment.NewLine;

        // 변수 타입마다 알맞게 포맷
        string memberType = r.Name.ToLower();
        switch (memberType)
        {
            case "bool":
            case "short":
            case "ushort":
            case "int":
            case "long":
            case "float":
            case "double":
                memberCode += string.Format(PacketFormat.memberFormat, memberType, memberName);
                readCode += string.Format(PacketFormat.readFormat, memberName, ToMemberType(memberType), memberType);
                writeCode += string.Format(PacketFormat.writeFormat, memberName, memberType);
                break;
            case "string":
                memberCode += string.Format(PacketFormat.memberFormat, memberType, memberName);
                readCode += string.Format(PacketFormat.readStringFormat, memberName);
                writeCode += string.Format(PacketFormat.writeStringFormat, memberName);
                break;
            case "list":
                Tuple<string, string, string> t = ParseList(r);
                memberCode += t.Item1;
                readCode += t.Item2;
                writeCode += t.Item3;
                break;
            default:
                break;
        }
    }

    // 코드 정렬화 (띄어쓰기 후 탭)
    memberCode = memberCode.Replace("\n", "\n\t");
    readCode = readCode.Replace("\n", "\n\t\t");
    writeCode = writeCode.Replace("\n", "\n\t\t");

    return new Tuple<string, string, string>(memberCode, readCode, writeCode);
}

// 리스트 파싱
public static Tuple<string, string, string> ParseList(XmlReader r)
{
    // null 체크
    string listName = r["name"];
    if (string.IsNullOrEmpty(listName))
    {
        Console.WriteLine("List without name");
        return null;
    }

    // List안에서 똑같은 파싱이 이루어지기 때문에 ParseMemeber 재사용
    Tuple<string, string, string> t = ParseMembers(r);
    string memberCode = string.Format(PacketFormat.memberListFormat,
        FirstCharToUpper(listName),
        FirstCharToLower(listName),
        t.Item1,
        t.Item2,
        t.Item3);

    string readCode = string.Format(PacketFormat.readListFormat,
        FirstCharToUpper(listName),
        FirstCharToLower(listName));

    string writeCode = string.Format(PacketFormat.writeListFormat,
        FirstCharToUpper(listName),
        FirstCharToLower(listName));

    return new Tuple<string, string, string>(memberCode, readCode, writeCode);
}

// To... 변수 타입마다 쓰임이 다르므로 체크해주기
public static string ToMemberType(string memberType)
{
    switch(memberType)
    {
        case "bool":
            return "ToBoolean";
        case "short":
            return "ToInt16";
        case "ushort":
            return "ToUInt16";
        case "int":
            return "ToInt32";
        case "long":
            return "ToInt64";
        case "float":
            return "ToSingle";
        case "double":
            return "ToDouble";
        default:
            return "";
    }
}

// 첫 글자 대문자 만들기
public static string FirstCharToUpper(string input)
{
    if (string.IsNullOrEmpty(input))
        return "";
    return input[0].ToString().ToUpper() + input.Substring(1);
}

// 첫 글자 소문자 만들기
public static string FirstCharToLower(string input)
{
    if (string.IsNullOrEmpty(input))
        return "";
    return input[0].ToString().ToLower() + input.Substring(1);
}
```

</div>
</details>

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}