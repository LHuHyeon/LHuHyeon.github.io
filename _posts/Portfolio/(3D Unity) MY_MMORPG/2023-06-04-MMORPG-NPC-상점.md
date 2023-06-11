---
title: (3D MMORPG) NPC와 상점
author: LHH
date: 2023-06-04 00:00 GMT+0900
categories: [Portfolio, (3D Unity) MY_MMORPG]
tags: [Unity, Portfolio]
---

> 구현 기간 : 06.01 ~ 06.04

NPC의 기본 기능과 상점 NPC를 구현한다.

## 🎮 구현 기능
### 📝 NpcController.cs
앞으로 만들 NPC는 전부 가만히 있기 때문에 다음과 같이 기능을 구현한다.
- 플레이어가 접근 시 플레이어 바라보기 (`Idle`)
- 플레이어가 가까이 있다면 상호작용 가능

모든 NPC는 `NpcController.cs`를 상속 받아 상호작용만 구현하면 된다.

### 📝 ShopNpcController.cs
플레이어와 가깝다면 `G`키를 눌러 상호작용할 수 있다.

상호작용하면 플레이어의 움직임을 멈추고 `상점 Popup`과 `인벤토리 Popup`을 활성화한다.

### 📝 UI_ShopPopup.cs
`UI_ShopPopup`은 소비, 방어구, 무기, 악세서리 다양한 상점으로 사용되기 때문에

`UI_ShopPopup`가 활성화 되면 `Npc`의 `Id`를 확인하고 아이템을 불러와 판매한다.

![상점](https://github.com/LHuHyeon/MY_MMORPG/assets/110723307/1759de97-a717-4733-9039-dae21fa25ee8)

### 📝 UI_ShopBuyItem.cs
`상점 Popup`에서 구매 슬롯으로 사용된다.

플레이어가 구매 슬롯을 클릭하면 골드를 확인하고 인벤에 넣어준다.

### 📝 UI_ShopSaleItem.cs
`상점 Popup`에서 판매 슬롯으로 사용된다.

인벤토리로 부터 아이템이 온다면 `UI_NumberCheckPopup`을 활성화하여 개수를 확인하고 판매 아이템을 등록한다.

`상점 Popup`에서 판매 버튼을 누르면 등록된 아이템이 판매되어 골드를 흭득한다.

## 🎬 구현 영상
![Shop](https://github.com/LHuHyeon/MY_MMORPG/assets/110723307/d7d9b052-74e7-4680-a946-c7f40fc7c4b4)
