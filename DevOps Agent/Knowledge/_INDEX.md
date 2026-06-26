# Knowledge Base — Infrastructure & Ops — Index

> Read before acting. Add server/Docker/backup docs here and list them below.

| File | Purpose | Last reviewed |
|---|---|---|
| _(none yet — add host/Docker/backup runbooks)_ | | |

## HMR facts this agent relies on (confirmed VPS spec)

- **Location:** Germany — Nuremberg
- **IP:** `91.107.159.48` (1 IP) · **Cloud type:** HIN4
- **Access:** user `root` · SSH key `HMRBOT` · alias `ssh hmrbot-server`
- **OS:** ubuntu-24.04 · **Docker**
- **CPU:** 2 Core · **RAM:** 4 GB · **Storage:** 40 GB
- **Snapshot:** none · **Backup:** none configured
- Runs: Flowise (chatbot backend) reachable as srv.hmrbot.com / api.hmrbot.com
- LLM: Gemini 3.1 Flash-Lite via OpenRouter (outbound from the box)
- ⚠️ **No backup/snapshot configured** — highest-priority ops gap; set up backups before risky changes.
- ℹ️ The Backend & Data agent shares this same host/IP — one source of truth.

## To document here

- Container layout + `docker compose` files
- Backup schedule, location, and tested restore procedure
- Firewall/SSH/security baseline
- Rollback runbook

## Review cadence

Re-verify any entry older than 30 days. Update here + log in `../Reports/REPORT-LOG.md` on any change.
