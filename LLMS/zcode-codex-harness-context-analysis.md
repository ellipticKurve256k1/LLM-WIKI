---
title: 하네스 컨텍스트 분석 — ZCode vs Codex CLI
tags:
  - llm
  - mcp
  - zcode
  - codex
  - harness
  - context-window
aliases:
  - ZCode Codex 컨텍스트 비교
  - ZCode vs Codex CLI
  - harness context analysis
created_date: 2026-09-06
---

# 하네스 컨텍스트 분석: ZCode vs Codex CLI

작성일: 2026-09-06
측정 환경: ZCode (GLM 모델 세션) / Codex CLI 0.153.4 (gpt-5.6-sol, effort=medium)

## 1. 목적

MCP 툴 정의가 컨텍스트에서 차지하는 비중과, 두 하네스의 MCP 로딩 방식 차이를 실측으로 확인한다. 계기는 "ZCode 세션에서 MCP 툴이 컨텍스트의 약 50%를 차지한다"는 관측과, "ChatGPT 웹에서 설치한 앱 10여 개가 Codex CLI에서도 전부 적재되는 것 아닌가"라는 우려.

## 2. 측정 방법

- **토큰 수치**: ZCode는 `~/.zcode/cli/db/db.sqlite`의 `model_usage` 테이블에서 세션별 `input_tokens`를 조회. Codex는 `codex exec` 실행 후 출력되는 "tokens used"와 세션 롤아웃 파일(`~/.codex/sessions/...jsonl`)을 사용.
- **구성 분석**: Codex는 롤아웃 파일에 시스템 프롬프트·메시지가 원문으로 기록되므로 이를 직접 파싱. ZCode는 섹션별 분해를 API가 제공하지 않으므로 툴 정의(description + JSON schema) 크기 기준 추정(오차 ±15%).
- **Codex 툴 확인**: 모델에게 "사용 가능한 툴 이름을 전부 나열하라"고 지시하는 방식으로 실제 요청에 실린 툴을 검증.

## 3. ZCode 측정 결과

이 세션의 첫 요청 입력: **약 35,700 토큰** (db.sqlite 실측).

### 3.1 MCP 툴 점유율 (전체 약 15k 토큰, 컨텍스트의 약 40~50%)

| 서버                      | 툴 수 | 추정 토큰   | MCP 내 비중 |
| ----------------------- | --- | ------- | -------- |
| Todoist                 | 46  | ~13,000 | ~80%     |
| exa                     | 3   | ~950    | ~5%      |
| browser-use (node_repl) | 4   | ~700    | ~4%      |
| context7                | 2   | ~450    | ~3%      |

- Todoist가 압도적 원인. 툴 개수 자체가 Firecrawl(26개)보다 많고, 스키마가 무겁다. `add-tasks`는 속성 20+개, `find-activity`는 긴 설명, `find-tasks-by-date`는 검증용 정규식 패턴을 통째로 포함 → 툴 하나가 500~700 토큰.
- 과거 Firecrawl로 인한 40% 부하는 이미 서버 `enabled: false`로 해소된 상태. 현재의 부하는 2026-09-01 Todoist MCP 추가가 거의 전부.

### 3.2 스키마란 (배경 설명)

- 스키마 = 데이터/구조의 규격("설계도"). LLM 맥락에서는 ① 툴 정의 스키마(모델이 툴을 쓰기 위해 읽는 설명서)와 ② 응답 스키마(structured output 강제)가 있다.
- 핵심: LLM은 스키마를 자연어로 읽어야 하므로 스키마 자체가 프롬프트 텍스트(토큰)로 변환되어 매 요청마다 실린다. 필드가 많고 검증이 엄격한 API일수록 스키마가 커지는 트레이드오프가 있다.

## 4. Codex CLI 측정 결과

### 4.1 기본 컨텍스트: 9,622 토큰 ("hello" 세션 실측)

| 구성 요소                            | 크기                 | 비중   |
| -------------------------------- | ------------------ | ---- |
| 기본 시스템 프롬프트 (base_instructions)  | 17,730자 ≈ ~4,400토큰 | ~46% |
| 스킬 목록 (skills_instructions)      | 10,450자 ≈ ~2,600토큰 | ~27% |
| 플러그인 광고 목록 (recommended_plugins) | 3,125자 ≈ ~780토큰    | ~8%  |
| 멀티에이전트 페르소나 (/root) + 해제 지시      | ~2,500자 ≈ ~640토큰   | ~7%  |
| 빌트인 툴 스키마 + 환경 정보                | 미기록(추정) ~1,500토큰   | ~15% |

특이사항:

- Codex도 ZCode의 `~/.agents/skills`를 스킬 루트로 공유한다 (r0로 확인).
- "설치 안 된 플러그인" 광고 목록이 사용자 메시지 자리에 실린다 (~800토큰).
- 멀티에이전트 페르소나를 싣고 바로 뒤에 `<multi_agent_mode>` 태그로 "서브에이전트 쓰지 마세요"를 되돌리는 중복 구조.

### 4.2 ChatGPT 웹 설치 앱은 툴 스키마로 적재되지 않는다 (핵심 우려 해소)

- 툴 전체 나열 요청 결과: 빌트인 22개 + `web.run` + `imagegen` 뿐. Todoist 등 앱 툴은 0개.
- ChatGPT 웹의 커넥터 설치는 Codex CLI에 툴 스키마로 이식되지 않는다. config.toml에는 `apps.asdk_app_*` 형태의 승인 모드 설정만 존재하고, 툴은 로드되지 않음.
- Codex의 설계: 플러그인은 목록 미리보기(광고)만 싣고, 필요 시 `request_plugin_install` 툴로 그때 설치·로드.
- 단, 앱이 Codex CLI에 "등록 자체가 안 된 것"은 아니다 — 4.4에서 확인하듯 `codex_apps` 카탈로그에 메타데이터로 존재하며, 필요 시 동적으로 호출된다. "미적재"는 정확히는 "툴 스키마 미선적재"를 의미한다.

### 4.3 MCP 지연 로딩의 실제 동작 (3단계 실측)

| 단계                         | 결과                                                                 | 토큰(누적)         |
| -------------------------- | ------------------------------------------------------------------ | -------------- |
| ① 기본 세션                    | MCP 툴 스키마 0개. `list_mcp_resources` 등 범용 탐색 툴만 노출                   | 9,622 ~ 10,178 |
| ② `list_mcp_resources` 호출  | 결과 JSON **48,008자**가 한 번에 컨텍스트로 유입. 단, MCP 툴 스키마는 여전히 0개 (툴 목록 불변) | 58,563         |
| ③ firecrawl scrape 툴 실제 호출 | 성공 (example.com, status 200). 스키마 사전 주입 없이 서버명 기반 동적 호출이 가능        | 20,186         |

**정정 사항**: 중간 분석에서 "MCP 사용 시 툴 스키마가 한꺼번에 주입된다"고 추정했으나, 재실측 결과 그 증가분은 리소스 목록 덤프였고 스키마 주입은 아니었다. Codex의 툴 호출은 스키마를 컨텍스트에 싣지 않는 동적 호출 메커니즘으로 동작한다.

### 4.4 `list_mcp_resources`의 정체 — 상세 해부 (원본 JSON 확보 후 실측)

48k 페이로드의 원본을 파일로 덤프해 직접 파싱한 결과, 그것은 firecrawl·elliptickurve256k1 같은 등록 MCP 서버의 리소스가 아니라 **ChatGPT 앱/커넥터 카탈로그**였다.

**배경 — MCP "리소스"란:** MCP 프로토콜에서 리소스(resource)는 서버가 모델에게 제공하는 읽기 전용 데이터(파일, 문서, DB 레코드 등)다. 툴이 "실행"이면 리소스는 "조회"다. `list_mcp_resources`는 연결된 모든 서버에 리소스 목록을 요청해 합쳐 돌려주는 범용 탐색 툴이다.

**실측 결과 (덤프 파일 `/tmp/mcp_resources.json`, 50,895바이트, 내부 JSON 48,008자):**

- 모든 엔트리의 `server` 값이 `codex_apps` — ChatGPT 웹 앱/커넥터 통합 계층이 하나의 가상 MCP 서버로 동작한다는 뜻.
- 구성: `plugin://` URI 19개(앱·커넥터) + `skill://` URI 27개(플러그인 제공 스킬). Codex가 보고한 전체 개수는 59개.
- plugin 항목 예시: AllTrails, Binance, Bitdefender, CamScanner, Canva, CoinGecko, **Context7, Exa, Firecrawl, GitHub, Gmail**, Google Calendar/Drive/Sheets/Slides, Healthcare Public Data, Hugging Face.
- skill 항목 예시: HF CLI, Datasets, Papers, Mermaid Diagrams, sites-building, URL To Code, Product Design 등.
- 엔트리당 필드: `uri`, `name`, `title`, `description`, `contents`, `skills`, `required scopes`, `allow_implicit_invocation`, `plugin_release_skill_id` 등 풍부한 메타데이터 → 엔트리당 평균 ~800자로 48KB에 도달.

**해석:**

1. **ChatGPT 웹에서 설치한 앱(Todoist, Exa, Gmail 등)은 Codex CLI에 "등록"되어 있다.** 툴 스키마로 선적재되지 않을 뿐, 카탈로그에 메타데이터로 존재하고 모델이 필요로 할 때 동적으로 호출된다.
2. **트랩의 실체**: 모델이 리소스 탐색을 한 번만 해도 앱 카탈로그 전체(~48KB ≈ 1.2만~1.5만 토큰)가 그 턴의 컨텍스트에 유입된다. "lazy = 공짜"가 아니라 "lazy = 쓰는 순간 대량 유입"이다.
3. **페이로드는 동적이다**: 동일한 호출이 어떤 실행에서는 0개(세션 9.4k 토큰), 어떤 실행에서는 59개(48k자)를 돌려줬다. 카탈로그는 서버·인증 상태에 따라 변동된다.
4. Codex는 이 JSON을 모델 컨텍스트에 그대로 통째로 전달한다. 요약·필터링 없음.

## 5. 두 하네스의 로딩 방식 비교

| 항목       | ZCode                         | Codex CLI                          |
| -------- | ----------------------------- | ---------------------------------- |
| 로딩 방식    | 정적(전량 사전 적재)                  | 지연(온디맨드) + 동적 호출                   |
| 기본 컨텍스트  | ~35.7k (MCP 55개 활성 시)         | ~9.6k                              |
| 툴 스키마    | 세션 시작 시 전부 적재, 세션 내내 유지       | 원칙적으로 미적재, 서버명 기반 호출               |
| 비용 구조    | 안 써도 매 세션 지불 (Todoist ~13k)   | 쓴 만큼 지불, 단 리소스 목록 조회 시 대량 유입(48k자) |
| 모델의 툴 인식 | 완전 (정확한 호출 가능)                | 부분 (발견·호출 정확도가 떨어질 여지)             |
| 비활성화 방법  | 서버 단위 `enabled: false` + 새 세션 | 기본적으로 이미 지연 로딩                     |

비유: ZCode는 도구를 책상 위에 전부 펼쳐놓는 방식, Codex는 서랍에 넣어두고 필요할 때 꺼내 쓰는 방식.

## 6. ZCode에서 지연 로딩을 구현하려면 (하네스 측 개발 필요 여부)

**결론: 에이전트 코어 연결이 필요하나, 프로토콜 계층은 이미 준비되어 있다.** 설치된 ZCode 번들(`zcode.cjs`, 12MB) 분석 결과:

- **Anthropic 방식 지원 코드 존재**: `tool_search_tool_regex/bm25` 서버 사이드 툴 검색 프로토콜(`anthropic.tool_search_regex_20251119`)과 검색 결과로 툴을 주입하는 `tool_reference` 처리 코드가 이미 포함. 이것이 Claude Code의 지연 로딩 메커니즘이다.
- **OpenAI 방식 지원 코드 존재**: `defer_loading` 필드를 OpenAI API로 전달하는 패스스루 코드 존재 (GPT-5.4+ 네이티브 툴 지연 로딩).
- **없는 것**: 에이전트 코어가 MCP 툴을 defer 대상으로 표시하는 로직, MCP용 tool_search 메타툴, 사용자 노출 설정. 현재는 MCP 툴 55개가 전부 eager 적재된다. 즉 프로토콜 계층은 완료, 코어 연결이 남은 상태.

**공식 roadmap**: 공개된 것을 찾지 못했다. 다만 업계 수렴 방향은 명확하다:

| 하네스/API     | 상태                                                                       |
| ----------- | ------------------------------------------------------------------------ |
| Claude Code | Tool Search 기본 탑재 (툴 정의가 컨텍스트 10% 초과 시 자동 활성화, `ENABLE_TOOL_SEARCH`로 제어) |
| OpenAI API  | GPT-5.4+ 네이티브 `defer_loading` + `tool_search` 지원 (토큰 최대 47% 절감 보고)       |
| OpenCode    | 가장 많은 커뮤니티 요청, `mcp_lazy` 실험적 플래그 + PR 진행 중                              |
| zeroclaw    | `mcp.deferred_loading` 기본값 true로 채택                                      |

**진짜 병목**: ZCode는 Z.ai GLM API를 직접 호출하며, 지연 로딩은 API 제공자의 서버 사이드 지원을 전제로 한다(Anthropic은 tool_reference, OpenAI는 defer_loading). Z.ai가 GLM API에 동등한 메커니즘을 제공해야 한는 것이 ZCode 코어 개발보다 큰 전제 조건이다.

## 7. 즉시 적용 가능한 우회책: smart MCP 프록시 (mcpproxy-go 방식)

서버 토글(권고 1) 외에, ZCode 코드 수정 없이 Codex식 지연 로딩을 재현하는 방법이 있다.

**동작 원리**: 클라이언트(ZCode)와 실제 MCP 서버들(Todoist, Firecrawl 등) 사이에 프록시를 하나 둔다. ZCode에는 프록시 하나(툴 1~2개)만 보이고, 프록시가 내부에 연결된 수십 개 서버의 툴 정의를 대신 보관한다.

1. ZCode에는 프록시의 `discover`/`call` 같은 메타툴 1~2개만 보인다. Todoist 46개 툴(~13k 토큰) 대신 수백 토큰.
2. 모델이 할 일을 기술하면 프록시가 BM25 유사도 검색으로 관련 툴만 선별해 노출.
3. 필요한 툴만 그때그때 모델 컨텍스트에 주입되고 호출은 프록시가 중계.

**대표 구현**: [mcpproxy-go](https://github.com/smart-mcp-proxy/mcpproxy-go) — 모든 MCP 클라이언트와 무관하게 동작(클라이언트 코드 변경 불필요), 툴 설명 기반 BM25 검색, 실사용 보고로 툴 정의 토큰 85~97% 절감.

**장단점:**

- 장점: ZCode 업데이트와 무관하게 즉시 사용 가능. 여러 MCP 서버를 하나의 프록시 엔트리로 통합해 config도 단순해짐.
- 단점: 로컬에서 프록시 프로세스를 상시 실행해야 함. 검색이 부정확하면 모델이 필요한 툴을 못 찾을 수 있음(툴 설명 품질에 의존). 프록시 자체의 신뢰성·보안(토큰 위임)을 검토해야 함.

**mcpproxy-go가 이 환경에서 막힌 이유 (2026-09-07 확인)**: 설치자는 "macOS 10 or later" 표기였으나 실행 시 macOS 13+ 요구. Docker Desktop은 현재 macOS 12 미지원(구버전은 EOL), colima 공식 빌드도 macOS 13+ 요구. 소스 빌드(Go 1.25)는 이론상 가능하나 아래의 Node 기반 대안이 더 단순해 기각.

### 7.1 실제 도입: mcp-lazy (2026-09-07 세팅 완료)

mcpproxy-go 대신 Node 기반 프록시 **mcp-lazy 0.1.7**(npx, Node 18+)를 도입했다. stdio 서버라 ZCode가 세션마다 직접 띄우므로 상시 데몬·포트·launchd가 전혀 필요 없다.

**구조:**
```
ZCode ──(stdio)── mcp-lazy proxy (mcp_search_tools / mcp_execute_tool 2개, ~303 토큰)
                      └─(npx mcp-remote, OAuth 토큰 로컬 캐시)── Todoist (47 툴)
```

**실측 결과 (프록시 단독 검증):**

| 항목 | 직접 연결 | mcp-lazy 경유 |
|---|---|---|
| 세션에 적재되는 툴 스키마 | 46개 ≈ ~13,000 토큰 | 2개 = 1,211바이트 ≈ **~303 토큰 (97.7% 절감)** |
| 실제 호출 (`find-tasks-by-date`) | — | 성공, 실제 태스크 데이터 반환 |
| OAuth | ZCode 자체 토큰 | mcp-remote 토큰 1회 발급 후 로컬 저장, 재인증 불필요 |

**세팅 과정에서 밝혀진 실무 디테일:**

1. mcp-lazy의 `~/.mcp-lazy/servers.json` 스키마는 `{servers: {이름: {command, args, env?, description?}}}` — README에 나오는 `{mcpServers: ...}`는 에이전트 설정 읽기용이지 저장 포맷이 아니다 (소스 확인).
2. OAuth는 init 시 브라우저가 자동으로 열리고 1회 승인. 이후 토큰이 로컬에 저장되어 프록시 재시작에도 재인증 없음.
3. 이 Mac의 npm 캐시 권한 문제(EACCES)가 있어, 프록시 서버 항목의 `env.npm_config_cache`를 `~/.mcp-lazy/npm-cache`(전용)로 지정해 우회.
4. **검색 언어 주의**: 검색이 툴 이름·설명의 부분문자열 매칭이라 툴 설명이 영어인 관계로 `"오늘 할 일 보여줘"` 같은 한국어 자연어 쿼리는 결과 0건. `"today tasks due date"`로 검색하면 `find-tasks-by-date` 1순위로 정확히 반환. → 모델은 `mcp_search_tools`를 항상 영어 키워드로 호출해야 한다.

**ZCode config 변경 (`~/.zcode/cli/config.json`):**

- `Todoist` (http 직접 연결): `enabled: false` — 프록시 뒤로 이동
- `mcp-lazy` (stdio): `{command: "npx", args: ["-y", "mcp-lazy", "serve"], env: {npm_config_cache: ...}}` 신규 등록
- `exa@claude-plugins-official` 플러그인: `false` — exa도 프록시 뒤로 이동 (2026-09-07 추가). exa는 인증 없는 HTTP 서버(`https://mcp.exa.ai/mcp?client=agent-plugin`)라 mcp-remote 브리지에 `--header "x-exa-source: agent-plugin"`만 추가해 등록. 검색·실제 웹 검색 호출 모두 프록시 경유 검증 완료 (`"web search internet"` → `exa.web_search_exa` 1순위, agent_run·web_fetch_exa도 각각 정확히 매칭). 프록시 툴 캐시는 총 50툴(todoist 47 + exa 3).
- context7/browser-use 플러그인은 그대로 (툴 수가 적어 프록시 돌릴 가치 없음)
- 백업: `~/.zcode/cli/config.json.bak-20260907`

**롤백**: 백업 복원 한 줄. npx 방식이라 언인스톨 불필요 (`~/.mcp-lazy/` 디렉터리 삭제로 완전 청소).

**효과 전망**: 새 세션의 MCP 토큰이 ~15k → ~1.2k로 감소 (Todoist 13k + exa 0.95k → 0.3k, context7/browser-use 유지). 세션 첫 입력 ~35.7k → ~22k 예상.

## 8. 결론 및 권고

1. **ZCode에서 Todoist MCP는 필요한 날만 켜라.** config의 서버 `enabled: false` 토글로 세션당 ~13k 토큰(전체의 약 1/3)을 절약할 수 있다. Codex의 지연 로딩 효과를 수동으로 재현하는 방법.
2. **더 근본적인 우회로는 smart MCP 프록시다 (§7).** mcpproxy-go를 두면 ZCode 수정 없이 Codex식 지연 로딩에 가까운 효과(툴 정의 토큰 85%+ 절감 사례)를 얻는다.
3. **ChatGPT 웹 앱 설치에 대한 우려는 불필요.** Codex CLI 기본 컨텍스트에는 앱 툴 스키마가 적재되지 않는다. 단, `list_mcp_resources`는 앱 카탈로그 전체를 덤프하므로 주의(§4.4).
4. **Codex에서 리소스 목록 조회(`list_mcp_resources`)는 주의.** 한 번 호출하면 48k자(앱/커넥터 카탈로그)가 들어온다. "lazy = 공짜"는 아니며, 쓰는 순간의 폭탄은 유효하다.
5. **Codex config에는 firecrawl이 여전히 활성화**되어 있다(`mcp_servers.firecrawl`, OAuth). ZCode에서는 꺼둔 것과 달리 Codex에서는 살아 있으므로, 필요 없다면 정리 대상.
6. 한계: ZCode의 MCP 내부 비중은 툴 정의 크기 기반 추정(±15%)이며, Codex의 빌트인 툴 스키마 크기는 롤아웃에 기록되지 않아 추정치다. `list_mcp_resources` 카탈로그 내용은 측정 시점의 앱 구성에 따라 달라질 수 있다.

## 관련 노트

- [[LLMS/mcp-tool-search-context|MCP 툴 정의와 컨텍스트 윈도우]] — 툴 스키마 비용과 Tool Search의 개념 정리
- [[LLMS/zcode-mcp-lazy-proxy-setup|ZCode용 mcp-lazy 설정 가이드]] — 이 분석에서 채택한 프록시의 실제 구성 절차
- [[LLMS/zcode-chat-history-storage-and-deletion|ZCode 대화 히스토리]] — 측정에 사용한 `db.sqlite`와 `model_usage` 저장 구조
