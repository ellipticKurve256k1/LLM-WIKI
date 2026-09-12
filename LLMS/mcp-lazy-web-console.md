---
title: mcp-lazy Web Console (로컬 웹 관리 UI)
tags:
  - llm
  - mcp
  - zcode
  - tooling
  - macos
aliases:
  - mcp-lazy web
  - mcp-lazy 웹 콘솔
  - mcp-lazy-web
  - MCP 서버 관리 UI
created_date: 2026-09-11
---

# mcp-lazy Web Console

`~/.mcp-lazy/servers.json`을 손으로 편집하지 않고 브라우저에서 관리하는 **로컬 전용 단일 파일 웹 앱**. 백엔드 MCP 서버 추가·삭제·엔드포인트 수정, 백업/복원, ZCode·Claude Code 연결 등록까지 한 화면에서 처리한다. 2026-09-09에 구현·검증을 마쳤고, 같은 날 작성한 설계 문서(`prd.md`, `functional_specification.md`, `implementation_plan.md`)가 리포에 함께 있다.

- 위치: `~/Desktop/Dev_Study/mcp-lazy-web/` — 볼트 밖 리포다. git 저장소는 아니다.
- 실행 파일: `web-ui.js` (565줄). Node 22 내장 모듈만 사용, **의존성 0**. 배포에 필요한 파일은 이 하나뿐이다.

![[mcp-lazy-web-console.png]]

*2026-09-09, 실제 `~/.mcp-lazy/servers.json`을 읽은 화면. 당시 원격 3개(todoist·exa·firecrawl)만 등록돼 있었고 지금은 7개다. exa URL의 query는 화면에서 `[hidden]`으로 마스킹된다.*

## 1. 실행

```sh
cd ~/Desktop/Dev_Study/mcp-lazy-web
node web-ui.js          # → http://127.0.0.1:8079
```

prefix 설치처럼 `mcp-lazy`가 PATH에 없으면 읽기 전용으로 뜬다. 이때는 진입점을 명시한다.

```sh
node web-ui.js --mcp-lazy-path "$HOME/.mcp-lazy/prefix/node_modules/mcp-lazy/dist/index.js"
node web-ui.js --port 8080 --mcp-lazy-path /absolute/path/to/mcp-lazy
```

- 시작 디렉터리가 **Claude 프로젝트 설정의 기준**이 된다. Claude 프로젝트 항목을 관리하려면 해당 프로젝트 루트에서 실행한다.
- 포트가 사용 중이면 자동으로 다른 포트로 넘어가지 않고 종료한다.
- `Ctrl+C`(SIGINT/SIGTERM)로 앱과 진행 중인 적용 프로세스를 함께 종료한다. 브라우저 탭을 닫는 것만으로는 적용이 중단되지 않는다.
- 기동 로그가 `설정 관리 가능`인지 `읽기 전용: <사유>`인지로 실제 조작 가능 여부를 알 수 있다.

## 2. 화면 구성

탭 3개로 나뉜다.

| 탭 | 하는 일 |
|---|---|
| **서버** | 등록된 서버 목록(원격 / 로컬 stdio / 인식 불가 구분), 원격 서버 추가, URL 수정, 모든 유형 삭제. 저장 시 자동으로 적용(init) 실행 |
| **에이전트 연결** | ZCode·Claude 프로젝트·Claude 전역의 **현재 프로젝트 기준** 등록 상태 표시와 등록·해제 |
| **백업** | 대상별 최신 10개 백업 목록과 복원 |

새 원격 서버는 `npx -y mcp-remote`가 아니라 설치된 `~/.mcp-lazy/prefix/node_modules/mcp-remote/dist/proxy.js`를 Node로 직접 실행하도록 등록한다 — [[LLMS/zcode-mcp-lazy-proxy-setup]] §3 Step 7의 12초 → 0.6초 수정이 여기 코드로 굳어 있다. 기존 원격 항목이 direct Node / mcp-remote 실행 파일 / `npx -y mcp-remote` 중 어느 형태든 인식해서 URL 인자만 교체한다. **로컬 명령과 env는 편집하지 않는다.** 그래서 `DART_API_KEY` 같은 값은 여전히 `servers.json`을 직접 열어야 한다.

## 3. 건드리는 설정 파일

| 대상 | 경로 | 인식하는 구조 |
|---|---|---|
| 서버 | `~/.mcp-lazy/servers.json` | `servers` |
| Claude 프로젝트 | 실행 디렉터리의 `.mcp.json` | `mcpServers` |
| Claude 전역 | `~/.claude.json` | `projects[실행 디렉터리].mcpServers` |
| ZCode | `~/.zcode/cli/config.json` | `mcp.servers` |

없는 파일, 손상된 JSON, 미지원 구조, **심볼릭 링크와 다중 hard link는 자동 변경하지 않는다.** 이런 경우 해당 에이전트에는 설정 예시만 보여준다. 파일을 새로 만들어 주지는 않는다는 뜻이므로, 처음 연결하는 에이전트는 수동으로 컨테이너를 만들어야 한다.

## 4. 안전장치

이 앱의 핵심 설계는 "설정 파일을 잘못 건드리지 않는 것"에 맞춰져 있다.

- **변경 전 백업** — 각 설정 파일 옆 `.mcp-lazy-web-backups/<파일명>/`에 원본 바이트 그대로, 권한 `0600`. 대상별 최신 10개 보존.
- **외부 변경 감지 + 원자적 저장** — 저장 직전까지 외부 수정 여부를 확인하고, 검사와 rename 사이의 경쟁은 완전히 막을 수 없다(README의 명시된 한계).
- **보존** — 기존 파일 권한과 지원 대상 JSON 필드를 유지한다. 단 저장 시 들여쓰기·줄바꿈은 재정렬된다.
- **소유권 추적** — 앱이 만든 에이전트 등록은 `~/.mcp-lazy/web-ui-managed.json`에 경로별 식별자 + 항목 해시로 기록한다. 항목이 외부에서 바뀌었거나 기록을 못 읽으면 **외부 등록으로 간주해 자동 해제를 막는다.**
- **저장 ≠ 적용 성공** — 적용이 실패하거나 취소돼도 저장된 파일은 유지한다. 취소는 실행 중 작업과 대기 중 적용을 모두 중단하고, 다시 적용하면 파일의 최신 버전을 쓴다.
- **`tool-cache.json`은 직접 편집하지 않는다.** 기존 `mcp-lazy init`에 맡긴다. 에이전트 등록·해제는 서버 캐시를 바꾸지 않으므로 init을 돌리지 않고, 백업 복원은 명세대로 init을 실행한다.

## 5. 보안 모델

`127.0.0.1`에만 바인딩하고, 그 전제 위에 방어를 얹는다.

- Host 확인 + 변경 요청에 정확한 Origin + 시작 시 생성한 **세션 토큰** 요구. GET은 파일 변경이나 명령 실행을 하지 않는다.
- 프로세스는 shell 없이 고정 명령과 인자로만 실행한다.
- 출력은 청크 경계에서 줄을 조립한 뒤 환경변수·인증 헤더·알려진 비밀값·URL query를 마스킹해 일반 텍스트로 렌더링한다. 정제 로그는 메모리에 500줄만 두고 재시작하면 사라진다.
- OAuth 원본 링크는 적용 중 메모리에만 두고 토큰으로 보호된 요청에만 돌려준다.
- 설치된 mcp-remote는 OAuth 시 시스템 브라우저를 자동 실행하는데, 앱은 **자신의 파일을 apply의 Node 프로세스에 preload해 `open` 등 브라우저 실행 명령을 차단**하고 승인 URL을 텍스트로 노출한다. 설치 패키지와 서버 설정은 건드리지 않는다.
- 포트 포워딩이나 외부 네트워크 공개는 지원하지 않는다.

## 6. 검증 상태와 한계

- `node --check web-ui.js` + `node --test`로 자동 테스트 **21개**. 테스트는 임시 HOME·프로젝트와 가짜 init을 써서 실제 설정·OAuth·캐시를 건드리지 않는다. 저장 충돌, 실패 주입, 권한 보존, 백업 복원, 부분 실패, 연속 저장, OAuth 출력, 프로세스 트리 취소, 관리 소유권, 요청 보호를 검증한다.
- `scripts/browser-smoke.cjs`로 브라우저 흐름을 별도 검증한다(Playwright 별도 설치 필요, 앱 런타임 의존성 아님). `artifacts/`에 데스크톱·모바일 스크린샷이 남아 있다.
- 검증 환경은 **macOS 12.7.6 Intel / Node 22.17.0 / Brave**. 다른 환경은 미검증이다.

> [!warning] 검증하지 않은 부분
> **실제 원격 서비스의 OAuth 완료와 새 에이전트 세션에서 도구가 붙는 것까지는 확인하지 않았다.** README가 그렇게 명시하고 있다. 여기에는 실제 계정 승인과 설정 변경이 필요하다. 즉 "파일을 올바르게 쓰고 init을 돌린다"까지는 검증됐고, "그 결과가 실제 세션에 반영된다"는 것은 [[LLMS/zcode-mcp-lazy-proxy-setup]] §4 절차로 사람이 확인해야 한다.

기타 한계:

- 앱 자체에는 apply/OAuth 대기 시간 제한이 없다. mcp-lazy·MCP SDK·mcp-remote·원격 서비스의 제한이 그대로 걸린다.
- 설치본 init이 일부 서버 검색 실패에도 종료 코드 0을 반환할 수 있어 출력의 실패 표시도 함께 검사한다. 설치본 출력 형식이 바뀌면 인식 규칙을 손봐야 한다. 서버가 0개면 init이 실패로 종료하고 UI는 그대로 보여준다.
- 동시에 여러 앱 인스턴스로 같은 설정을 편집하는 사용은 피한다.
- 실행 상태와 적용 완료 버전은 앱 메모리에만 있고, 재시작하면 과거 적용 성공을 추정하지 않는다.
- 내장 아이콘은 Lucide static 0.468.0(ISC), 라이선스 고지를 `web-ui.js`에 포함.

## 7. 설치 절차 노트와의 관계

[[LLMS/zcode-mcp-lazy-proxy-setup]] §4의 수동 추가 절차(`servers.json` 편집 → init)를 **대체하는 것이 아니라 UI로 감싼 것**이다. 백엔드 런처 패턴, `servers` 키, init의 역할은 동일하다.

- 이 앱은 **서버를 추가/삭제/URL 수정**한다. 툴 캐시 재생성은 init에 위임한다.
- 로컬 서버의 `env`(API 키)는 편집하지 않으므로, 키 설정은 여전히 수동이다.
- 서버 목록 자체의 현재 상태는 [[LLMS/mcp-lazy-server-inventory]]가 정본이다. 앱으로 서버를 바꿨으면 그 노트의 §1·§2 표를 갱신한다.

## Related Notes

- [[LLMS/zcode-mcp-lazy-proxy-setup]] — 설치·유지보수·트러블슈팅, 서버 추가 절차 (§4)
- [[LLMS/mcp-lazy-server-inventory]] — 등록된 서버 현황 정본. 이 앱이 편집하는 파일의 내용
- [[LLMS/mcp-tool-search-context]] — mcp-lazy가 왜 필요한지(메타툴 2개로 컨텍스트 절감)
- [[LLMS/mcp-remote-transport]] — 원격 서버를 stdio로 브리지하는 transport 배경, OAuth 토큰 위치
- [[LLMS/Subagents-Format/agents-guide]] — 에이전트별 MCP 설정 문법
