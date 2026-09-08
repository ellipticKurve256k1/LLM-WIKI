---
title: API Base URLs and Chat Endpoints
tags:
  - llm
  - api
aliases:
  - base url
  - chat completions
  - openai-compatible api
  - v1/messages
created_date: 2026-08-31
---

# API Base URLs and Chat Endpoints

## Base URL

A **Base URL** is the root address of an API.

Example:

```text
https://openrouter.ai/api/v1
```

SDKs and API clients usually append a specific endpoint path to this Base URL.

For example:

```text
Base URL:
https://openrouter.ai/api/v1

Endpoint:
 /chat/completions

Final request URL:
https://openrouter.ai/api/v1/chat/completions
```

If an application asks specifically for a **Base URL**, it usually expects only the root URL, not the full `/chat/completions` path.

---

## OpenAI-Compatible API

The typical OpenAI-style chat endpoint is:

```text
/chat/completions
```

Example:

```text
POST https://openrouter.ai/api/v1/chat/completions
```

This format is commonly called the **OpenAI-compatible API format**.

It is not limited to OpenAI models. Many platforms and inference servers support this API convention, including OpenRouter and other OpenAI-compatible providers.

For example, a Claude model accessed through OpenRouter can still be called through:

```text
/chat/completions
```

The model may be Anthropic Claude, while the API interface itself follows the OpenAI-compatible format.

---

## Anthropic API

Anthropic uses a different native API format.

The standard Messages endpoint is:

```text
/v1/messages
```

Example:

```text
POST https://api.anthropic.com/v1/messages
```

This API uses Anthropic's own request structure, headers, parameters, and response format.

---

## `/v1/messages` vs `/chat/completions`

| Endpoint            | API Convention        | Typical Use                                        |
| ------------------- | --------------------- | -------------------------------------------------- |
| `/v1/messages`      | Anthropic API         | Native Claude API requests                         |
| `/chat/completions` | OpenAI-compatible API | OpenAI and compatible providers such as OpenRouter |

These endpoints should be understood as **API protocols or interface conventions**, rather than endpoints exclusively tied to a specific model vendor.

A Claude model can therefore be accessed through either format depending on the provider:

```text
Anthropic directly
→ /v1/messages

Claude through OpenRouter
→ /api/v1/chat/completions
```

The important distinction is:

**Model provider and API format are separate concepts.**

A model can be served through an API format originally popularized by another provider.

---

## 관련 노트

- [[LLMS/codex-openrouter-alias|Codex CLI OpenRouter 모델 alias]] — OpenRouter를 API 라우팅으로 사용하는 실제 사례
- [[LLMS/llm-local-tool|LLM Local Tool (Open WebUI)]] — OpenAI-compatible API로 로컬 엔진/원격 API에 연결하는 UI 레이어
- [[LLMS/local-llm-engine|Local LLM Engine]] — OpenAI-compatible 인터페이스를 제공하는 로컬 추론 서버들
