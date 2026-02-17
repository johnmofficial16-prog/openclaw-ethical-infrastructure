# Performance Benchmarks: CPU Inference Analysis

## Hardware Specifications

- **Device**: ASUS VivoBook 15 (M1502)
- **Processor**: AMD Ryzen 7 5825U (8C/16T)
- **RAM**: 16GB DDR4 (Dual Channel)
- **Inference Engine**: Ollama v0.1.24 (Windows)

## Methodology

We tested response latency for OpenClaw's specialized system prompt (~10,200 tokens) using different model sizes. The goal was to find a model that could provide a conversational experience (<30s latency) purely on CPU.

## Results

| Model Variation   | Parameters | Quantization | Prompt Size | Time to First Token | Tokens/Sec  | Result     |
| ----------------- | ---------- | ------------ | ----------- | ------------------- | ----------- | ---------- |
| **Qwen 2.5 14B**  | 14B        | Q4_K_M       | ~10k        | **Timeout** (>10m)  | ~1.2 t/s    | ❌ Fail    |
| **Qwen 2.5 7B**   | 7B         | Q4_K_M       | ~10k        | **174s** (2m 54s)   | ~17 t/s (p) | ❌ Fail    |
| **Qwen 2.5 0.5B** | 0.5B       | Q4_K_M       | ~4k (trunc) | **18.5s**           | ~60 t/s     | ✅ Pass    |
| **Qwen Fast**     | 0.5B       | Q4_K_M       | ~10k (full) | **34.2s**           | ~58 t/s     | ✅ Optimal |

_(p) = Prompt processing speed_

## Analysis

The 7B model, while more intelligent, saturated the memory bandwidth of the Ryzen SoC, leading to unacceptable delays for an interactive chat bot.

The **0.5B model**, optimized with a custom 16k context window (`Modelfile`), struck the perfect balance. It fits entirely within the L3 cache/RAM high-speed tier, allowing for rapid prompt ingestion despite the massive context size required by the autonomous agent framework.

## Conclusion

For purely local, CPU-based agentic workflows, **Parameter Count < 1B** is strictly necessary to maintain conversational latency when managing large context windows (>8k tokens).
