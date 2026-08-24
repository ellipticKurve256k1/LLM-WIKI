---
title: local-llm-engine
tags:
  - llm
  - engine
type: reference
priority: 2
finished: true
created_date: 2026-05-03
---

# Local LLM Engine

## Abstract

Brief overview of local LLM engines like llama.cpp, vLLM, and Ollama.

## Why Local?

- **Privacy**: Data never leaves your machine
- **Cost**: No API fees after initial setup
- **Customization**: Fine-tune, system prompts, embeddings
- **Offline capability**

## Popular Engines

### llama.cpp

- **What**: C++ implementation of LLaMA inference, designed for GGUF format
- **Key Feature**: Pure CPU inference, no GPU required; supports massive quantization
- **Strength**: First-party GGUF support, cross-platform, active dev community
- **Use Case**: Beginners, CPU-only systems, quantize models

### vLLM

- **What**: High-performance LLM inference engine from UC Berkeley
- **Key Feature**: PagedAttention, continuous batching, tensor parallelism
- **Strength**: Fast throughput, KV cache optimization, multiple format support
- **Use Case**: Production, high-volume inference, GPU servers

### Ollama

- **What**: User-friendly wrapper around llama.cpp/vLLM
- **Key Feature**: One-command setup, model library, open-weight model collection
- **Strength**: Easy UX, cross-platform installer, active ecosystem
- **Use Case**: Quick prototyping, non-technical users

### Text Generation Inference (TGI)

- **What**: Hugging Face's Rust-based inference engine
- **Key Feature**: FlashAttention 2, continuous batching, custom kernels
- **Strength**: HF integration, optimum hardware support
- **Use Case**: HF ecosystem users, production at scale

## GGUF Format

GGUF (GPT-Generated Unified Format) is the binary file format designed by llama.cpp for storing quantized language models.

### File Structure

| Section | Contents |
|---------|----------|
| **Header** | Magic "GGUF", version, tensor count, KV count |
| **Metadata KV** | Model config, hyperparameters, tokenizer info |
| **Tensor Descriptors** | Name, shape, type, offset for each weight |
| **Tensor Data** | Actual weights (aligned to 32 bytes) |

### Header Format

```
4 bytes  - Magic: 0x46554747 ("GGUF")
4 bytes  - Version: 3 (current)
8 bytes  - Tensor count (uint64)
8 bytes  - KV pair count (uint64)
```

### Supported Quantization Types

| Type | Bits | Description |
|------|-----|-------------|
| **F32** | 32 | Full float32 |
| **F16** | 16 | Half precision |
| **BF16** | 16 | Brain float16 |
| **Q8_K** | 8 | 8-bit quantization |
| **Q6_K** | 6 | 6-bit (block size 256) |
| **Q5_K** | 5 | 5-bit (block size 256) |
| **Q4_K** | 4 | 4-bit (block size 256) |
| **Q3_K** | 3 | 3-bit (block size 256) |
| **Q2_K** | 2 | 2-bit (block size 256) |

### Common Metadata Keys

| Key | Example Value | Description |
|-----|-------------|--------------|
| `general.architecture` | "llama" | Model architecture |
| `general.name` | "Llama 3 8B" | Model name |
| `general.file_type` | 15 (Q4_K_M) | Quantization level |
| `llama.context_length` | 8192 | Max context window |
| `llama.embedding_length` | 4096 | Hidden dimension |
| `llama.block_count` | 32 | Number of layers |
| `llama.attention.head_count` | 32 | Attention heads |
| `tokenizer.ggml.model` | "llama" | Tokenizer type |

### Working with GGUF

```bash
# Convert to GGUF (FP16)
python convert.py /path/to/model --outfile model-f16.gguf

# Quantize
./quantize model-f16.gguf model-q4.gguf Q4_K_M

# Inspect metadata
./quantize --show model.gguf
```

## Comparison

| Engine | Best For | GPU | Quantization | HF Compatible |
|--------|----------|-----|--------------|---------------|
| **llama.cpp** | CPU, beginners | Optional | Native | Via ctransformers |
| **vLLM** | Production throughput | Required | Various | Yes |
| **Ollama** | Quick setup | Optional | Native | Partial |
| **TGI** | HF ecosystem | Required | Various | Yes |

## Quick Start Commands

```bash
# Ollama (easiest)
ollama run llama3

# llama.cpp
./main -m model.gguf -n 256

# vLLM
vllm serve meta-llama/Llama-3-8B

# TGI
text-generation-launcher --model-id meta-llama/Llama-3-8B
```
