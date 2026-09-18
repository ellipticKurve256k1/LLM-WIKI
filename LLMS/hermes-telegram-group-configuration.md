---
title: Hermes Telegram 그룹 및 봇 간 통신 설정
created_date: 2026-09-18
tags:
  - llm
  - hermes
  - telegram
  - bot
  - gateway
aliases:
  - Hermes Telegram 그룹 설정
  - Telegram 봇 그룹 설정
---

# Hermes Telegram 그룹 및 봇 간 통신 설정

Hermes Gateway의 Telegram 봇을 그룹에서 사용하고, 다른 봇의 메시지도 허용하기 위한 설정 정리.

## 1. BotFather 설정

봇을 그룹에 추가하고 그룹 메시지를 받을 수 있도록 다음을 설정한다.

- `Allow Groups` → `Enable` (`/setjoingroups`)
- `/setprivacy` → `Disable`
- `/setbot2bot` → `Enable`

Privacy 설정을 변경한 뒤 기존 그룹에서 메시지를 받지 못하면 봇을 그룹에서 제거한 다음 다시 추가한다.

## 2. Hermes `.env` 설정

봇 간 메시지를 허용하고 다른 봇의 메시지에 멘션을 요구하지 않으려면 다음과 같이 설정한다.

```env
TELEGRAM_ALLOW_BOTS=all
TELEGRAM_BOTS_REQUIRE_MENTION=false
```

## 3. 현재 그룹 접근 방식

현재 `.env`에는 `TELEGRAM_GROUP_ALLOWED_CHATS`가 설정되어 있지 않다. 대신 다음 설정이 있다.

```env
TELEGRAM_ALLOWED_USERS=7120832322
```

`TELEGRAM_ALLOWED_USERS`는 DM뿐 아니라 그룹에도 적용된다. 따라서 허용된 사용자 `7120832322`가 그룹에서 봇을 호출할 수 있고, 현재 그룹에서 봇이 동작한다.

## 4. 새 그룹을 만들 때 Gateway 재시작이 필요한가?

현재 방식에서는 새 그룹을 만들거나 봇을 새 그룹에 추가할 때마다 Gateway를 재시작할 필요가 없다. Telegram 업데이트를 Gateway가 실행 중인 상태에서 계속 수신하므로, 허용된 사용자가 새 그룹에서 호출할 수 있다.

다음 경우에는 재시작이 필요하다.

- `.env` 값을 변경한 경우
- Hermes의 `config.yaml` 설정을 변경한 경우
- Gateway 프로세스를 새 설정으로 다시 읽혀야 하는 경우

## 5. 그룹 전체 멤버를 허용하는 경우

특정 그룹의 모든 멤버가 사용할 수 있도록 그룹 자체를 허용하려면 다음 필드를 사용한다.

```env
TELEGRAM_GROUP_ALLOWED_CHATS=-100xxxxxxxxxx
```

이 필드는 현재처럼 허용된 사용자 중심으로 사용하는 경우 필수가 아니다. 여러 그룹을 명시적으로 관리해야 할 때 적합하다.

## 관련 노트

- [[LLMS/zcode-telegram-bot-channel-and-notify]] — ZCode Telegram 봇 채널과 완료 알림
- [[telegram-account-migration]] — Telegram 계정 및 기기 변경
