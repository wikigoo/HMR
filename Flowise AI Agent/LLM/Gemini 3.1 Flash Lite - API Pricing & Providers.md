---
title: "Gemini 3.1 Flash Lite — API Pricing & Providers"
source: "https://openrouter.ai/google/gemini-3.1-flash-lite"
captured: 2026-06-27
hmr_role: "Primary chat LLM for the HMR chatflow (via OpenRouter)"
---

# Google: Gemini 3.1 Flash Lite

Google's GA high-efficiency multimodal model, optimized for low-latency, high-volume workloads.
Supports text, image, video, audio, and PDF input. Built for lightweight agentic workflows, simple
data extraction, and apps where responsiveness and API cost are the main constraints. Supports full
thinking levels (minimal / low / medium / high). Priced at half the cost of Gemini 3 Flash.

## Key facts

| | |
|---|---|
| Input / Output price | **$0.25 / $1.50** per 1M tokens |
| Cache read / Audio cache | $0.025 / $0.05 per 1M |
| Context window | 1,048,576 tokens (1M) |
| Max output | 65,536 tokens |
| Released | May 7, 2026 |
| Providers on OpenRouter | 2 (Google Vertex, Google AI Studio) |

## Providers (list price)

| Provider | Input /M | Output /M | Cache Read /M | Audio Cache /M | Latency | Throughput | Uptime |
|---|---|---|---|---|---|---|---|
| Provider A | $0.25 | $1.50 | $0.025 | $0.05 | 0.82s | 86 tps | 99.06% |
| Provider B | $0.25 | $1.50 | $0.025 | $0.05 | 0.61s | 92 tps | 99.41% |

## Effective pricing (30-day rolling, after prompt caching)

Weighted avg: **Input $0.186 /M · Output $1.50 /M**

| Provider | Input $/1M | Output $/1M | Cache hit rate | Token share |
|---|---|---|---|---|
| Google Vertex | $0.215 | $1.50 | 17.9% | 56.7% |
| Google AI Studio | $0.150 | $1.48 | 45.1% | 43.3% |

## Performance & uptime

- Throughput (best): 92 tok/s · Latency p50 (best): 0.60s
- Google AI Studio — avg 105 tok/s · latency 0.61s · E2E 1.72s
- Google Vertex — avg 99 tok/s · latency 1.06s · E2E 2.84s
- Avg provider uptime (3d): 98.59%

## Activity (recent)

Prompt 18.5B · Completion 1.68B · Reasoning 516M tokens.

> Pricing/performance numbers are point-in-time (captured 2026-06-27). Charts and the upstream model
> catalog were removed; re-verify against the source URL if these drive a decision.
