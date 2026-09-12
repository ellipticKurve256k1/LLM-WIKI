---
title: mcp-lazy 등록 MCP 서버 현황
tags:
  - llm
  - mcp
  - zcode
  - inventory
  - macos
aliases:
  - MCP 서버 현황
  - mcp-lazy 서버 목록
  - MCP inventory
created_date: 2026-09-11
---

# mcp-lazy 등록 MCP 서버 현황

`~/.mcp-lazy/servers.json`에 등록된 **현재** 서버 목록을 관리하는 인벤토리 노트. 설치 절차·트러블슈팅은 [[LLMS/zcode-mcp-lazy-proxy-setup]]에 있고, 이 노트는 "지금 무엇이 꽂혀 있고 어떻게 인증되는가"만 다룬다. 서버를 추가/삭제하면 이 표부터 갱신한다.

> [!info] 확인 시점과 방법
> 2026-09-11 확인. 근거는 `~/.mcp-lazy/servers.json`과 `~/.mcp-lazy/tool-cache.json`(09-11 18:01 생성, fingerprint `4feb2997…`) 직접 조회. 에이전트가 보는 목록이 아니라 **프록시에 등록된 실제 백엔드** 기준이다.

## 1. 요약

- **7개 서버 / 154개 툴** (todoist 47, hwp 35, firecrawl 25, korean-dart 18, real-estate 16, korean-law 10, exa 3)
- 원격 3개(todoist·exa·firecrawl)는 모두 `mcp-remote` 브리지, 로컬 4개(korean-law·hwp·korean-dart·real-estate)는 로컬 프로세스
- 툴 스키마 전체를 직렬화하면 **약 246KB ≈ 61k tokens(추정)**. mcp-lazy는 이걸 메타툴 2개(~300 tokens)로 대체한다 → [[LLMS/zcode-mcp-lazy-proxy-setup]]의 실측(세션당 ~15k → ~1.5k) 참조
- ChatGPT/Claude 시절과 달리 한국어 도메인 서버(법령·HWP·DART·부동산) 비중이 크다

## 2. 서버 목록

| 서버          | 종류  | 실행 방식                                                                                          | 툴   | 인증                   | 출처                                                                          |
| ----------- | --- | ---------------------------------------------------------------------------------------------- | --- | -------------------- | --------------------------------------------------------------------------- |
| todoist     | 원격  | `mcp-remote` → `https://ai.todoist.net/mcp`                                                    | 47  | OAuth                | Todoist 공식 MCP                                                              |
| exa         | 원격  | `mcp-remote` → `https://mcp.exa.ai/mcp?client=agent-plugin` (+`x-exa-source` 헤더)               | 3   | OAuth (`mcp:tools`)  | Exa 공식 MCP                                                                  |
| firecrawl   | 원격  | `mcp-remote` → `https://mcp.firecrawl.dev/v2/mcp-oauth`                                        | 25  | OAuth                | Firecrawl 공식 MCP                                                            |
| korean-law  | 로컬  | `/usr/local/bin/korean-law-mcp` (전역 npm v4.13.0)                                               | 10  | `LAW_OC`             | [chrisryugj/korean-law-mcp](https://github.com/chrisryugj/korean-law-mcp)   |
| hwp         | 로컬  | `node …/prefix/node_modules/hwp-mcp/dist/server.js` (v0.3.0)                                   | 35  | —                    | [treesoop/hwp-mcp](https://github.com/treesoop/hwp-mcp)                     |
| korean-dart | 로컬  | `node …/prefix/node_modules/korean-dart-mcp/build/index.js` (v0.10.1)                          | 18  | `DART_API_KEY`       | [chrisryugj/korean-dart-mcp](https://github.com/chrisryugj/korean-dart-mcp) |
| real-estate | 로컬  | `uv run --directory ~/mcp-servers/real-estate-mcp python src/real_estate/mcp_server/server.py` | 16  | `DATA_GO_KR_API_KEY` | [tae0y/real-estate-mcp](https://github.com/tae0y/real-estate-mcp)           |

## 3. 서버별 메모

- **todoist** — 가장 큰 서버(47). `find-tasks-by-date`, `add-tasks`, `complete-tasks`(ids 배열), `update-tasks`, `reschedule-tasks`가 주력. 우선순위는 `"p1"`~`"p4"` 문자열만 허용(정수 불가). `search`/`fetch`는 서버 일반 검색용. 작업 관리는 전부 이 서버를 거치므로 사실상 매 세션 사용된다.
- **exa** — 3개뿐: `web_search_exa`, `web_fetch_exa`, `agent_run`. URL만 보면 익명처럼 보이지만 **OAuth로 인증되어 있다** — `mcp-remote`가 최초 연결 때 브라우저 승인을 거쳐 Exa 발급 refresh token(`exart_…`)·JWT access token(scope `mcp:tools`)을 저장하고, 접속마다 자동 갱신한다. ZCode의 exa **플러그인은 꺼져 있고**(`exa@claude-plugins-official: false`) 툴만 프록시로 들어와 있다. 플러그인을 끄면 번들 스킬(`exa:search`, `exa:exa-agent`)도 함께 사라지는 트레이드오프가 있다.
- **firecrawl** — 스크래핑·크롤링·맵·검색에 더해 `firecrawl_monitor_*`(모니터링), `firecrawl_research_*`(논문), `firecrawl_agent`까지 포함한 25개. 2026-09-08 추가.
- **korean-law** — 국가법령정보(법제처) 조회. 흥미로운 점은 **자체 `discover_tools`/`execute_tool` 메타툴을 내장**해 서버 안에서도 lazy 발견을 한다는 것. `LAW_OC`는 법제처 신청 OC 값(= `dongyukang`).
- **hwp** — 한글 문서 툴킷 35개: 읽기(`read_hwp_text`, `read_hwp_tables`), 편집(`replace_hwp_text`, `set_hwp_cell_text`), 표/셀 병합, 렌더(`render_hwp_page`, `render_hwp_html`), 신규 생성(`create_hwpx_document`)까지.
- **korean-dart** — OpenDART 공시. `search_disclosures`, `get_financials`, `get_shareholders`, `insider_signal` 등 분석용 파생 툴이 많고 `dart_raw`로 원시 호출도 가능. `DART_API_KEY` 필요.
- **real-estate** — 국토부 실거래가. 아파트/오피스텔/빌라/단독 거래·전월세, 청약 공고/결과, 지역코드 조회 + `calculate_loan_payment` 같은 계산 툴. 로컬 리포를 `uv run`으로 띄우는 **유일한 파이썬 서버**라 `uv` 경로(`~/.local/bin/uv`)와 리포 경로가 깨지면 조용히 실패한다.

> [!warning] API 키는 이 노트에 옮기지 않았다
> `servers.json`은 `DART_API_KEY`, `DATA_GO_KR_API_KEY`, `LAW_OC`를 **평문**으로 담고 있고, 이 볼트는 git 리포다. 값이 필요하면 `cat ~/.mcp-lazy/servers.json`으로 직접 확인하고 노트·커밋에는 절대 복사하지 않는다.

## 4. 인증(토큰) 상태

OAuth가 필요한 서버만 `~/.mcp-auth/mcp-remote-v1/`에 항목을 만든다 (mcp-remote가 URL을 해시해 파일명으로 쓴다).

| 해시(앞 8자)   | 대응 서버   | client_info 발급 | tokens 갱신  |
| ---------- | ------- | -------------- | ---------- |
| `8e269e0b` | todoist | 2026-09-07     | 2026-09-11 |
| `bc0fa3a0` | firecrawl | 2026-09-08     | 2026-09-11 |
| `3499d73f` | exa     | 2026-09-07     | 2026-09-11 |

- 토큰 파일명의 해시는 `md5(serverUrl | headers …)`다 — URL이 같아도 `--header`(exa의 `x-exa-source`)가 붙으면 별도 해시가 나온다. 그래서 exa 해시(`3499d73f`)는 URL만 해시한 값과 일치하지 않는다.
- 세 서버 모두 OAuth이며, 토큰이 살아 있으면 `init`이 브라우저 팝업 없이 통과한다 (2026-09-11 검증).
- 토큰이 없어지면 `init`이나 첫 툴 호출 때 브라우저 승인 창이 뜬다. 정상 동작이니 놀라지 말 것.

## 5. 에이전트 측 연결 현황

| 에이전트  | 설정 파일                      | MCP 연결                                                                              |
| ----- | -------------------------- | ----------------------------------------------------------------------------------- |
| ZCode | `~/.zcode/cli/config.json` | **`mcp-lazy` 하나만** (stdio, `node …/mcp-lazy/dist/index.js serve`)                   |
| Codex | `~/.codex/config.toml`     | firecrawl·korean-law·korean-dart를 **직접 등록**(프록시 밖) + `elliptickurve256k1`(Smithery) |

- ZCode 플러그인 상태: `context7`, `browser-use` 활성(프록시 밖에서 자체 MCP 제공), `exa`·`zcode-guide`·`computer-use` 비활성.
- Codex 쪽 firecrawl·korean-law·korean-dart는 같은 URL/경로를 직접 물고 있어 **mcp-lazy와 중복**이다. 프록시 효과를 보려면 정리 대상 — [[LLMS/zcode-codex-harness-context-analysis]]에서도 지적된 항목.

## 6. 관리 절차

아래는 수동(CLI) 절차다. 같은 작업을 브라우저에서 하려면 [[LLMS/mcp-lazy-web-console]]을 쓴다 — 서버 추가·삭제·URL 수정과 백업/복원을 UI로 처리하고 init까지 자동 실행한다. 단 로컬 서버의 `env`(API 키)는 UI가 편집하지 않으므로 키 설정은 아래 수동 절차가 필요하다.

**서버 추가** — [[LLMS/zcode-mcp-lazy-proxy-setup]] §4와 동일:

1. `~/.mcp-lazy/servers.json`의 `servers`에 엔트리 추가 (최상위 키는 `mcpServers`가 아니라 **`servers`**)
2. 백엔드 런처는 npx 대신 `node /Users/DongMyeongKang/.mcp-lazy/prefix/node_modules/mcp-remote/dist/proxy.js <url>` 패턴 복사 (2026-09-08 전환, 12초 → 0.6초)
3. `node ~/.mcp-lazy/prefix/node_modules/mcp-lazy/dist/index.js init` — 연결 확인 + `tool-cache.json` 재생성 (OAuth 서버는 여기서 브라우저 1회)
4. **이 노트의 §1·§2 표 갱신**

**현황 확인 명령**

```bash
# 등록된 서버와 실행 방식
node -e 'console.log(Object.entries(require(process.env.HOME+"/.mcp-lazy/servers.json").servers).map(([k,v])=>k+" | "+v.command+" "+v.args.join(" ")).join("\n"))'

# 서버별 툴 수 (캐시 기준)
node -e 'const m={};for(const t of require(process.env.HOME+"/.mcp-lazy/tool-cache.json").tools)m[t.server]=(m[t.server]||0)+1;console.log(m)'
```

**갱신 시점** — ① 서버 추가/삭제 ② 툴 수가 눈에 띄게 변했을 때(백엔드 업데이트) ③ OAuth 재인증이 발생했을 때. `tool-cache.json` mtime이 마지막 `init` 시각이다.

**주의** — prefix 설치는 자동 갱신되지 않는다: `npm update --prefix ~/.mcp-lazy/prefix`. korean-law만 전역 npm(`/usr/local/lib/node_modules`) 설치라 갱신 경로가 다르다.

## 7. 갱신 이력

| 날짜         | 변경                        |
| ---------- | ------------------------- |
| 2026-09-11 | 최초 작성. 7개 서버 / 154개 툴 기준. |
| 2026-09-11 | 정정: exa는 익명이 아니라 OAuth 인증 서버였음. `3499d73f`는 잔재가 아니라 exa 본인 토큰 파일(URL+헤더 해시)로 확인, `init` 무팝업 통과로 검증. |

## Related Notes

- [[LLMS/zcode-mcp-lazy-proxy-setup]] — 설치·유지보수·트러블슈팅 절차 (이 노트의 절차 파트)
- [[LLMS/mcp-lazy-web-console]] — 이 파일들을 브라우저에서 관리하는 로컬 웹 앱
- [[LLMS/mcp-tool-search-context]] — 메타툴 2개가 컨텍스트를 어떻게 줄이는지
- [[LLMS/zcode-codex-harness-context-analysis]] — ZCode/Codex의 MCP 컨텍스트 실측과 Codex 중복 등록 지적
- [[LLMS/mcp-remote-transport]] — 원격 서버를 stdio로 브리지하는 transport 배경
- [[LLMS/Subagents-Format/agents-guide]] — 에이전트별 MCP 설정 문법
