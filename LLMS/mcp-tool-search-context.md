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
created_date: 2026-08-27
---

# MCP 툴 정의와 컨텍스트 윈도우 (Tool Search)

MCP 서버의 툴 정의(`name` + `description` + `inputSchema`)가 LLM 컨텍스트에 어떻게 적재되는지, 그리고 이를 최적화하는 **Tool Search / progressive discovery** 메커니즘을 정리한 노트. "MCP 툴 정의가 session마다 매번 컨텍스트에 적재되나?"라는 질문에 대한 답이 핵심.

## 결론: "매턴(turn) 재로딩"이지 "session당 한 번"이 아니다

- 클라이언트는 연결 시 `tools/list`로 툴 목록을 가져온다.
- 툴 서치가 꺼진 상태(클래식 MCP)에서는 각 툴의 `name`/`description`/`inputSchema`가 **매 요청·매 턴마다** 대화가 시작되기 전에 시스템 접두사로 컨텍스트에 들어간다.
- 즉 session은 history를 유지하지만, **툴 정의 자체는 turn 단위로 재적재**된다. 사용자가 아무것도 안 쳐도 턴마다 정의가 토큰을 차지하는 고정 비용.

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

## 관련 노트

- [[LLMS/Subagents-Format/agents-guide|.agents, Skills, and MCP]] — MCP 서버 설정·관리(`codex mcp add`, `config.toml`) 참조
- [[LLMS/local-llm-engine|Local LLM engine]] — 로컬 LLM/컨텍스트 관련 참조
- [[LLMS/codex-openrouter-alias|Codex CLI OpenRouter alias]] — agent 툴 구성 참조

## References

- [Claude Code docs: MCP](https://code.claude.com/docs/en/mcp) — Tool Search 기본 활성화, 10% 임계값, 77K→8.7K
- [Anthropic Engineering: Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp) — 툴 정의 컨텍스트 과부하, 150K→2K
- [MCP spec: Client best practices](https://modelcontextprotocol.io/docs/2026-07-28/develop/clients/client-best-practices) — progressive discovery, 임계값, 레이어드 로딩
- [MCP: Lazy Tool Hydration RFC (#1978)](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/1978) — 106툴 ~54.6K 토큰 실측
