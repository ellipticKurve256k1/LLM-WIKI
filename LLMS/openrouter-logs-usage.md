---
title: OpenRouter Logs로 요금·토큰·캐시 확인
tags:
  - llm
  - openrouter
  - token
  - cost
aliases:
  - openrouter logs
  - openrouter 사용량 확인
  - openrouter provider 확인
  - input output token cost
created_date: 2026-08-27
---

# OpenRouter Logs로 요금·토큰·캐시 확인

OpenRouter에서 내가 쓴 요청의 세부 정보를 확인하는 방법. 어떤 provider(모델 실제 실행처)가 선택됐는지, input/output 토큰당 비용과 합계, 그리고 cached 여부 등을 알고 싶다면 **Logs 메뉴**에 들어가면 된다.

## Logs 메뉴에서 볼 수 있는 것

OpenRouter 대시보드의 **Logs**(활동 로그)는 각 요청에 대해 다음을 보여준다:

- **Model** — 내가 요청한 모델 식별자 (예: `provider/model-name`)
- **Provider** — 요청이 실제로 라우팅된 provider(모델이 실행된 곳). 같은 모델이라도 변동/우선순위에 따라 달라질 수 있음
- **Input tokens / Output tokens** — 입력·출력 토큰 수
- **Input cost / Output cost** — 각각의 토큰당 비용과 그 합계
- **Cached 여부** — 프롬프트 캐싱(prompt caching)이 적용됐는지 여부

각 항목을 필터링하거나 상세 보기를 열면 요청별 정확한 수치를 확인할 수 있다.

> [!tip] 캐시와 비용 확인 팁
> - **Cached** 표시: 프롬프트 캐시가 히트되면 input 비용이 크게 줄어든다. 캐시가 적용되는 조건(LM 공급사별 캐시 활성화)이 Provider/LM 설정에서 켜져 있어야 한다.
> - **Provider 비교**: Logs로 어떤 provider에 실제로 청구됐는지 보면, `--model provider/model` 방식에서 의도한 provider와 실제 라우팅 대상이 다른 경우를 잡아낼 수 있다.

## 토큰 단가(추론 비용) 해석

Logs에 보이는 `Input cost`/`Output cost`는 모델 실행(추론)의 **토큰당 단가**로, 요청별 청구는 단가에 실제 토큰 수를 곱해 계산된다:

```
총 비용 = (input tokens × input $/M) + (output tokens × output $/M)
```

- **input(입력) 토큰** — 프롬프트·컨텍스트 처리 비용
- **output(출력) 토큰** — 답변 생성 비용. 보통 input보다 크게 비싸다.

> [!note] 캐시 비용도 별도 항목
> 총비용에는 캐시 관련 비용(cached input 할인 단가, cache write 수수료)이 추가로 붙을 수 있다. 즉 청구 = 토큰 단가 + 캐시 단가 구조다.

## 코드스/CLI 요청도 Logs에 잡히나?

OpenRouter API를 거친 요청(예: `codex-openrouter-alias`의 `ori` CLI 호출)도 모두 OpenRouter 대시보드 **Logs**에 기록된다. 터미널에서 모델/라우팅을 바꿔가며 쓴 요청의 비용을 한곳에서 추적하는 데 유용하다.

## 관련 노트

- [[LLMS/codex-openrouter-alias|Codex CLI OpenRouter alias]] — `ori` CLI로 OpenRouter 모델을 alias로 실행하는 방법
- [[LLMS/mcp-tool-search-context|MCP 툴 정의와 컨텍스트 윈도우]] — 토큰/컨텍스트 비용 관련 참조
