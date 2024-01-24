---
title: (Unity) Asset 메모리 최적화
author: LHH
date: 2024-01-09 16:00 GMT+0900
categories: [Study, Unity]
tags: [Unity, 메모리 최적화]
---

> [2월 알쓸유잡 : 메모리 최적화를 위한 에셋 관리](https://www.youtube.com/watch?v=52ehLUfk3DQ&t=123s) 강의를 정리한 글입니다.

## 컴퓨터와 모바일의 가상 메모리 시스템
컴퓨터는 게임을 하다보면 4기가의 메모리를 가져도 하드디스크에서 메모리를 빌려주기 때문에 8기가의 게임을 진행할 수가 있습니다. (스왑페이지/페이지파일)

하지만 모바일의 경우 CPU GPU에 대한 기능이 나눠지지 않고 하드웨어 메모리 하나로 작동을 해야만 합니다. (UMA - Unified Memory Architecture)

하나의 메모리에서 운영체제, 백그라운드, 게임 등 모든 것을 사용하기 때문에 어디서 빌려올 메모리도 없기에,, 모바일의 메모리관리는 필수입니다.

## Memory Profiler
그러므로 필요한게 `Memory Profiler`입니다.

`Memory Profiler`는 유니티 Editor 상에서도 메모리를 확인할 수 있지만 모바일에 연결하여 운영체제나 모바일에 있는 각종 요소들을 확인할 수 있는 큰 장점이 있습니다.

`Memory Profiler`를 공부하고 싶다면 다음 링크를 클릭하시면 됩니다.

[[유니티 TIPS] 새로워진 Memory Profiler 1.0 활용법 소개](https://www.youtube.com/watch?v=rdspAfOFRJI&t=176s)

## 메모리에 영향을 미치는 Asset 관리
### 중복 리소스
- 유니티는 중복 파일을 체크하지 않습니다.

- Font, Audio, Texture, Mesh 등.. 다른 폴더에 실수로 동일한 파일을 둔다면 메모리에 그대로 올라가게 됩니다.

### Asset Dependency(종속 관계)
- **오브젝트의 Mesh -> Material -> Texture** 이런 식으로 유니티상에선 이어져 있는 경우가 많습니다. (*종속 관계*)

- 만약 **Asset Bundle을 사용한다면 에셋 디펜더시에 의해 중복 메모리가 발생**할 수 있습니다. 

- 그러므로 **Addressables를 사용하여 디펜더시를 수월하게 관리**할 수 있습니다.

- 유튜브 링크1 : [유나이트 서울 2020 - Addressables 사용 가이드 (초급) Track3-4](https://www.youtube.com/watch?v=EP3pvPAcHSo)

- 유튜브 링크2 : [워크플로 속도 향상을 위한 기능 소개#3 어드레서블(Addressables)](https://www.youtube.com/watch?v=-B_jcrrWMdA)

### Audio
Audio Clip 옵션에 관한 내용입니다.

- **Force To Mono 설정하기** (간단하게 사운드 메모리 줄이는 방법!)

- **Load Type**
    - **Decompress on load (< 256KB)**
        - 사운드의 압축을 풀어 메모리에 전송
        - 1MB의 mp3를 압축을 풀면 20~30MB도 될 수 있음!
        - **매우 위험한 옵션이니 되도록 사용 금지!**
        - 하지만 압축을 풀어도 작은 사이즈면 사용 고려 가능.

    - **Compressed into memory (< 1MB)**
        - 1mb 아래의 사운드가 사용하기 좋은 옵션
        - **반응 속도가 빠른 사운드라면 무조건 설정!**

    - **Streaming (> 1MB)**
        - **1mb 이상의 배경음악 같은 사운드**가 사용하기 좋은 옵션

- **Compression Format**
    - 짧은 Clip은 `ADPCM` 포맷
    - 그게 아니라면 대부분 `Vorbis` 포맷
    - iOS에서 MP3 사용이 이점이라지만 유니티에서는 H/W 디코딩을 하지 않기 때문에 일반적으로 `Vorbis` 포맷으로 가는게 좋습니다.

- **Mute 되어도 메모리에는 존재하기 때문에 사용을 안한다면 메모리에서 빼는게 좋습니다.**

### Mesh
Mesh 옵션에 관한 내용입니다.

- **Mesh Compression은 저장 용량 줄이기용**
    - 메모리와는 전혀 무관합니다.

- **Read/Write Enabled 꼭 옵션 해제**
    - CPU와 GPU의 메모리의 양쪽에 존재 해버리기 때문에 메모리량이 두배로 뻥튀기 되므로 꼭 비활성화 하는게 좋습니다.
    - 2019.3 부터는 기본 Disable

- **불 필요 시 끌 옵션들 확인**
    - Rig
    - BlendShapes
    - Normal
    - Tangent
    - Lightmap UVs
    - Generate Colliders

### Shader Variants
- **패키지 용량 이슈가 크지만 메모리도 영향 있음**

- **하나의 쉐이더가 수 많은 바이너리로 파생됨**

- **수 많은 조건들의 조합 경우의 수**
    - 노멀맵, 포그, 라이트맵, 인스터싱, 플랫폼 등등

- **Editor와 빌드 상황과의 메모리 사용량이 다르기 때문에 정확한 확인을 위해 빌드 후 확인하는게 좋습니다.**

- **Shader Variants 줄이기 위한 기법들**
    - Graphics API 지정 (Player Settings)
    - Shader Stripping (Graphics / URP Settings)
    - URP Asset에서 사용하지 않는 기능은 모두 비활성화
    - multi_compile vs shader_feature : 다만드냐 vs 설정된 것만 만드냐 (Shader 코드 작성 시 고민)
    - ShaderVariantCollection (Hiccup vs Memory)
    - Log Shader Compilation

- 유튜브 링크 : [[유니티 TIPS] 셰이더 배리언트 A to Z](https://www.youtube.com/watch?v=v3v3Q4TqMeM)

### Text Mesh Pro
- **글자들을 미리 텍스쳐로 제작해 놓는 방식**

- **영미권 언어는 큰 문제 없음!**

- **한글은 초성 중성 종성 조합 방식이기 때문에 허용 글자 범위가 필요합니다.**
    1. 한글 2350자
    2. 추가 129자 : 괞웱궴긜꺆꺍꺠껼꾺끨뉌뉍뉑댤댬댱됀됐...
    3. 한글의 위대함까지 챙긴다면.. : 꾫떃뉇뉴류뎭됇뒱랢랲뮕밢볝봹뺁뼪뽋쑖...

- **Static vs Dynamic**
    - **Static** 
        - 사용될 모든 문자 Atlas를 미리 생성
        - 런타임시 원본 폰트가 필요 없는 장점이 있음

    - **Dynamic**
        - 실시간으로 원하는 폰트 Atlas 갱신
        - 런타임시 원본 폰트 필요
        - 사용될 문자들이 예측하기 힘든 범위라면 유용 (채팅 같은 곳!)

- **Multi Atlas Textures**
    - Draw call vs 대역폭(Atlas Resolution)

### Backed Lighting
- **Lightmap**
    - static object에 적용 ( 배경, 건물 고정된 곳에 사용 )
    - 저장형태: Texture 2D
    - 메모리 이슈 발생 가능성 있음 ( 해상도를 높게 쓰거나, 넓은 맵 같은 경우 )
    - Subtractive, Backed Indirect, Shadowmask

- **Light Probe**
    - 주로 dynamic object 대응 용도 (static도 가능)
    - Diffuse term만 가능 (specular term 불가)
    - 저장형태: 개당 float x 30 + a
    - 저장형태가 적기 떄문에 메모리 이슈가 별로 없음

- **Reflection Probe**
    - Specular term을 위한 데이터
    - 저장형태: 개당 Cubemap Texture (큐브 텍스쳐는 6개의 Texture 사용)
    - 메모리 이슈 발생 가능성 큼

### Texture
대부분의 게임에선 Texture 쪽 메모리가 많이 생기기 때문에 꼭 최적화 해주기로 합시다!

- **사이즈는 가능한 작게 설정**

- **적절한 Texture Compression Format** (다음 설명 참고)

- **2D 및 UI는 SpriteAtlas 적극 활용**

- **Read/Write Enabled 옵션 해제**

- **Mipmap**
    - 2D 및 UI에서는 대부분 필요 없음
    - 3D 게임에서는 거의 필수

### Texture Compression
유니티 GPU에서 사용하는 압축 포맷을 설정합니다.

- **PNG, JPEG 등.. 원본 이미지 포맷은 무관**
    - ETC, PVRTC, ASTC ...

- **기기별 지원되는 Texture Format 사용**
    - 기기 미 지원 포맷 사용 시 압축이 풀어짐

- **모바일에서는 ASTC 추천**
    - 타겟 디바이스에 따라 다르기도 함

- **POT vs NPOT**
    - POT(Power of Two): 128, 256, 512, 1024, ... 

    - ASTC는 NPOT(Not POT) 사이즈 지원

    - Mipmap 사용 시 POT 강제 설정
        - 즉 3D, 에셋용 텍스쳐는 사실상 POT만 사용

    - UI 및 2D는 NPOT 가능
        - 하지만, Sprite Atlas 권장!
        - Sprite Atlas는 POT

#### ASTC 포맷
ASTC는 안드로이드, 아이폰 둘다 지원되는 포맷 방식입니다.

다른 포맷 방식과 달리 **ASTC는 낮은 메모리 사용량에 비해 높은 퀄리티**를 보여주기 때문에 되도록이면 ASTC를 추천합니다.

하지만 타겟팅 시장이 어디인지에 따라 다른 포맷을 사용해줘야 합니다.

구글 플레이 지원되는 기기의 비율에 따르면

|텍스처 압축 형식|지원되는 Google Play 기기의 비율|
|:---|:---|
|ETC1|99%|
|ETC2|87%|
|ASTC|77%|
|ATC|35%|
|PVRTC|11%|
|DXT1|0.7%|

ASTC는 77% 사용으로 우리나라, 미국 등에선 사용 가능하겠지만 모바일 시장이 제대로 확장되지 못한 곳에선 압축을 해제하고 그대로 메모리에 올릴 수 있기 때문에 타겟팅 시장에 맞게 사용해주는 것이 좋습니다.

#### Mipmap
Mipmap은 **가까이 있는 것은 고해상도, 멀리 있는건 저해상도로 설정**해줍니다.

예를 들어 1024 해상도의 Texture가 있다면 512, 256, 128.. 의 Texture를 더 생성합니다.

대신 3D 상황에서는 무조건 활성화 되는데 이 것이 활성화 되면 33.3333..% 메모리가 늘어나게 됩니다.

이러한 기능 때문에 2D에서는 활성화할 필요가 없습니다.

### Scene Loading
RPG 같은 게임에서 **마을 Scene -> 전장 Scene** 을 로드하려고 한다면 전장 Scene을 로드하는 과정에서 커더란 Scene인 마을과 전장이 공존하기 때문에 엄청난 메모리량이 발생하게 됩니다.

이렇게 되면 **버퍼링이 크게 발생**하거나 **튕기는 현상**이 발생하게 됩니다.

그렇기 때문에 **로딩 Scene을 하나 만들어서 기존 Scene을 완벽히 제거한 후 다음 Scene으로 넘어가는 것**이 좋습니다.

예를 들면 다음과 같은 순서로 진행하는 것 입니다.

1. 마을 Scene 로드
2. Loading Scene 로드
3. 마을 Scene 제거
4. 전장 Scene 로드
5. Loading Scene 제거

<br>

( *강의 링크는 상단에 고정되어 있습니다.* )

<br>

## 💡 도움이 되는 유니티 가이드
- [게임 개발을 최적화하는 9가지 방법(다운로드)](https://on.unity.com/3EkDewO){:target="_blank"}

- [Optimize your game performance for Consoles and PC](https://on.unity.com/3YB3hrH){:target="_blank"}

- [Ultimate guide to profiling Unity games](https://on.unity.com/3lMB1nq){:target="_blank"}

- [모바일 게임 성능 최적화 팁](https://on.unity.com/3XIs1wW){:target="_blank"}