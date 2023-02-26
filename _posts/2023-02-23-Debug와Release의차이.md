---
title: Debug와 Release의 차이
author: LHH
date: 2023-02-23 11:20 GMT+0900
categories: [C#, Server]
tags: [Rookiss 강의, C#, Server]
---

### Debug란?
Debug는 시스템의 논리적인 오류나 비정상적인 연산을 찾아 원인을 밝히고 수정할 수 있다. <br>
우리가 흔히 컴파일을 진행할 때 방식을 변경할 수 있는데 기본적으로 설정되어 있는게 Debug다.

### Release란?
Release는 스스로 코드를 최적화 후 컴파일을 진행한다. <br>
개발단계가 아닌 배포할 시기에 적용하는 것이 좋다.

## Debug와 Release 비교

|               | Debug | Release |
|:--------------|:-----:|:-------:|
| 코드를 최적화  |  X    | O       |
| 코드 실행 속도 | 느림  | 빠름    |
| 컴파일 속도    | 빠름  | 느림    |
| 파일 크기      | 많음  | 적음    |
| 메모리 사용량  | 많음  | 적음    |

## 💻 코드
```cs
// * 이번 코드 핵심
// 컴파일 모드를 Debug -> Release 모드로 변경 시 어떤 상황이 벌어지는가?
// Release 모드로 변경 시 최적화가 되기 때문에 우리가 의도하지 않았던 잘못된 상황이 발생 할 수 있다.
static bool _stop = false;

static void ThreadMain()
{
    Console.WriteLine("쓰레드 시작!");

    while (_stop == false)
    {
        // 누군가 Stop 신호를 해주길 기다린다
    }

    /*
    Release 모드의 경우 위의 while 코드를 아래와 같은 코드로 최적화하게 된다.
    if (_stop == false)
    {
        while (true)
        {

        }
    }
    */

    Console.WriteLine("쓰레드 종료!");
}

static void Main(string[] args)
{
    Task t = new Task(ThreadMain);
    t.Start();

    Thread.Sleep(1000);

    _stop = true;

    Console.WriteLine("Stop 호출");
    Console.WriteLine("종료 대기중!");

    t.Wait();

    Console.WriteLine("종료 성공!");
}
```

## 결과
Debug : 코드 정상 작동 <br>
Release : 코드 최적화로 인한 무한루프 <br>

이로써 최적화의 장점은 있지만 우리가 의도하지 않는 방향으로 코드가 변형될 수 있어 Release를 사용하는 경우 코드를 더 분석하고 수정할 필요가 있다.

<br>

## 💡 참고
- [Rookiss 강의: Part4 서버](https://www.inflearn.com/course/%EC%9C%A0%EB%8B%88%ED%8B%B0-mmorpg-%EA%B0%9C%EB%B0%9C-part4)
- [lewns2 Blog](https://salon.tistory.com/19)