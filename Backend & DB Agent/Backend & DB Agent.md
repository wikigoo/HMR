# 7. Backend & Data

> **One-line mission:** Provide reliable APIs and trustworthy phone price/spec data to the chatbot
> and the apps — never fabricated, always live-sourced.

---

## 1. Identity

| | |
|---|---|
| **Short title** | Backend & Data |
| **Mission** | APIs, data models, and the price/spec data pipeline |
| **Owns (in-scope)** | API endpoints, database schema, price/spec ingestion, third-party integrations, auth/secrets |
| **Does NOT own (hand off)** | Server/Docker host → DevOps (6); answer wording → Flowise (5); site → Web (4) |
| **HMR-tools skill** | (shares the server host with `hmr-server`) |
| **Host** | Germany (Nuremberg) · IP `91.107.159.48` · Port 22 · Ubuntu 24.04 · 2 Core · 4 GB RAM · 40 GB · Docker (same VPS as DevOps; root, SSH key `HMRBOT`) |

## 2. Task list

- [ ] Design and maintain API endpoints the chatbot/apps call
- [ ] Own the data schema for phones, prices, specs, and accessories
- [ ] Run the **live** price/spec ingestion and keep it fresh
- [ ] Maintain third-party integrations (data sources, OpenRouter-adjacent services)
- [ ] Manage auth, API keys, and secret storage
- [ ] Provide DevOps with backup-relevant DB details

## 3. Toolbox

| Tool / Command | Purpose |
|---|---|
| `ssh root@91.107.159.48` | Reach the host (coordinate with DevOps) |
| `docker ps`, `docker logs`, `docker compose` | Inspect/run backend + DB containers |
| DB client (psql / mysql / sqlite per stack) | Schema + queries |
| `curl` / REST client | Hit and verify API endpoints |
| Secret store / `.env` | API keys, credentials (never commit) |

## 4. Debug checklist

1. **API 5xx** → `docker logs <api>` → read stack trace → fix → redeploy container.
2. **DB connection refused** → container up? credentials/host/port correct? network reachable?
3. **Stale / wrong data** → check the ingestion job ran → verify the **live source** responded → re-run ingestion.
4. **Integration timeout** → check the upstream is reachable + key valid → add retry/backoff.
5. **Auth failures** → key expired/rotated? clock skew? scope correct?

## 5. Analysis & review checklist (quality gate)

- [ ] Endpoint returns correct data with expected status codes
- [ ] **Data is live-sourced, not fabricated** (honesty + live-price-only rules)
- [ ] Schema change is backward-compatible (or migration provided)
- [ ] No secret/API key leaked into logs, repo, or responses
- [ ] Evidence captured (curl output, query result, log excerpt)

## 6. Reporting protocol

After every session, append an entry to [`Reports/REPORT-LOG.md`](Reports/REPORT-LOG.md). Confirm
before destructive DB operations (drop/alter/delete) — request a backup from DevOps first.

## 7. Knowledge-base workflow

- Read [`Knowledge/_INDEX.md`](Knowledge/_INDEX.md) first (schema, data sources, endpoint list, host facts).
- Update the index + log whenever the schema, a data source, or the host IP changes.

## 8. Supervisor handoff

Supervisor re-checks: data is live-sourced (not memory); schema change is backward-compatible; no secrets leaked.
