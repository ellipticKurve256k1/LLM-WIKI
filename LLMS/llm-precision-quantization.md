---
title: LLM Precision & Quantization
tags:
  - llm
  - quantization
  - precision
  - inference
aliases:
  - precision
  - fp8
  - bf16
  - mixed precision
  - llm precision
created_date: 2026-08-29
---

# LLM Precision & Quantization

Reference for numeric precision formats used in LLM serving, how they relate to [[LLMS/quantization-note|quantization]], and what provider precision labels (e.g. OpenRouter `FP8`) actually mean.

## LLM Precision & Quantization

* **BF16 / FP16**

  * 16-bit floating point
  * Common for training and high-quality inference
  * Generally the safest for preserving model quality

* **FP8**

  * 8-bit floating point
  * Common in modern server inference
  * Reduces VRAM and compute cost with relatively small quality loss
  * FP8 can also be considered a form of quantization

* **INT8 / INT4**

  * Integer quantization — see [[LLMS/quantization-note|Quantization Note]] for scale/zero-point mechanics
  * Lower bit-width reduces memory and inference cost
  * More aggressive quantization can cause larger quality degradation, especially for reasoning, coding, and tool use

## Mixed Precision

A model does not necessarily use the same precision everywhere.

Example:

* Most weights: FP8
* Sensitive layers: BF16
* Activations: FP8 or BF16
* KV cache: FP8
* Accumulation: FP16 / FP32

So when OpenRouter shows `FP8`, it may actually mean an **FP8-centered mixed-precision deployment**.

## Comparing Providers

* Same token price does **not** imply the same precision
* Pricing also depends on GPU type, batching efficiency, infrastructure cost, utilization, and provider strategy
* If precision is not disclosed, treat it as `unknown`
* Unusually low prices combined with very high throughput may indicate more aggressive optimization or quantization

For quality-sensitive coding and agent workloads, a rough preference order is:

**BF16 / FP16 > FP8 > unknown > INT8 > INT4**

> [!tip] Checking actual provider costs
> Use [[LLMS/openrouter-logs-usage|OpenRouter logs]] to compare real input/output token costs per provider — price differences there reflect precision, batching, and infrastructure trade-offs combined.

## Related Notes

- [[LLMS/quantization-note|Quantization Note]] — INT8/INT4 quantization mechanics, scale & zero-point, PTQ/QAT
- [[LLMS/local-llm-engine|Local LLM Engine]] — GGUF quantization types (Q8_K ~ Q3_K) and engine support
- [[LLMS/token|Token]] — token cost basics that provider pricing builds on
