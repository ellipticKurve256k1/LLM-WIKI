---
title: ZCode 텔레그램 봇 채널과 완료 알림 구축
tags:
  - llm
  - zcode
  - telegram
  - bot
  - notification
  - hooks
aliases:
  - zcode telegram bot
  - zcode 텔레그램 알림
  - telegram-notify 스킬
  - zcode 봇 채널
created_date: 2026-09-13
---

# ZCode 텔레그램 봇 채널과 완료 알림 구축

ZCode(Z.ai 데스크톱 harness)의 텔레그램 봇 채널("elliptic") 구조를 app.asar 코드 분석 + 실측으로 파악하고, "데스크톱 작업 완료를 휴대폰 텔레그램으로 알림"을 달성하기까지의 과정을 정리한 노트. 2026-09-13 기준.

## 봇 채널의 파일 구성

| 위치 | 내용 |
| --- | --- |
| `~/.zcode/v2/bot-config.json` | 봇 정의 — 이름, provider, allowedCommands, replyMode, credentialRef |
| `~/.zcode/v2/bot-state.v2.json` | 봇 런타임 상태 — `mode`(draft/task), `activeTaskId`, draftOptions, telegramOffset |
| `~/.zcode/v2/credentials.json` | 봇 토큰 등 — **전부 암호화(`enc:v1...`)**, 스크립트에서 읽을 수 없고 macOS 키체인에도 없음. 토큰 필요하면 봇파더(/mybots)에서 재발급 확인 |

## 내장 명령어 (app.asar에서 확인)

`/help` `/bind` `/status` `/new`(=`/clear`) `/project` `/model` `/mode` `/think` `/reply` `/task` `/stop` `/reconnect`

- `/task`는 allowedCommands에 없어도 항상 허용되는 내장 명령. 현재 workspace의 최근 task 10개를 보여주고 선택 시 봇 상태를 그 task로 전환.
- `/reply`는 답변 전달 상세도 4단계: `assistant_changes` / `assistant_toolcalls_changes` / `summary_changes` / `streaming_card`.
- 권한 승인, plan 승인, 모델 질문(elicitation)도 텔레그램 버튼으로 처리 가능.

> [!important] /task 붙임의 실제 동작 (실측으로 확정)
> `/task` 전환은 봇 상태(`bot-state.v2.json`의 mode:task + activeTaskId)만 저장하고 **텔레그램 전달 구독을 걸지 않는다**. 구독(watchTaskStream)이 걸리는 경로는 3개: ① 텔레그램 메시지로 새 task 생성, ② 붙은 task에 텔레그램에서 일반 메시지(프롬프트) 전송, ③ 예약 자동화 실행. 구독은 턴 단위로 task_complete에서 해제된다. 또한 텔레그램 프롬프트는 데스크톱 세션의 프롬프트로 그대로 들어온다(양방향). 이 상태는 앱 재시작 후에도 유지됨.

## 시도했던 경로와 실패/한계

1. **`/task` 붙임으로 완료 알림** — 데스크톱에서 시작한 작업은 실행 중 구독을 시작할 방법이 없어 자동 푸시 불가. 붙여도 명령어(/reply 등)만 보내면 전달 안 됨.
2. **Stop 훅 + curl** — `/hooks` UI에 등록했더니 **저장 버그로 빈 events만 기록됨**(`{"hooks":{"events":{}}}`). → `~/.zcode/workspace/default/.zcode/config.json`에 직접 기록으로 우회. 이후 앱 재시작 시 trust 승인 프롬프트가 떴고(훅 선언 sha256 digest 검증), 승인 후 정상 발화. 요약 포함 버전까지 만들어 동작 확인.
3. **중복 발송 문제** — 같은 내용이 2통 도착. notify.log 진단 결과 훅은 1회만 발송 → 원인은 **이중 채널**: 훅 1통 + 봇 엔진이 붙은 task의 답변 원문 1통(replyMode 전달). 두 번째 메시지에 "✅" 헤더가 없다는 것이 발신원 구분의 단서.

## 최종 해결: telegram-notify 스킬

훅은 폐기(workspace config 원상복구)하고 스킬로 전환 — `~/.agents/skills/telegram-notify/`

- `scripts/send.sh "메시지"`: `~/.zcode/hooks/telegram.env`(토큰+chat_id, chmod 600)를 읽어 Telegram Bot API `sendMessage` 호출, JSON 응답 출력.
- 트리거: "끝나면 텔레그램으로 알려줘" 등. 작업 마무리 직전 1회, ✅/📢/⚠️ 라벨 + 결과 중심 불릿 요약(≤1,000자). 기계적 발췌가 아니라 모델이 직접 작성하므로 품질이 좋음.
- chat_id는 봇 1:1 채팅 기준 봇 설정의 `providerUserId`(본인 계정 ID)와 동일.

> [!tip] 운영 원칙
> **봇 붙임(/task + 프롬프트) = 실시간 모드, telegram-notify 스킬 = 완료 알림 모드. 동시에 쓰면 2통 간다.** 판단 기준: 대화가 텔레그램에서 시작됐으면 봇이 이미 전달 중(스킬 생략), 데스크톱에서 맡기고 자리를 떴으면 스킬 사용.

## 관련 파일

- 발송 스크립트: `~/.agents/skills/telegram-notify/scripts/send.sh`
- 시크릿: `~/.zcode/hooks/telegram.env` (`TG_BOT_TOKEN`, `TG_CHAT_ID`)
- 봇 설정/상태: `~/.zcode/v2/bot-config.json`, `~/.zcode/v2/bot-state.v2.json`
- 훅 스키마(참고): 워크스페이스 `.zcode/config.json`의 `hooks.events.{Stop,...}` — 이벤트 7종(SessionStart, UserPromptSubmit, PreToolUse, PermissionRequest, PostToolUse, PostToolUseFailure, Stop)

## 관련 노트

- 세션 대화 기록(rollout jsonl) 구조는 [[LLMS/zcode-chat-history-storage-and-deletion]] 참조 — 훅 요약 실험에서 `response.text`로 마지막 답변을 추출하는 데 사용했던 경로.
- 컨텍스트/MCP 로딩 관점의 harness 분석은 [[LLMS/zcode-codex-harness-context-analysis]], MCP 프록시 구성은 [[LLMS/zcode-mcp-lazy-proxy-setup]].
