---
title: C# Serialization_3
author: LHH
date: 2023-03-24 11:21 GMT+0900
categories: [Study, C# Rookiss Part4 게임서버]
tags: [Rookiss 강의, C#, Server, Serialization]
---

지난 글([Serialization #2](/posts/CSharp-Serialization-2))에 이어서 코드 효율성과, 자동화를 계속 진행한다.

## 💻 코드
### 하드코딩 가독성
변수 사이즈만큼 `count`에 더해주는 작업을 했었다. 이렇게 숫자로 넣기 보다는 가독성을 위해 `sizeof()`를 사용하는 것이 더 좋다.
```cs
// 하드코딩 보다는
count += 2;
count += 8;

// sizeof로 무엇을 넣는 목적인지 보여주는게 좋음.
count += sizeof(ushort);
count += sizeof(long);
```

### Write 수정
앞전에는 `Span`을 생성하여 바로 넣는 작업을 진행했었다.

하지만 이렇게 사용하는 것 보다 `Span`을 미리 생성하여 `Slice`로 범위를 지정해주는 것이 더 가독성 좋으므로 변경해준다.

`Read()`에 `ReadOnlySpan`도 똑같이 작업해준다.
```cs
ArraySegment<byte> segment = SendBufferHelper.Open(4096);

// [ 변경 전 ]
BitConverter.TryWriteBytes(new Span<byte>(segment.Array, segment.Offset + count, segment.Count - count), this.packetId);

// [ 변경 후 ]
// Span : 배열의 일부분을 가리키는 것
Span<byte> s = new Span<byte>(segment.Array, segment.Offset, segment.Count);
BitConverter.TryWriteBytes(s.Slice(count, s.Length - count), this.packetId);
```

### string 관리
`int`, `long` 같은 변수는 크기가 고정되어 있지만, `string`같은 경우 가변적인 크기를 가지고 있으므로 관련 구현이 필요하다.

그리고 `UTF-8`, `UTF-16` 중에서 고민을 해봐야 되는데 기본적으로 `C#`에서 `UTF-16`을 선호하고 있기 때문에

굳이 `UTF-8`로 맞춰서 수정하기 보다는 구현에 있어서 `UTF-16`으로 맞춰주기로 한다.

우선 `string`의 유무를 확인하기 위해 2바이트의 문자열크기 패킷을 먼저 받고 그다음에 문자열을 받도록 진행한다.
```cs
// string 문자열 크기 공간(ushort) +
ushort nameLen = (ushort)Encoding.Unicode.GetByteCount(this.name);
success &= BitConverter.TryWriteBytes(s.Slice(count, s.Length - count), nameLen);
count += sizeof(ushort);

// name 문자열 공간(nameLen) +
Array.Copy(Encoding.Unicode.GetBytes(this.name), 0, segment.Array, count, nameLen);
count += nameLen;

// [string Length(2)][string(...)]
```

`UTF-16` 방식의 바이트 변환법은 `Encoding.Unicode`를 사용한다. 

현재 구현된 코드도 괜찮지만 크기를 뱉어내며 `name`을 `segment`에 집어넣는 방법을 동시에 진행하는 더 효율적인 코드도 존재한다.

`string` 값을 먼저 넣다보니 크기를 고려하여 구현해야되고 마지막에 `count`를 해주기 때문에 헷갈릴 수 있다.
```cs
ushort nameLen = (ushort)Encoding.Unicode.GetBytes(this.name, 0, this.name.Length, segment.Array, segment.Offset + count + sizeof(ushort));
success &= BitConverter.TryWriteBytes(s.Slice(count, s.Length - count), nameLen);
count += sizeof(ushort);
count += nameLen;
```
<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}