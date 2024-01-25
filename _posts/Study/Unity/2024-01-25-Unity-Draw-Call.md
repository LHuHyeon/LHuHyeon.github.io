---
title: (Unity) Draw Call 최적화
author: LHH
date: 2024-01-25 16:00 GMT+0900
categories: [Study, Unity]
tags: [Unity, Draw Call, Batches, SetPass Calls]
---

## 목차
1. Draw Call
2. SetPass Call
3. Batches
4. 최적화 방법

## 1. Draw Call
+ **드로우 콜(Draw Call)**은 CPU가 GPU에게 그리기 명령을 호출하는 것을 말합니다.

+ 한 프레임에서 오브젝트를 그릴 때 여러 정보들이 CPU->GPU로 이동되며 오브젝트를 다 그리면 화면에 보여주게 됩니다.

## 2. SetPass Call
+ CPU가 GPU로 명령을 할 때 따라오는 데이터를 **Command Buffer**라고 합니다.

+ **Command Buffer** 안에는 여러 데이터들이 존재하는데 그 중에서 그래픽 계열(메테리얼, 쉐이더 등..)을 담는 곳을 **SetPass**라고 하며 이것을 전달할 때 **SetPass Call**이라고 합니다.

## 3. Batches
이렇게 **Draw Call**과 **SetPass Call** 등을 다같이 포장해서 GPU에게 보내주는 것을 **Batches**라고 부릅니다.

Batches가 많아지면 부하가 오기 때문에 Batches 수를 줄이기 위해 최적화를 해줘야 합니다.

<br>

## 4. 최적화 방법
### 적정 Draw Call
+ PC : 1000 ~ 3000개

+ Mobile : 100 ~ 200개 ( 가급적이면 100 이하 맞추기 )

PC 같은 경우 성능이 좋기 떄문에 1000개가 넘어도 문제 없이 작동한다.

하지만 모바일 같은 경우 100개도 많은 편이라 부하를 막으려면 모바일 최적화는 필수이다.

### Frame Debugger
+ 프레임 디버거는 실행중인 게임에서 한 프레임에 드로우 콜을 나눠 볼 수 있는 기능입니다.

+ 이 기능을 통해 드로우 콜에 최적화가 필요한 부분을 찾아 다음과 같은 최적화를 진행해 주도록 합시다.

### Material과 Texture 묶기
+ 화면에 보여지는 객체들에서 사용되는 메테리얼과 텍스쳐를 따로따로 사용할 시 SetPass Call이 그대로 증가하게 됩니다.

+ 사용되는 용도가 비슷하다면 Atlas처럼 각 메테리얼과 텍스쳐를 묶어 사용하면 SetPass Call을 줄일 수 있습니다.

+ 예:
    + 6개의 mesh 객체가 있을 때 메테리얼, 텍스처를 묶기 전/후 상황의 변화입니다.

    + 묶기 전 = Batches : 6, SetPass Call : 6

    + 묶은 후 = Batches : 6, SetPass Call : 1

    + Material과 Texture는 그래픽 계열이기 때문에 Batches는 그대로인 반면 그래픽 계열 데이터인 SetPass Call은 6개에서 1개로 줄어들게 됩니다. (하나로 묶었기 떄문에 1이 나오는 것!)

### Dynamic Batching (동적 배칭)
+ 설정법: Project Setting -> Player -> Other Setting -> Dynamic Batching 체크

+ 동일하게 사용하고 있는 Mesh들을 Save해줌으로써 Batches를 단 하나로 줄일 수 있는 기능입니다.

+ 2D는 기본적으로 Dynamic Batching을 사용합니다.

+ 예:
    + 6개의 객체가 존재할 때 다이나믹 설정 전/후 상황입니다.

    + 설정 전 = Batches : 6, Saved by batching: 0

    + 설정 후 = Batches : 1, Saved by batching: 5

    + 유니티에서 같은 재질을 공유하는 오브젝트들을 한번에 모아 처리합니다.

    + 간단한 설정으로 Batches를 줄일 수 있기 떄문에 같은 객체를 많이 사용한다면 다이나믹 배칭을 사용해줍시다.

+ 예외:
    + 정점 속성을 900개 이상을 가지거나, 225개 이상의 정점을 포함할 때

    + 같은 재질이여도 서로 다른 인스턴스 재질 사용할 때

    + 트랜스폼에 미러링을 포함(Scale 값이 음수일 때)

### Sprite Atlas
+ 2D에서 Sprite를 하나의 Atlas로 묶어 Batches를 Save할 수 있는 기능입니다.

+ 효과: Batches 최적화

### Static Batching (정적 배칭)
+ 움직이지 않는 객체에 사용되는 설정입니다.

+ 여러 Mesh들의 정점을 하나로 통합되면서 정적으로 올라갔기 때문에 트랜스포밍 불가. (한마디로 이동 불가)

+ 설정법:
    + Project Setting -> Player -> Other Setting -> Static Batching 체크

    + 오브젝트의 Static을 체크

+ 효과: Batches 최적화

<br>

## 💡 참고
+ [[유니티 TIPS] 유니티 최적화를 위한 필수 기본기! Batching 방법 소개](https://www.youtube.com/watch?v=w14yjBlfNeQ){:target="_blank"}

+ [유니티 디버깅 - 프레임 디버거 (Frame Debugger)](https://seungsunsee.tistory.com/3){:target="_blank"}

+ [유니티에서 동적배칭 사용하기(dynamic batching)](https://learnandcreate.tistory.com/508){:target="_blank"}