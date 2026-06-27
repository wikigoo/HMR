# Knowledge Base — Backend & Data — Index

> Read before acting. Add backend/data docs here and list them below.

| File | Purpose | Last reviewed |
|---|---|---|
| [`../Server-Status-Report-SOP.md`](../Server-Status-Report-SOP.md) | On-demand server status report runbook (read-only host/Docker/Flowise/DB/network/backup/security diagnostics) | 2026-06-27 |
| _(add API/schema/data-source docs here)_ | | |

## HMR facts this agent relies on

- Host (same VPS as DevOps): Germany/Nuremberg · IP `91.107.159.48` · Port 22 · root · SSH key `HMRBOT` · Ubuntu 24.04 · 2 Core · 4 GB RAM · 40 GB · Docker
- ✅ Local file backup configured & verified (2026-06-27): `/root/backup_flowise.sh` (daily cron 03:30) archives `~/.flowise` incl. `database.sqlite`. ⚠️ No off-site copy / provider snapshot yet — still take a manual off-host copy before destructive ops.
- Data must be **live-sourced** (price/spec) — never fabricated, never from model memory
- Consumers of the API: Flowise chatflow (5), Flutter app (3), Astro site (4)

## To document here

- API endpoint list + contracts
- Database schema and migrations
- Price/spec data sources and the ingestion job
- Secret/key inventory (names + location, **not values**)

## Review cadence

Re-verify any entry older than 30 days. Update here + log in `../Reports/REPORT-LOG.md` on any change.
