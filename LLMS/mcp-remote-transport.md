---
title: MCP 원격 전송 (Transport) — HTTP+SSE에서 Streamable HTTP로
tags:
  - llm
  - mcp
  - transport
  - sse
  - http
aliases:
  - mcp transport
  - streamable http
  - mcp http sse
  - mcp 원격 전송
  - mcp 전송 계층
created_date: 2026-08-31
---

# MCP 원격 전송 (Transport) — HTTP+SSE에서 Streamable HTTP로

MCP의 원격 전송(transport)이 어떻게 진화했는지 정리한 노트. 로컬 STDIO 전송이 아니라 원격 서버와의 HTTP 기반 통신 방식의 역사와 설계 변화가 주제.

## 역사: 세 단계의 변화

### 1. HTTP+SSE (스펙 2024-11-05) — deprecated

초창기 원격 전송. **두 개의 연결**을 상시 유지하는 방식:

- `/sse` 엔드포인트로 SSE 연결을 열어두고 서버→클라이언트 메시지를 받음
- `/message` 엔드포인트로 POST를 보내 클라이언트→서버 요청을 전달

### 2. Streamable HTTP (스펙 2025-03-26) — HTTP+SSE 대체

Streamable HTTP는 SSE를 없앤 게 아니라, SSE를 **독립 전송 계층에서 응답 스트리밍 메커니즘으로 격하**시킨 것. 클라이언트의 POST 하나에 대해 서버는:

- `Content-Type: application/json` — 단일 JSON 응답
- `Content-Type: text/event-stream` — SSE 스트림 응답

중 선택해 답할 수 있다. 즉 SSE는 이제 "필요할 때 서버가 스트리밍 응답에 쓸 수 있는 포맷"이지, 연결을 상시 유지하는 전송 방식 자체가 아니다. SSE 기술 자체(EventSource wire format)는 그대로 남아 있고 쓰임새와 역할이 달라진 것. "Streamable HTTP"라는 이름 자체가 "HTTP인데 응답을 스트리밍(SSE)으로 흘려줄 수도 있다"는 뜻.

### 3. 단순화 (스펙 2026-07-28)

- GET 기반 서버 푸시 스트림(빈 GET으로 서버→클라이언트 알림을 받던 채널)과 프로토콜 레벨 세션이 제거됨.
- 모든 통신은 POST 하나의 MCP 엔드포인트(예: `/mcp`)로 간다.
- 서버→클라이언트 요청(sampling, elicitation 등)은 별도 스트림이 아니라 **응답 안에 임베드**된다.

## 왜 바꿨나

HTTP+SSE의 문제:

- **상시 장기 연결**이 필요해 서버 가용성 부담이 큼
- **무상태(stateless) 서버가 불가능** — 연결 상태를 서버가 유지해야 함
- 연결 끊김 시 **재개(resumability)가 안 됨**

Streamable HTTP의 해법: 단순 요청은 그냥 HTTP request/response로 처리하고, 긴 작업만 SSE 스트림으로 승격하는 유연한 동작. 서버 구현 부담이 줄고 수평 확장이 쉬워진다.

## 관련 노트

- [[LLMS/mcp-tool-search-context|MCP 툴 정의와 컨텍스트 윈도우]] — 같은 MCP 주제의 툴 정의·컨텍스트 비용 관점
- [[LLMS/Subagents-Format/agents-guide|.agents, Skills, and MCP]] — MCP 서버 설정·관리(`codex mcp add`, `config.toml`)

## References

- [MCP specification revision 2024-11-05](https://modelcontextprotocol.io/specification/2024-11-05) — HTTP+SSE transport 원 정의
- [MCP specification: Streamable HTTP (2025-03-26)](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports) — HTTP+SSE deprecated 처리
- [MCP specification (2026-07-28)](https://modelcontextprotocol.io/specification/2026-07-28) — GET 스트림·프로토콜 레벨 세션 제거
