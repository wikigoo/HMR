# Knowledge Base — Conversational AI / Flowise — Index

> Read before acting. This is the largest KB. The most HMR-relevant guides are listed first;
> `configuration/` and `integrations/` are **vendored upstream Flowise docs** kept for reference.
> `README.md` here is the vendored upstream readme — not the index.

## HMR-relevant guides (read these first)

| File | Purpose | Last reviewed |
|---|---|---|
| `Complete Guide_ Flowise Multi-Agent Architecture (Supervisor + Worker)_.md` | The Supervisor+Worker pattern HMR uses | 2026-06-26 |
| `Complete-Guide-to-Document-Stores-in-FlowiseAI.md` | RAG / Document Store setup | 2026-06-26 |
| `FlowiseAI Prompt System Structure.md` | Prompt structure for the chatflow | 2026-06-26 |
| `API-settings.md` | Prediction API + API settings | 2026-06-26 |

## LLM / model reference (in `../LLM/`)

Cleaned OpenRouter snapshots (charts + upstream model catalog stripped). Captured 2026-06-27 —
re-verify pricing/uptime at the source URL before any cost decision.

| File | Model | HMR role | Last reviewed |
|---|---|---|---|
| `../LLM/Gemini 3.1 Flash Lite - API Pricing & Providers.md` | Gemini 3.1 Flash Lite | Primary chat LLM | 2026-06-27 |
| `../LLM/Rerank 4 Pro - API Pricing & Providers.md` | Cohere Rerank 4 Pro | RAG reranker | 2026-06-27 |
| `../LLM/Text Embedding 3 Large - API Pricing & Providers.md` | OpenAI Text Embedding 3 Large | RAG embeddings | 2026-06-27 |

## Flow artifacts (in `../HMR-Chatbot/`)

| File | Status |
|---|---|
| `HMR-Agentflows-v2 Agents.json` | **Active flow (production)** |
| `HMR-Agentflows-v1 Agents.json` | Previous version (legacy/archive) |
| `Instructions.md` | Flow build/run instructions — keep in sync with the active flow |

## Vendored upstream docs (reference only)

- `configuration/` — env vars, databases, auth, queue, proxy, production
- `integrations/` — LangChain, 3rd-party platforms, etc.

## HMR facts this agent relies on

- Models (via OpenRouter): chat = Gemini 3.1 Flash-Lite · embeddings = OpenAI Text Embedding 3 Large · reranker = Cohere Rerank 4 Pro (see `../LLM/`)
- Output rules: Persian, honest (no fabricated specs/prices), live-price-only, advisory-not-decisional
- Backend reachable as srv.hmrbot.com
- Active chatflow: `HMR-Agentflows-v2 Agents.json`

## Review cadence

Re-verify any entry older than 30 days. Update here + log in `../Reports/REPORT-LOG.md` on any flow/model/RAG change.
