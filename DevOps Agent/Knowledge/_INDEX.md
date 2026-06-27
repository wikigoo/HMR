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
- **CPU:** 2 Core · **RAM:** 4 GB · **Storage:** 40 GB (≈27 GB free as of 2026-06-27)
- **Snapshot:** none · **Backup:** local file backup **configured & verified** (2026-06-27)
- Runs: Flowise (chatbot backend) reachable as srv.hmrbot.com
  - Running image: `flowiseai/flowise:3.1.3` (pinned in `/opt/flowise/docker/docker-compose.yml`); data volume `~/.flowise` (≈44 MB: `database.sqlite`, secret `.key`s, `faiss/` RAG indexes)
- LLM: Gemini 3.1 Flash-Lite via OpenRouter (outbound from the box)
- ✅ **Backup (local):** `/root/backup_flowise.sh` tars `~/.flowise` → `/root/backups/flowise_<ts>.tar.gz`, runs `gzip -t`, keeps 14 newest; cron daily `30 3 * * *`. Restore-readiness verified (archive holds db + keys + faiss).
- ⚠️ **Still no off-site copy and no provider snapshot** — backups live on the same VPS; a host/disk loss loses them too. Off-site DR is the next ops gap.
- ℹ️ The Backend & Data agent shares this same host/IP — one source of truth.

## To document here

- Container layout + `docker compose` files
- Backup schedule, location, and tested restore procedure
- Firewall/SSH/security baseline
- Rollback runbook

## Review cadence

Re-verify any entry older than 30 days. Update here + log in `../Reports/REPORT-LOG.md` on any change.
