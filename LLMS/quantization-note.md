---
title: quantization note
tags: [note, reference, quantization, llm]
type: reference
priority: 2
finished: true
created_date: 2026-05-03
---

# Quantization Note

![[llm-oneshot.png]]

## Abstract

Quantization is the process of reducing the precision of numerical values in a model, typically from 32-bit floating point (FP32) to lower bit representations like 8-bit (INT8). This technique significantly reduces model size and memory usage while speeding up inference, making large language models more practical for deployment on resource-constrained devices.

## Core Concepts

### What is Quantization?

Large language models store weights in high-precision(**full-precision**) formats (typically 32-bit floating point). Quantization converts these high-precision values into **integer** representations (e.g., 8-bit integers). This compression reduces:
- **Model size** (2-4x smaller)
- **Memory footprint** (less VRAM required)
- **Inference latency** (faster computation)

### Why It Matters

Running a 70B parameter model in FP32 requires ~280GB of memory. Quantized to INT8, it needs ~70GB—making it feasible on consumer GPUs. This enables:
- Deployment on edge devices
- Reduced cloud inference costs
- Faster response times
- Greater accessibility for personal use

## Technical Mechanics

### How It Works: Scale & Zero-Point

Quantization maps float32 values to integers using two parameters:

- **Scale (S)**: A float32 value representing the "step size" of each integer
- **Zero-Point (Z)**: An integer corresponding to float32 value 0.0

**Quantization formula:**

```
q = round(x / S) + Z
```

**Dequantization (during inference):**

```
x ≈ S × (q - Z)
```

This is why quantized models are "approximate" — values are reconstructed at runtime using these stored parameters.

### Symmetric vs Asymmetric Quantization

| Type | Float Range | Integer Range | Zero-Point | Use Case |
|------|-------------|----------------|------------|----------|
| **Symmetric** | [-a, a] | [-127, 127] | Z = 0 | Weights (faster) |
| **Asymmetric** | [min, max] | [-128, 127] | Z ≠ 0 | Activations (more accurate) |

### Integer Bit Depths

| Format | Range | Notes |
|--------|-------|-------|
| **INT8** | -128 to 127 (signed) | Most common |
| **UINT8** | 0 to 255 (unsigned) | Requires asymmetric |
| **INT4** | -8 to 7 (signed) | Often packed 2-per-byte |
| **UINT4** | 0 to 15 (unsigned) | Common in modern LLMs |

### Floating-Point Structure: Mantissa & Exponent

Before understanding quantization, it's helpful to understand what we're replacing:

| Component | FP32 Bits | FP16 Bits | Purpose |
|-----------|-----------|----------|---------|
| **Sign** | 1 | 1 | Positive/negative |
| **Exponent** | 8 | 5 | Scale (power of 2) |
| **Mantissa/Fraction** | 23 | 10 | Precision digits |

```
FP32 representation: [-1^(sign)] × 2^[exponent - 127] × 1.mantissa
Example: -0.0012345 = sign(-) × exponent(-7) × mantissa(1.2345)
```

#### Why Quantization Removes These

| FP32 (32-bit) | INT8 (8-bit) |
|--------------|--------------|
| Sign + Exponent + Mantissa (complex) | Raw integer only |
| Requires FPU (floating-point unit) | Uses ALU (arithmetic logic unit) |
| Per-value exponent/mantissa storage | Shared scale replaces exponent |
| Complex decoding per operation | Direct computation |

**Key insight:** When quantizing to INT8, you eliminate the need to store/execute exponent and mantissa. The scale factor (S) replaces the exponent's role for the entire tensor — no per-value mantissa needed.

## Types of Quantization

### Weight Quantization

Compresses the model parameters (weights) stored in the model layers.

**Effect:**
- Reduces model file size on disk
- Cuts memory needed to store weights
- Minimal impact on inference speed alone (weights are pre-stored)

### Activation Quantization

Compresses the intermediate outputs (activations) produced during the forward pass.

**Effect:**
- Reduces memory bandwidth during computation
- Speeds up computation (integer ops faster than float)
- Often combined with weight quantization for maximum efficiency

### Combined (Weight + Activation)

Both weights and activations are quantized. This provides the greatest efficiency gains but requires careful implementation to maintain accuracy.

## Key Trade-offs

### Accuracy vs. Efficiency

- Lower precision → greater compression → more accuracy loss
- Aggressive quantization (INT4 or lower) may degrade output quality
- The sweet spot is often INT8 with minimal quality loss

### Model Size vs. Quality

- 4-bit quantization can reduce size by 8x but may noticeably affect model quality
- 8-bit quantization typically preserves most of the original model's capabilities

## Common Approaches

### Post-Training Quantization (PTQ)

Quantize a trained model after training. Simpler but may lose accuracy.

### Quantization-Aware Training (QAT)

Simulate quantization effects during training. More accurate but computationally expensive.

### Static vs Dynamic Quantization

- **Static**: Pre-compute quantization parameters; faster but less accurate
- **Dynamic**: Compute at runtime; more accurate but slightly slower

## Practical Impact

### Use Cases

- Running large models on consumer GPUs
- Edge device deployment
- Reducing API costs for self-hosted models
- Enabling longer context windows with limited VRAM

### When to Use

- Production deployment where speed/size matters
- Resource-constrained environments
- Batch inference workloads

### When Not to Use

- When maximum accuracy is critical
- Research/experimentation requiring original precision
- Training (quantization primarily helps inference)

## Summary

Quantization transforms high-precision model weights into compact low-precision representations. Weight quantization reduces storage and memory; activation quantization speeds up computation. The key trade-off is between efficiency gains and potential accuracy loss. For most practical applications, INT8 quantization offers a good balance—typically achieving 2-4x compression with minimal quality degradation. Combined weight + activation quantization provides the best efficiency but requires careful implementation.
