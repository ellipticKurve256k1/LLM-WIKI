---
title: macOS caffeinate & screen sleep commands
date: 2026-09-13
tags:
  - macos
  - shell
aliases:
  - caffeinate
  - pmset displaysleepnow
  - screen-off commands
---

# macOS caffeinate & 화면 절전 명령어

Mac을 자리 비움 상태로 만들 때 쓰는 명령어 모음. `caffeinate`로 절전 방지를 걸고 `pmset displaysleepnow`로 **화면만** 끄면 프로세스(Telegram 봇 등)는 계속 살아 있다. 실제 실행은 `screen-off.sh` 스크립트(아래 관련 섹션)가 자동화한다.

## caffeinate — 절전 방지

시스템이 잠들지 않도록 assertion을 거는 명령. `-t`로 걸린 시간이 끝나면 자동 종료된다.

| 옵션 | 의미 |
| --- | --- |
| `-i` | 시스템 idle sleep 방지 (화면 꺼짐은 허용) |
| `-d` | 디스플레이 절전만 방지 |
| `-m` | 디스크 절전 방지 |
| `-s` | 시스템(AC 전원 시) 절전 방지 |
| `-u` | 사용자 활동 assertion (10초 유효, `-t`와 조합해 연장) |
| `-t <초>` | assertion 유지 시간. 없으면 Ctrl+C까지 유지 |

### 백그라운드 실행 (세션 종료 후에도 생존)

```sh
nohup caffeinate -i -t 86400 >/dev/null 2>&1 &   # 24시간 idle-sleep 방지
```

- `nohup ... &`로 띄우면 터미널/앱을 닫아도 프로세스가 살아 있다.
- `-t 86400` = 24시간.

## 잔여 시간 확인

```sh
pgrep -x caffeinate                # PID 확인 (없으면 실행 중 아님)
ps -o etime= -p <PID>              # 경과 시간 (예: 2-03:15:00 = 2일 3시간 15분)
```

`etime` 형식: `[[dd-]hh:]mm:ss`. 남은 시간 = `86400 - 경과초`.

### 경과 시간을 초로 바꾸기 (스크립트에서 쓰는 방식)

```sh
case "$elapsed" in
  *-*)   : # dd-hh:mm:ss 형태
  *:*:*) : # hh:mm:ss
  *:*)   : # mm:ss
esac
```

`10#` 접두사로 08, 09 같은 값을 8진수 오해 없이 10진수 강제 계산한다.

## 갱신·해제

```sh
kill <PID>                                          # 기존 caffeinate 종료
nohup caffeinate -i -t 86400 >/dev/null 2>&1 &      # 24시간으로 재시작
```

## pmset — 화면/전원 제어

```sh
pmset displaysleepnow   # 화면만 즉시 절전 (프로세스는 계속 실행)
pmset -g                # 현재 전원 관리 설정 조회
sudo pmset sleep 0      # 시스템 자동 슬립 비활성화 (설정 변경용)
```

`displaysleepnow`는 시스템을 잠그지 않고 **디스플레이만** 끈다. 화면은 알림 배너(예: Telegram 알림)가 뜨면 다시 켜졌다가 배너가 사라지면 자동 재절전된다 — 이상 현상이 아님.

> [!warning] 뚜껑(클램쉘)은 막을 수 없다
> caffeinate가 살아 있어도 **노트북 뚜껑을 닫으면 무조건 잠긴다.** 자리를 비울 때는 뚜껑을 열어둬야 백그라운드 작업(Telegram 봇 등)이 계속 돈다.

## 관련

- 실행 스크립트: `/Users/DongMyeongKang/.agents/skills/screen-off/scripts/screen-off.sh` (`--dry-run`으로 화면끄기 생략하고 판단 결과만 확인)
- 기본 셸 명령은 [[bash-ssh/bash-commands]]
- 백그라운드 유지 관련: [[bash-ssh/bash-script]]
