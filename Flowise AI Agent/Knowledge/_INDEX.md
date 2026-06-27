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

- Model: Gemini 3.1 Flash-Lite via OpenRouter
- Output rules: Persian, honest (no fabricated specs/prices), live-price-only, advisory-not-decisional
- Backend reachable as srv.hmrbot.com / api.hmrbot.com

## Review cadence

Re-verify any entry older than 30 days. Update here + log in `../Reports/REPORT-LOG.md` on any flow/model/RAG change.
