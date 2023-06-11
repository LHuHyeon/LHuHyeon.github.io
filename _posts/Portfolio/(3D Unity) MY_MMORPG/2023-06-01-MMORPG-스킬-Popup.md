---
title: (3D MMORPG) 스킬 Popup
author: LHH
date: 2023-06-01 00:00 GMT+0900
categories: [Portfolio, (3D Unity) MY_MMORPG]
tags: [Unity, Portfolio]
---

> 구현 기간 : 05.25 , 05.26 , 06.01

레벨에 따른 스킬을 얻을 수 있도록 스킬 Popup을 구현한다.

## 🎮 구현 기능
플레이어 레벨에 따라 스킬을 얻을 수 있고, Scene에 있는 스킬Slot에 등록하여 사용할 수 있다.

### 📝 UI_SkillPopup.cs
스킬창을 On/Off 한다.

![스킬창](https://github.com/LHuHyeon/MY_MMORPG/assets/110723307/7581c0f6-4805-430d-8872-54e2ab7bc32d)

### 📝 UI_SkillItem.cs
스킬 Popup안에 있는 SkillSlot이다. 스킬 데이터를 받아 관리한다.

### 📝 UI_SkillBarItem.cs
스킬 Popup으로 붙어 Skill을 받아 사용할 수 있는 Slot이다.

이 곳에 등록되면 `Key`로 스킬을 사용할 수 있다.

스킬을 사용하면 쿨타임이 발생하고 쿨타임 동안에 스킬을 사용할 수 없다.

## 🎬 구현 영상
![SkillPopup](https://github.com/LHuHyeon/MY_MMORPG/assets/110723307/ff3c0318-13e2-446a-9cd7-57e262ebf930)

## 그 외
- 아이템 설명 팝업 구현 (`UI_SlotTipPopup.cs`)
- 확인 팝업창 구현 (`UI_ConfirmPopup.cs`)