---
title: TCP/UDP 프로토콜 요약
author: LHH
date: 2023-03-14 22:45 GMT+0900
categories: [Study, Network]
tags: [Network, 전송 계층, TCP, UDP]
---

## TCP / UDP 차이점
- **연결 지향성**
  - TCP
    - 연결을 위해 할당되는 논리적인 경로가 있다.
    - `전송 시 순서가 보장됨.`
  - UDP
    - 연결이란 개념이 없다.
    - `전송 시 순서가 보장되지 않는다.`

- **속도와 신뢰성**
  - TCP
    - `신뢰성을 보장한다.`
    - `흐름과 혼잡을 제어한다.`
    - 고려할 것이 많기 때문에 속도가 느리다.
  - UDP
    - `신뢰성이 없다.`
    - `보내고 생각한다.`
    - 패킷이 단순하기 때문에 TCP보다 더 빠르다.

<br>

## 비교

|        종류       | TCP | UDP |
|:-----------------:|:---:|:---:|
| 연결성             |  O  |  X  |
| 신뢰성             |  O  |  X  |
| 데이터 경계 구분    |  O  |  X  |
| 전송 순서 보장      |  O  |  X  |
| 흐름제어 및 혼잡제어 |  O  |  X  |
| 수신 여부 확인      |  O  |  X  |
| 속도               | 느림 | 빠름 |
| 통신 방식           | 1:1 | 1:1, 1:N, N:N |

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4){:target="_blank"}
- [MangKyu: TCP와 UDP의 특징과 차이](https://mangkyu.tistory.com/15)