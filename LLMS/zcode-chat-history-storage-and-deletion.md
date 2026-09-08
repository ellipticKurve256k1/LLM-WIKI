---
title: ZCode 대화 히스토리 — 저장 위치와 완전 삭제
tags:
  - llm
  - zcode
  - harness
  - sqlite
  - privacy
aliases:
  - zcode chat history
  - zcode 대화 삭제
  - zcode db.sqlite
  - zcode rollout jsonl
  - zcode tasks-index
created_date: 2026-09-01
---

# ZCode 대화 히스토리 — 저장 위치와 완전 삭제

ZCode(Z.ai 데스크톱 harness)가 대화를 어디에 저장하고, UI의 "보관(archive)"과 달리 실제로 지우는 방법을 정리한 노트. 2026-09 기준 macOS에서 직접 검증한 내용.

## 저장 구조

| 위치 | 내용 | 성격 |
| --- | --- | --- |
| `~/.zcode/cli/db/db.sqlite` (+`-wal`/`-shm`) | 대화 본체. `session`/`message`/`part` 테이블 | 영속 — 지우지 않으면 남음 |
| `~/.zcode/v2/tasks-index.sqlite` (+`-wal`/`-shm`) | **사이드바 목록의 실제 원천.** `tasks` 테이블(`task_id`=`sess_<uuid>`, `title`, `task_status`, `archived`, `deleted`)이 타이틀·상태·보관 여부를 인덱싱 | 영속 — db.sqlite만 지우면 여기가 남음 |
| `~/.zcode/cli/rollout/model-io-sess_<id>.jsonl` | 모델 입출력 raw 로그 (프롬프트/응답 원문). 가장 프라이버시 민감 | 비지속 — 앱이 주기적으로 자동 청소 |
| `~/.zcode/cli/config.json` | MCP 서버 등 앱 설정 | DB와 별개 — DB 삭제와 무관 |

`~/.zcode/v2/sessions/*.json`은 폐기된 구 레이아웃.

### db.sqlite의 테이블 (대화 전용이 아님)

대화 관련: `session`, `message`, `part` (message는 session에 cascade-delete), `input_history`, `session_*`, `todo`, `tool_usage`

대화 밖 데이터:
- `local_setting` — **프로젝트별 권한 허용 룰(allowed tools)·모드(yolo/edit)·reasoningLevel** 저장. DB를 지우면 다시 허용 클릭해야 하고 reasoning 설정도 초기화됨
- `model_usage` — 모델별 사용량 통계 (reasoning 토큰 비교 스크립트의 데이터 원천)
- `workflow_definition` 등 — 여기서 정의되지만 보통 비어 있음

## 보관(archive)의 실체

- Archive는 **아무것도 옮기지 않는다.** `db.sqlite`의 `session` 행에 `time_archived` 정수 타임스탬프가 찍히는 것뿐이고, message/part 행과 rollout jsonl은 제자리에 남는다 → 용량 회수 0
- 사실상 UI 숨김 플래그. 완전 제거는 여전히 아래 삭제 경로 필요
- (참고) `tasks-index.sqlite`의 `tasks` 테이블에도 별도의 `archived` 컬럼이 있어 목록 필터에 함께 쓰임

## 고스트 타이틀 문제 (db.sqlite를 지워도 사이드바가 남는 이유)

db.sqlite만 전체 삭제하면 재실행 시 사이드바에 타이틀은 그대로 남고 내용만 "없음" 상태가 된다. 원인: 사이드바 목록은 `~/.zcode/v2/tasks-index.sqlite`의 `tasks` 테이블에서 읽기 때문. 이 인덱스 DB까지 같이 지워야 목록이 완전히 비운다.

### 전체 삭제 (수정본)

```bash
# ZCode를 완전히 종료한 후 실행
rm ~/.zcode/v2/tasks-index.sqlite* \
   ~/.zcode/cli/db/db.sqlite* \
   ~/.zcode/cli/rollout/model-io-sess_*.jsonl
```

- `tasks-index.sqlite`도 다음 실행 때 빈 상태로 자동 재생성되므로 앱 구성에는 지장 없음
- **앱 종료가 필수인 이유**: SQLite WAL 모드. 실행 중 WAL 파일을 rm하면 공식 문서에 명시된 손상 경로. 종료 시 체크포인트가 되어 WAL/-shm이 정리됨
- 날아가는 부수 데이터를 아끼려면 삭제 전 `cp ~/.zcode/cli/db/db.sqlite ~/db-backup.sqlite`로 백업 후 `sqlite3`로 `local_setting`·`model_usage`만 골라 복원 가능

### 특정 세션만 삭제 (인덱스 포함)

```bash
# 사이드바 목록에서 제거
sqlite3 ~/.zcode/v2/tasks-index.sqlite "DELETE FROM tasks WHERE task_id='sess_xxxx';"
# 대화 본체에서 제거
sqlite3 ~/.zcode/cli/db/db.sqlite "DELETE FROM session WHERE id='sess_xxxx';"
# 세션 목록: SELECT id, title FROM session;
# 인덱스 DB의 task_groups/automations 등 다른 테이블은 대화 타이틀과 무관
```

- row 단위 DELETE는 실행 중에도 비교적 안전 (동시 쓰기 충돌만 주의). DB 파일 자체를 rm하는 것만 위험
- 용량 회수에는 `VACUUM`이 필요하고 WAL이 살아있으면 효과 제한적

### jsonl만 삭제

- `lsof` 확인 결과 파일 핸들이 잡혀있지 않아 **앱 실행 중에도 안전하게 unlink 가능** (macOS 특성상 앱은 계속 새 파일에 기록)
- 단, raw 모델 입출력 로그만 지워질 뿐 UI 대화 히스토리는 그대로 남음 (사이드바 목록은 tasks-index.sqlite, 대화 본문은 db.sqlite에서 읽음)
- 지워도 재생성됨: 새 API 호출마다 라인이 추가되며 2분 내로 파일이 다시 생김. 재생성을 멈추는 방법은 앱 종료뿐

## jsonl 포맷 참고

한 줄 = 모델 API 호출 1회. 키: `requestId`, `sessionId`, `turnId`, `model`, `request`, `response`. 대화 본문은 `request.messages`에 있음 (`request.body`는 파라미터만). 세션 타이틀 생성 호출도 별도 라인으로 기록됨.

## 관련 노트

- [[LLMS/zcode-codex-harness-context-analysis|하네스 컨텍스트 분석 — ZCode vs Codex CLI]] — `model_usage`를 이용한 하네스 컨텍스트 비용 실측
- [[LLMS/zcode-mcp-lazy-proxy-setup|ZCode용 mcp-lazy 설정 가이드]] — `config.json`에서 MCP 스키마 부하를 줄이는 실제 구성
- [[LLMS/mcp-tool-search-context]] — MCP 도구 정의가 context를 차지하는 문제 (ZCode config.json의 MCP 설정과 직결)
- [[LLMS/token-economics]] — `model_usage` 통계와 토큰 소비 분석
