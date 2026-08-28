---
title: Codex CLI OpenRouter 모델 alias
tags:
  - llm
  - codex
  - openrouter
  - zsh
aliases:
  - ori codex alias
  - openrouter codex
  - kimi-k3 alias
  - glm-5.2 alias
  - glm-5.3-flash alias
  - claude-wiki-glm alias
  - ori-deepseek alias
  - ori-claude-deepseek alias
  - claude-wiki alias
created_date: 2026-08-26
---

# Codex CLI OpenRouter 모델 alias

`ori codex`(OpenRouter Codex CLI)에 `--model` 플래그로 다른 LLM을 지정하면, OpenRouter 라우팅으로 해당 모델을 Codex 인터페이스에서 실행할 수 있다. 자주 쓰는 모델은 `~/.zshrc`에 alias로 등록해두면 된다.

## 등록한 alias

```sh
alias ori-kimi="ori codex --model moonshotai/kimi-k3"
alias ori-glm="ori codex --model z-ai/glm-5.2"
alias ori-deepseek="ori codex --model deepseek/deepseek-v4-flash-0731"
alias ori-claude-deepseek="ori claude --model deepseek/deepseek-v4-flash-0731"
alias ori-claude-glm="ori claude --model z-ai/glm-5.3-flash"
alias ori-claude-kimi="ori claude --model moonshotai/kimi-k3"
alias claude-wiki="wiki && ori-claude-deepseek"
alias claude-wiki-glm="wiki && ori-claude-glm"
```

### Codex 인터페이스 (`ori codex`)

- `ori-kimi` — Moonshot AI의 Kimi K3 모델 실행
- `ori-glm` — Zhipu AI의 GLM-5.2 모델 실행
- `ori-deepseek` — DeepSeek의 DeepSeek V4 Flash 실행

### Claude 인터페이스 (`ori claude`)

- `ori-claude-deepseek` — 같은 DeepSeek V4 Flash를 `ori claude`로 실행
- `ori-claude-glm` — Zhipu AI의 GLM-5.3-flash를 `ori claude`로 실행
- `ori-claude-kimi` — Moonshot AI의 Kimi K3를 `ori claude`로 실행

### 복합 alias

- `claude-wiki` — `wiki`로 위키로 이동한 뒤 `ori-claude-deepseek` 실행
- `claude-wiki-glm` — `wiki`로 위키로 이동한 뒤 `ori-claude-glm` 실행

## 모델 식별자 규칙

- `--model` 값은 OpenRouter의 `provider/model-name` 형식을 따른다.
- 새 모델을 추가하려면 OpenRouter 모델 페이지에서 식별자를 확인한 뒤 같은 패턴으로 alias를 만든다.

## 적용 및 확인

```sh
source ~/.zshrc
alias | grep ori
```

## 관련 노트

- [[LLMS/Subagents-Format/agents-guide|`.agents`, Skills, and MCP]] — Codex CLI, MCP 설정, `codex` 명령어 참조
- [[bash-ssh/bash-commands|Bash commands]] — 셸 명령어 및 alias 기본 참조
- [[LLMS/openrouter-logs-usage|OpenRouter Logs로 요금·토큰·캐시 확인]] — `ori` 호출의 provider·토큰·비용·캐시 확인
