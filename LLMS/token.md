---
title: What Is a Token (LLM)
tags:
  - llm
  - token
  - context-window
aliases:
  - token
  - tokens
  - tokenizer
created_date: 2026-08-27
---

# What Is a Token (LLM)

A **token** is the basic unit of text that an LLM reads and generates. Models do not process text character-by-character; they split it into tokens and work with that sequence.

## Key Idea

- A token is the smallest chunk an LLM tokenizer produces.
- As a rough rule of thumb: **~1 token ≈ 4 characters of English ≈ 1–1.5 characters of Korean.**
- Whitespace, punctuation, and symbols each count toward the token total.

## Examples

```
"Hello, world!"  →  ["Hello", ",", " world", "!"]  → ~4 tokens
"안녕하세요"        →  → ~3–4 tokens (Korean tends to use more tokens per unit of text)
```

## Why Tokens Matter

1. **Cost** — API pricing is billed per token (input + output).
2. **Context limit** — a model can only hold a fixed number of tokens at once (e.g. a 200K context window).
3. **Performance** — more tokens means slower responses and higher cost.

## Input vs Output Tokens

- **Input tokens** — the prompt plus prior conversation history.
- **Output tokens** — the response the model generates.
- **Tokenizer** — the tool that converts text to and from tokens; each model family has its own.

## Markdown Token Efficiency

Does writing in Markdown inflate token count? **Yes, but only slightly — ~5–15% overhead over plain text.** The structural symbols are cheap relative to your actual prose.

### Cost of common Markdown characters

| Syntax | Typical token cost |
|---|---|
| `-` bullet marker | ~1 token |
| `#`, `##`, `###` heading | ~1 token |
| `**bold**` | `**` ≈ 1 token each (2 tokens overhead per phrase) |
| Backticks / code fences | ~1–2 tokens per fence |
| Language identifier (e.g. `python`) | 1–2 tokens |

For a ~1,000-word document, Markdown formatting adds roughly **50–150 tokens** of overhead — about **5–15%** over plain text. Measured density: plain text ≈ 3.7 chars/token vs Markdown ≈ 3.6 — only a few percent difference.

### Markdown is the most token-efficient structured format

| Format | Cost vs Markdown | Why |
|---|---|---|
| Plain text | ~5–15% cheaper | Drops `#`, `-`, `\|` but loses all structure |
| HTML | **3–5×** more | Tags, CSS classes, scripts, tracking — up to 5.3× on real pages |
| JSON | ~1.4–1.5× more | Quotes, braces, repeated keys can't merge; 15–20% worse if pretty-printed |
| LaTeX | 2–3× more | Verbose commands, preambles |

### What actually eats tokens (not Markdown symbols)

- **Code blocks** — indentation and brackets fragment into small tokens, ~2–3× a same-sized prose chunk.
- **URLs / long IDs** — a UUID or long URL splits into many tokens.
- **Emoji / non-Latin script** — an emoji can be 2+ tokens; Korean often uses several tokens per character.

### Takeaway

Markdown is the **densest format that preserves structure**: it beats HTML (3–5×), JSON (~1.4–1.5×), and LaTeX (2–3×), while plain text costs only a few percent less and discards every structural signal (headings) models use to navigate, chunk for RAG, and cite. The `*` and `-` characters do cost ~1 token each, but they are rare in normal writing, so they are a rounding error next to code fences and URLs.

> **Note:** tokenization differs by tokenizer (OpenAI `cl100k_base`/`o200k_base`, Claude, Google SentencePiece), but the relative ranking above holds across all major BPE tokenizers. Tokens are not standardized across models/providers — counting with the right tokenizer for your model is the only way to get exact numbers.

## Related Notes

- [[LLMS/mcp-tool-search-context|MCP tool definitions & context window]] — how tool definitions consume token budget per turn, and Tool Search savings
- [[LLMS/openrouter-logs-usage|OpenRouter logs usage]] — concrete input/output token cost calculation
- [[LLMS/local-llm-engine|Local LLM engine]] — tokenizer metadata and model config
- [[LLMS/quantization-note|Quantization]] — model size and memory trade-offs

## References

- [All Markdown Tools: LLM Markdown formatting guide](https://allmarkdowntools.com/blog/llm-markdown-formatting-guide) — per-character token costs, 5–15% overhead
- [BulkMD: Markdown vs JSON vs Text for LLM context](https://bulkmd.app/blog/markdown-vs-json-vs-text-llm-context) — per-format token density table
- [MD Is Better: HTML vs Markdown token counts (20 real pages)](https://mdisbetter.com/blog/html-vs-markdown-token-count-comparison) — ~5.3× average reduction
- [Mechanical Advantage: Markdown is the lingua franca of AI](https://mechanicaladvantage.ai/articles/markdown-lingua-franca-of-ai) — format overhead comparison, RAG accuracy
- [IloveMD: Markdown token counter](https://iluvmd.com/markdown-token-counter) — code ~2–3×, ~1,300–1,400 tokens per 1,000 words
- [OpenAI help: What are tokens and how to count them](https://help.openai.com/en/articles/4936856-what-are-tokens-and-how-to-count-them) — tokenizer taxonomy, pricing

> Sources are external references; figures are measured with OpenAI tokenizers unless noted and are indicative ranges, not exact per-model guarantees.
