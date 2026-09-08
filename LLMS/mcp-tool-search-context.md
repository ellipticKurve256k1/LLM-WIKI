---
title: MCP 툴 정의와 컨텍스트 윈도우 (Tool Search)
tags:
  - llm
  - mcp
  - context-window
  - token
aliases:
  - mcp tool search
  - progressive discovery
  - mcp 컨텍스트
  - mcp tool definition token
  - "/context"
  - context usage breakdown
created_date: 2026-08-27
---

# MCP 툴 정의와 컨텍스트 윈도우 (Tool Search)

MCP 서버의 툴 정의(`name` + `description` + `inputSchema`)가 LLM 컨텍스트에 어떻게 적재되는지, 그리고 이를 최적화하는 **Tool Search / progressive discovery** 메커니즘을 정리한 노트. "MCP 툴 정의가 session마다 매번 컨텍스트에 적재되나?"라는 질문에 대한 답이 핵심.

## 결론: "매턴(turn) 재로딩"이지 "session당 한 번"이 아니다

- 클라이언트는 연결 시 `tools/list`로 툴 목록을 가져온다.
- 툴 서치가 꺼진 상태(클래식 MCP)에서는 각 툴의 `name`/`description`/`inputSchema`가 **매 요청·매 턴마다** 대화가 시작되기 전에 시스템 접두사로 컨텍스트에 들어간다.
- 즉 session은 history를 유지하지만, **툴 정의 자체는 turn 단위로 재적재**된다. 사용자가 아무것도 안 쳐도 턴마다 정의가 토큰을 차지하는 고정 비용.
- 다만 "재적재"의 의미를 구분해야 한다:
  - **컨텍스트 윈도우 점유 — 누적 아님(상수).** 매 요청의 컨텍스트는 `[지침] + [툴 스키마] + [대화 히스토리]` 구조인데, 툴 정의는 항상 같은 자리에 **딱 1번**만 실린다. 100턴째여도 스키마가 100개 쌓이는 게 아니라 여전히 한 세트. 턴이 늘어나서 커지는 건 히스토리 부분뿐이다.
  - **과금 — 턴 수에 비례해 누적.** API는 stateless라 매 요청마다 클라이언트가 전체 prefix(툴 정의 포함)를 새로 보낸다. 10턴 대화 = 툴 스키마를 10번 과금. "session당 한 번"이 아니라 "매 요청마다 다시 내는 구독료"에 가깝다.

## Prompt caching: 같은 prefix를 매번 정가로 안 내는 장치

매 요청마다 앞부분(시스템 프롬프트 + 툴 스키마 + 초기 대화)은 **완전히 동일**하다. 이를 이용해 프로바이더는 동일한 prefix를 **캐시**해두고, 다음 요청에서 그 부분을 새로 계산하지 않고 재사용한다. 쉽게 비유하면:

- 대화 = 턴마다 페이지가 늘어나는 책. 툴 정의는 책 맨 앞의 목차 페이지.
- 매 요청(턴)마다 책 전체를 **처음부터 다시 읽어** 모델에 넣어야 한다(stateless).
- 그런데 목차 앞부분은 매번 똑같으니, 서버가 "이 페이지들은 어제 것과 같네" 하고 **복사본을 꺼내 쓴다** — 다시 읽을 필요가 없다.
- 복사분(캐시 히트)은 정가의 **~10%** 수준으로만 과금. 새로 추가된 뒷부분(새 턴의 메시지)만 정가.

캐시가 깨지는 경우:

- 시스템 프롬프트·툴 목록·히스토리 앞부분이 **하나라도 바뀌면** 그 지점 이후 전부 캐시 미스(정가). 그래서 MCP 서버를 켜고/끄는 것만으로도 캐시가 무효화된다.
- 캐시는 보통 **수 분(TTL)** 후 만료 — 턴 사이가 길어지면 복사본이 사라져 다시 정가로 읽는다.

즉 툴 정의의 실제 비용은 "17.9k × 턴 수 전액"이 아니라 **"첫 요청 정가 + 이후 턴은 대부분 ~10% 할인가"**로 수렴한다. 그래도 0원이 아니므로 툴 정의가 턴마다 토큰을 먹는 구조 자체는 그대로다 — Tool Search(code mode)는 이 고정비 자체를 줄이는 접근이고, 캐싱은 그 비용을 할인하는 접근으로 서로 보완적이다.

### 캐시는 누가, 뭘 저장하나

- **저장 주체는 프로바이더 측 서빙 인프라**다(모델을 서빙하는 GPU 클러스터). 유저나 클라이언트(Claude Code)는 아무것도 저장하지 않는다.
- 저장하는 것은 텍스트가 아니라 **KV 캐시** — prefill 단계에서 계산한 토큰별 attention 중간 텐서(K, V). 원래는 요청 끝나면 버려지지만, 캐싱 시 GPU/호스트 메모리에 잠깐 보관한다. 재계산(연산) 없이 저장분을 **읽기만** 하니 할인이 되는 이유다.
- "같은 목차" 판정은 의미 비교가 아니라 **정확한 접두사 매칭 + 해시 룩업**:
  1. 새 요청의 토큰을 블록 단위로 해싱해 "이 prefix, 저장된 게 있나?" 조회
  2. 매칭 → 저장된 KV 텐서 그대로 재사용, 새 토큰(뒷부분)만 새로 계산
  3. 앞부분이 **1토큰이라도 다르면** 그 지점 이후 전부 무효(캐시 미스)
- 흐름: 첫 요청에서 KV 계산 후 prefix 부분을 캐시에 보관(TTL 수 분) → 다음 요청에서 클라이언트는 stateless라 전체 prefix를 다시 보내긴 한다 → 서빙 계층이 해시 대조로 "아까 그거네" 확인 → 계산 생략 + 할인 과금.
- 정리하면 "재적재"는 **네트워크 전송·과금 관점**의 진실이고, "캐시"는 **서버가 계산을 생략해주는 장치**다. 클라이언트는 매번 툴 스키마를 다시 보내지만, 서버가 받아보니 같아서 계산을 생략하는 것.

## 컨텍스트 직렬화 구조: "목차 먼저, 대화는 뒤에 append"

매 요청의 프롬프트는 서빙 쪽에서 이렇게 직렬화된다:

```
[시스템 프롬프트] → [툴/MCP 정의들] → [대화 히스토리: 턴1 유저, 어시스턴트, 턴2 유저, ...] → [이번 턴 유저 메시지]
 └────── 매번 동일 (캐시 대상) ──────┘   └────────── 턴마다 뒤에 계속 붙음(append) ──────────┘
```

- 툴 스키마·MCP 정의는 요청 시 messages 배열과 별도 필드(`tools`)로 전송되지만, 서빙 쪽에서 **프롬프트 앞부분에 직렬화**한다. 시스템 프롬프트와 툴 정의의 세부 순서는 provider마다 다를 수 있으나, 둘 다 대화보다 앞이라는 건 공통.
- 이 구조가 캐시가 작동하는 이유다 — 앞부분(목차)이 매 요청 바이트 단위로 동일하니 해시 매칭이 되고, 대화는 뒤에 append만 되니 prefix가 안 깨진다.
- 반대로 목차 중간(툴 목록)이 바뀌면 그 지점부터 대화 전체가 캐시 미스가 된다. **MCP 서버를 켜고/끄면 캐시가 전부 무효화되는 정체가 이 구조**다.

## 클래식 MCP: 모든 정의를 매번 적재

- 툴 정의를 전부 컨텍스트에 넣고 `tools/call` 메타툴로 호출.
- 툴이 많을수록 비용 급증: 예시 실측은 MySQL MCP 106개 툴 → 초기화 때마다 ~54,600 토큰(~207KB). 여러 서버를 붙이면 200K 컨텍스트의 ~70%가 툴 정의에 소모될 수 있다.
- 작은 세트(1~2개 서버, 몇 개 툴)라면 충분히 합리적.

## Tool Search / progressive discovery: 필요할 때만 적재

- 툴 정의를 컨텍스트에 미리 넣지 않고, 가벼운 `search_tools`/`ToolSearch` 메타툴만 제공한 뒤 **모델이 그 턴에 필요한 툴만 골라 로드**하는 레이어드 방식.
  1. **Catalog(List)** — 툴 이름/요약 정도의 가벼운 목록
  2. **Inspect** — 후보 툴 하나만 전체 스키마 로드
  3. **Execute** — 해당 툴 호출
- Anthropic 보고: 77K → ~8.7K 토큰 (약 95% 감소).
- **Claude Code에서는 기본 활성화(GA)**. 툴 정의가 컨텍스트의 약 10%를 초과하면 자동으로 툴 서치 모드로 전환.
- MCP 공식 클라이언트 best practices도 1%~5% 임계값을 넘으면 progressive discovery로 전환하라고 권장.

## 실제 저비용 질문에 대한 요약

- 툴 서치 **꺼짐**: 모든 툴 정의가 매 턴 컨텍스트에 로드 (고정 재료비).
- 툴 서치 **켜짐**: 정의가 기본적으로 컨텍스트 밖에 있고, 필요할 때만 로드.
- 더 극단적으로, **Code Execution with MCP(code mode)** 로 툴을 코드 API처럼 노출하면 중간 결과까지 샌드박스에 두고 최종 요약만 모델에 돌려줘서 예시상 150K → 2K (~98.7%).

## 실측: Claude Code `/context`로 세션 컨텍스트 분해 (2026-08-28)

Claude Code 내장 명령 **`/context`**는 현재 세션의 토큰 사용량을 항목별 실측치로 분해해 보여준다. system prompt / tools / MCP / skills / messages가 각각 몇 토큰인지 즉시 확인 가능. 실측 세션: `z-ai/glm-5.3-flash` (OpenRouter 경유), 윈도우 1M, 총 41.8k (4%).

| 항목 | 토큰 | 내용 |
|---|---|---|
| System prompt | 1.7k (0.2%) | 도구 스키마 **제외한** 지침 텍스트만 |
| System tools | 17.9k (1.8%) | 내장 도구 JSON 스키마 — 최대 고정 비용 |
| MCP tools | 77 (0.0%) | exa 2개뿐 (firecrawl ~20개 툴 서버는 `/mcp`로 꺼둠) |
| Custom agents | 569 (0.1%) | `.claude/agents/` 2개 요약부 (본문은 호출 시 로드) |
| Skills | 2k (0.2%) | 14개 스킬 설명줄만 (본문은 실행 시 Messages에 주입) |
| Messages | 19.9k (2.0%) | 실제 대화 누적 |
| Free space | 924.9k (92.5%) | 여유분 |
| Autocompact buffer | 33k (3.3%) | 예약분 — 사용량 도달 시 자동 요약(autocompact) |

시사점:

- **툴 정의가 지침 텍스트의 ~10배** (17.9k vs 1.7k). 흔히 "system prompt 비용"이라 부르는 것의 대부분은 별도 `tools` 필드의 스키마다.
- MCP 서버는 `/mcp`로 끄면 툴 목록에서 즉시 빠진다 — 끈 뒤 exa 2개 = 77 토큰. (끄기 전 firecrawl ~20개 스키마는 실측 없음, 추정 ~15k급)
- 스킬 14개의 상시 비용은 설명줄 합계 ~2k뿐. SKILL.md 전문은 실행 턴에 Messages에 기록.
- 세션에서 보이는 `15,000,000 tokens left` 같은 카운터는 **OpenRouter 잔여 예산**이지 컨텍스트 윈도우가 아니다. 윈도우 크기는 `/context`로 확인.

## 관련 노트

- [[LLMS/zcode-codex-harness-context-analysis|하네스 컨텍스트 분석 — ZCode vs Codex CLI]] — ZCode와 Codex CLI의 MCP 로딩 방식·컨텍스트 비용 실측 비교
- [[LLMS/zcode-mcp-lazy-proxy-setup|ZCode용 mcp-lazy 설정 가이드]] — eager MCP 스키마 비용을 줄이는 프록시 구성 절차
- [[LLMS/mcp-remote-transport|MCP 원격 전송 (Transport)]] — HTTP+SSE → Streamable HTTP로의 전송 계층 진화
- [[LLMS/Subagents-Format/agents-guide|.agents, Skills, and MCP]] — MCP 서버 설정·관리(`codex mcp add`, `config.toml`) 참조
- [[LLMS/local-llm-engine|Local LLM engine]] — 로컬 LLM/컨텍스트 관련 참조
- [[LLMS/codex-openrouter-alias|Codex CLI OpenRouter alias]] — agent 툴 구성 참조
- [[LLMS/token|What Is a Token]] — 토큰 단위·토크나이저 기본 개념 (이 노트의 분해 대상)

## References

- [Claude Code docs: MCP](https://code.claude.com/docs/en/mcp) — Tool Search 기본 활성화, 10% 임계값, 77K→8.7K
- [Anthropic Engineering: Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) — 툴 정의 컨텍스트 과부하, 150K→2K
- [MCP spec: Client best practices](https://modelcontextprotocol.io/docs/2026-07-28/develop/clients/client-best-practices) — progressive discovery, 임계값, 레이어드 로딩
- [MCP: Lazy Tool Hydration RFC (#1978)](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/1978) — 106툴 ~54.6K 토큰 실측
