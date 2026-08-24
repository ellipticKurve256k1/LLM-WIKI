---
title: LLM Benchmarks
tags: [llm, reference, note, benchmark]
type: reference
priority: 2
finished: true
created_date: 2026-05-03
---

# LLM Benchmarks 

## Abstract

Short note of what each benchmark measures

### Measurement

#### MMLU-Pro (Massive Multitask Language Understanding)

- What: Multiple-choice questions across 57 subjects (math, science, history, law, etc.)
- How: 5-shot prompting (model sees 5 examples, then answers)
- Higher % = better — measures broad knowledge and reasoning

#### SWE-bench (Software Engineering)
- What: Real-world GitHub issues requiring code fixes
- How: Model must generate a patch to solve the bug
- Higher % = better — measures coding + problem-solving ability

#### HumanEval
- What: Python coding problems with test cases
- How: Pass@1 — model generates code, must pass unit tests on first try
- Higher % = better — measures code generation capability

#### MATH (Competition Mathematics)

- What: Competition-level math problems (AIME, MATH dataset) across 5 difficulty levels
- How: Solve math problems requiring multi-step reasoning without tools
- Higher % = better — measures mathematical reasoning and problem-solving

#### GPQA Diamond (Graduate-Level Science)

- What: Multiple-choice questions at graduate-level physics, chemistry, biology
- How: Answer questions requiring deep domain expertise
- Higher % = better — measures advanced scientific knowledge and reasoning

#### Throughput (tokens/second)

- What: How fast the model generates output
- How: Measured on identical hardware (same GPU)
- Higher = faster inference — measures deployment efficiency

--- 

### Real Benchmarks

#### Proprietary Frontier Models

| Model | MMLU | HumanEval | SWE-bench | MATH | GPQA |
|-------|------|-----------|-----------|------|------|
| GPT-5.4 (OpenAI) | 91.8% | 94.1% | 62.3% | 90.2% | 72.8% |
| Claude Opus 4.6 (Anthropic) | 92.1% | 92.4% | 58.4% | 91.5% | 74.2% |
| Gemini 3.1 Ultra (Google) | 90.4% | 89.3% | 54.2% | 88.7% | 71.5% |
| o3-pro (OpenAI) | 88.2% | 91.2% | 60.1% | 96.7% | 87.4% |

#### Open-Source / Open-Weight Models

| Model | MMLU | HumanEval | SWE-bench | MATH | GPQA |
|-------|------|-----------|-----------|------|------|
| DeepSeek V4 | 87.2% | 88.7% | 48.3% | 84.1% | 65.2% |
| DeepSeek R1 | 89% | 93% | ~50% | 91% | 72% |
| Llama 4 Maverick | 84.7% | 82.1% | 42.6% | 78.4% | 58.3% |
| Qwen3 72B | 83.6% | 85.4% | 44.7% | 80.7% | 61.2% |
| Mistral Large 2 | 78.2% | 76.8% | 34.2% | 68.4% | 46.8% |

#### Key Takeaways

- Proprietary models dominate SWE-bench (coding agents) and GPQA (graduate-level science)
- Open-source has closed the gap on MMLU (~5-8 point gap) but still trails on agentic coding tasks
- o3-pro leads on MATH (96.7%) and GPQA (87.4%) — specialized for deep reasoning
- Claude leads general knowledge (MMLU), GPT leads coding (HumanEval/SWE), Gemini leads multimodal (MMMU)

Interpreting the Numbers
- Accuracy benchmarks (MMLU, SWE, HumanEval): Higher % = more capable
- Throughput: Higher = cheaper to run, lower latency
- Trade-off: Faster models often sacrifice some accuracy (hence Nemotron trails Qwen slightly on MMLU but wins on speed