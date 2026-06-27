# Server Status Report — SOP (Backend & Data, agent 7)

> **Purpose:** A single, repeatable procedure for producing a complete, professional **server status
> report** for the HMR VPS on demand. When the user asks for "server status", "how's the server",
> "گزارش وضعیت سرور", or similar, follow this document **exactly**, end to end.
>
> **Mode:** **READ-ONLY diagnostics.** This SOP never changes server state. It inspects, measures, and
> reports. Any fix it surfaces is **handed off to DevOps (6)** — the host/Docker owner — with evidence.
>
> **Coordination:** Backend & DevOps share the same VPS (IP `91.107.159.48`). Run read-only checks
> freely; coordinate with DevOps before anything that could touch state, and never restart/stop/scale
> anything from this SOP.

---

## 0. Preconditions & safety

- [ ] Confirm the run is **authorized** and **read-only** is acceptable (it always should be here).
- [ ] Have SSH access: `ssh hmrbot-server` (alias) or `ssh root@91.107.159.48` (port 22, key `HMRBOT`).
- [ ] **Never print secrets.** Do not `cat` `.env`, keys, tokens, or `key.properties`. Report a secret's
      **presence and location only**, never its value.
- [ ] **No state changes.** Forbidden in this SOP: `restart`, `stop`, `up`, `down`, `rm`, `prune`,
      `apt upgrade`, `ufw` edits, file writes, DB writes. Those belong to **DevOps (6)**.
- [ ] If a check requires a state change to diagnose, **stop and hand off to DevOps** — log why.
- [ ] Capture **evidence** (command + trimmed output) for every section; the report must be verifiable.

### Known host facts (verify, don't assume — re-check against the live box)

| Item | Expected value |
|---|---|
| Location / IP | Germany (Nuremberg) · `91.107.159.48` · port 22 |
| Access | user `root` · SSH key `HMRBOT` · alias `ssh hmrbot-server` |
| OS / engine | Ubuntu 24.04 · Docker |
| Resources | 2 Core · 4 GB RAM · 40 GB disk |
| Flowise | image `flowiseai/flowise:3.1.3` · compose `/opt/flowise/docker/docker-compose.yml` · data `~/.flowise` (`database.sqlite`, `.key`, `faiss/`) |
| Public backend | `srv.hmrbot.com` (Flowise behind Nginx) |
| Prediction API | `https://srv.hmrbot.com/api/v1/prediction/<chatflow-id>` |
| Backup | `/root/backup_flowise.sh` · cron `30 3 * * *` · `/root/backups/` · keeps 14 · `gzip -t` verified |
| Optional proxy | `hmr-proxy` (Node/Express) on `127.0.0.1:8081`, Nginx `location /predict` |

---

## How to use this SOP

Run sections **1 → 12 in order**. Each section has: **Goal · Commands (read-only) · Capture · Pass /
Warn / Fail**. Record a status (`✅ OK` / `⚠️ Warn` / `❌ Fail`) and the evidence for each. Then fill the
**Report template** (§13) and append it to `Reports/REPORT-LOG.md`. End by handing any `⚠️`/`❌` items to
DevOps (6) with the evidence.

---

## 1. Host identity, OS & time

**Goal:** Confirm we're on the right box, OS is current, clock is synced (token/SSL/JWT depend on it).

```bash
hostnamectl                         # host, OS, kernel, virtualization
uname -r                            # kernel
uptime -p ; uptime                  # how long up + load averages
timedatectl                         # time zone + NTP sync status
cat /etc/os-release | head -n 3     # distro
```

**Capture:** hostname, OS/kernel, uptime, NTP "System clock synchronized: yes".
**Pass:** OS = Ubuntu 24.04, NTP synced. **Warn:** uptime < 10 min (recent reboot — investigate why).
**Fail:** clock not synced, or wrong host.

## 2. CPU & load

**Goal:** Is the 2-core box overloaded?

```bash
nproc                               # core count (expect 2)
uptime                              # 1/5/15-min load averages
top -bn1 | head -n 15               # top processes by CPU
mpstat 1 3 2>/dev/null || vmstat 1 3 # %idle / steal (install not required)
```

**Capture:** core count, load averages, top 3 CPU consumers, `%idle`, `%steal`.
**Pass:** 15-min load < cores (< 2.0). **Warn:** load between 1×–2× cores, or `%steal` > 5% (noisy
neighbour). **Fail:** load > 2× cores sustained, or `%idle` near 0.

## 3. Memory & swap

**Goal:** 4 GB is tight with Flowise + Node + Nginx — watch for OOM risk.

```bash
free -m                             # used/free/available RAM + swap
docker stats --no-stream            # per-container mem (also used in §5)
dmesg -T 2>/dev/null | grep -i -E 'oom|killed process' | tail -n 5   # past OOM kills
```

**Capture:** total/used/available MB, swap used, any OOM-kill lines.
**Pass:** available > 500 MB, swap < 20% used, no recent OOM. **Warn:** available 200–500 MB or swap
20–60%. **Fail:** available < 200 MB, heavy swapping, or OOM kills present.

## 4. Disk & inodes

**Goal:** 40 GB disk must not fill (DB writes, logs, backups, Docker images all grow).

```bash
df -h /                             # root usage (expect ≈27 GB free baseline)
df -i /                            # inode usage
du -sh /var/lib/docker 2>/dev/null  # Docker footprint
du -sh /root/backups 2>/dev/null    # backup footprint
docker system df                    # images/containers/volumes/build cache sizes
```

**Capture:** disk %used + free GB, inode %used, Docker + backup sizes.
**Pass:** disk < 75% and inodes < 75%. **Warn:** 75–90%. **Fail:** > 90% (recommend DevOps prune /
log-rotate / off-load backups — do not prune here).

## 5. Docker engine & containers

**Goal:** Engine healthy; expected containers running and not restart-looping.

```bash
docker version --format '{{.Server.Version}}'   # engine up?
systemctl is-active docker                      # service state
docker ps -a                                    # all containers + STATUS + uptime
docker stats --no-stream                        # live CPU/mem/net/IO per container
docker ps --filter health=unhealthy             # any unhealthy?
docker inspect -f '{{.Name}} restarts={{.RestartCount}}' $(docker ps -q)  # restart loops
```

**Capture:** engine version, table of containers (name, image, status, uptime, restarts), unhealthy list.
**Pass:** all expected containers `Up`, health `healthy`/none, restarts stable. **Warn:** a container
restarted recently or sits at high mem. **Fail:** an expected container is `Exited`/`Restarting` loop or
`unhealthy`.

## 6. Flowise health (the core service)

**Goal:** Flowise is up, serving, and answering — this is the chatbot backend.

```bash
docker logs --tail 50 $(docker ps -qf "ancestor=flowiseai/flowise:3.1.3") 2>&1 | tail -n 50  # recent logs/errors
curl -s -o /dev/null -w "%{http_code} %{time_total}s\n" https://srv.hmrbot.com/api/v1/ping   # edge reachability
# Prediction API smoke test (read-only, harmless question). Replace <chatflow-id>; use a server-side
# token if the endpoint is protected — never paste the token into the report.
curl -s -m 30 -X POST https://srv.hmrbot.com/api/v1/prediction/<chatflow-id> \
  -H "Content-Type: application/json" \
  -d '{"question":"سلام، تست سلامت سرویس"}' -o /tmp/hmr_ping.json -w "%{http_code} %{time_total}s\n"
head -c 300 /tmp/hmr_ping.json
```

**Capture:** Flowise log error count (last 50 lines), edge HTTP code + latency, prediction HTTP code +
latency + a snippet of the `text` field.
**Pass:** logs clean, edge 200, prediction 200 with a non-empty Persian `text`. **Warn:** elevated
latency (> 8 s) or sporadic non-fatal log errors. **Fail:** non-200, empty/`error` body, or repeated
stack traces in logs.

> Note the smoke test verifies the **path** is alive. It does **not** validate answer quality or the
> live-price rule — that's the Flowise agent's (5) job; hand off if the content looks wrong.

## 7. Backend API / proxy health (if `hmr-proxy` is deployed)

**Goal:** If the token-hiding proxy is live, confirm `/predict` works and rate-limits.

```bash
docker ps -a --filter name=hmr-proxy            # proxy container present?
ss -tlnp | grep 8081 2>/dev/null                # listening on 127.0.0.1:8081
docker logs --tail 30 hmr-proxy 2>&1 | tail -n 30
curl -s -o /dev/null -w "%{http_code}\n" -X POST https://srv.hmrbot.com/predict \
  -H "Content-Type: application/json" -H "x-app-key: <app-key>" \
  -d '{"question":"تست"}'                        # expect 200; 401 without key
```

**Capture:** proxy up? port bound? 200 with key / 401 without key / 429 when rate-limited.
**Pass:** proxy `Up`, `/predict` returns 200 with key and 401 without. **Warn:** proxy present but
errors in logs. **Fail:** proxy expected but down, or `/predict` 5xx. *(If the proxy isn't deployed yet,
mark "N/A — not deployed" and note it as a known gap.)*

## 8. Database

**Goal:** Verify the data store is intact and not corrupt.

```bash
# Flowise SQLite (inside the data volume). Read-only integrity check.
ls -lh ~/.flowise/database.sqlite
sqlite3 ~/.flowise/database.sqlite "PRAGMA integrity_check;" 2>/dev/null || \
  echo "sqlite3 not on host — run integrity_check from inside the container or via DevOps"
# If a separate price/spec DB exists (psql/mysql), check connectivity read-only:
# psql "$DB_URL" -c "SELECT 1;"   /   mysql -e "SELECT 1;"   (never print the DSN/secret)
```

**Capture:** DB file size, `integrity_check` = `ok`, connectivity probe result (1/SELECT 1).
**Pass:** `integrity_check ok`, file present, probe returns. **Warn:** DB growing unusually fast.
**Fail:** integrity errors, file missing, or connection refused.

## 9. Network, ports, firewall, Nginx & TLS

**Goal:** Only intended ports are open; reverse proxy is healthy; the cert isn't about to expire.

```bash
ss -tlnp                                          # listening sockets (expect 22, 80, 443, internal 3000/8081)
ufw status verbose 2>/dev/null || iptables -S | head  # firewall posture
nginx -t 2>&1                                     # Nginx config valid?
systemctl is-active nginx                         # Nginx running?
# TLS expiry for the public backend:
echo | openssl s_client -servername srv.hmrbot.com -connect srv.hmrbot.com:443 2>/dev/null \
  | openssl x509 -noout -dates -subject
getent hosts srv.hmrbot.com                       # DNS resolves to this box?
```

**Capture:** open ports list, firewall state, Nginx valid+active, cert `notAfter` (days remaining), DNS
resolution.
**Pass:** only expected ports public (22/80/443), Nginx OK, cert > 21 days, DNS correct. **Warn:** cert
7–21 days to expiry, or an unexpected high port open. **Fail:** cert expired/< 7 days, Nginx config
invalid/down, or sensitive port (e.g. 3000) exposed to `0.0.0.0`.

## 10. OpenRouter outbound connectivity

**Goal:** The box can reach the LLM provider (chat/embeddings/rerank all depend on it).

```bash
curl -s -o /dev/null -w "%{http_code} %{time_total}s\n" https://openrouter.ai/api/v1/models
# Auth probe (only if a key is available server-side; never echo the key):
# curl -s -o /dev/null -w "%{http_code}\n" https://openrouter.ai/api/v1/key -H "Authorization: Bearer $OPENROUTER_KEY"
```

**Capture:** reachability HTTP code + latency; key-status code if probed.
**Pass:** 200 reachable, key valid (200). **Warn:** high latency or intermittent. **Fail:** unreachable,
DNS/egress blocked, or key 401/402 (expired/no credit).

## 11. Backups

**Goal:** A recent, verified backup exists (the data is the project's core asset).

```bash
ls -lh /root/backups | tail -n 5                  # newest archives
stat -c '%y %n' /root/backups/* 2>/dev/null | sort | tail -n 1   # most recent backup time
gzip -t /root/backups/$(ls -t /root/backups | head -n1) && echo "integrity: OK"  # verify newest
crontab -l 2>/dev/null | grep -i backup           # cron entry present (expect 30 3 * * *)
```

**Capture:** newest backup name + age, integrity result, cron present, count retained.
**Pass:** newest backup < 24 h old, `gzip -t` OK, cron present. **Warn:** 24–48 h old, or fewer than
expected retained. **Fail:** > 48 h old / missing / integrity fail.
**Always note the standing gap:** ⚠️ backups are **on the same VPS** — no off-site copy / provider
snapshot yet. Flag off-site DR as an open recommendation every time.

## 12. Security posture

**Goal:** SSH hardened, intrusion protection active, updates not dangerously behind.

```bash
sshd -T 2>/dev/null | grep -E 'permitrootlogin|passwordauthentication'  # SSH policy
systemctl is-active fail2ban 2>/dev/null ; fail2ban-client status sshd 2>/dev/null | tail -n 5
grep -c 'Failed password' /var/log/auth.log 2>/dev/null                 # brute-force noise
apt-get -s upgrade 2>/dev/null | grep -i security | wc -l               # pending security updates (dry-run)
who ; last -n 5                                                          # current + recent logins
```

**Capture:** root-login/password-auth policy, fail2ban active + jail status, failed-login count,
pending security updates count, recent logins.
**Pass:** key-only SSH, fail2ban active, no surprising logins, few pending security updates.
**Warn:** password auth enabled, moderate failed-login volume, several pending security updates.
**Fail:** root password login enabled, fail2ban down with active brute-force, or unrecognized successful
login (treat as incident → escalate immediately).

---

## 13. Report template (fill and append to `Reports/REPORT-LOG.md`)

```markdown
## <YYYY-MM-DD HH:MM> — Server Status Report

- **Agent:** Backend & Data (7)  ·  **Triggered by:** <user/Supervisor>  ·  **Mode:** read-only
- **Host:** 91.107.159.48 (Ubuntu 24.04 · 2C/4GB/40GB · Docker)

### Overall verdict: <✅ Healthy | ⚠️ Degraded | ❌ Incident>

| # | Area | Status | Key figures | Notes |
|---|------|--------|-------------|-------|
| 1 | Host/OS/time | ✅/⚠️/❌ | uptime …, NTP … | |
| 2 | CPU/load | | load …/…/… | |
| 3 | Memory/swap | | avail … MB, swap …% | |
| 4 | Disk/inodes | | …% used, … GB free | |
| 5 | Docker/containers | | … up / … total | |
| 6 | Flowise | | edge …, predict … (…s) | |
| 7 | Proxy (/predict) | | 200/401/429 or N/A | |
| 8 | Database | | integrity … | |
| 9 | Net/Nginx/TLS | | cert … days, ports … | |
| 10 | OpenRouter | | … (…s) | |
| 11 | Backups | | newest …h old, gzip … | off-site: ⚠️ none |
| 12 | Security | | fail2ban …, updates … | |

### Evidence
- <command → trimmed output per area>

### Issues & risks (ranked)
- <❌/⚠️ items, most severe first>

### Recommendations & hand-offs
- → **DevOps (6):** <any fix requiring a state change — restart, prune, cert renew, firewall, updates>
- → **Flowise (5):** <if answer quality / chatflow looks wrong>
- <other follow-ups>

- **Supervisor verdict:** <Approved | Needs-rework: reason | Pending>
```

---

## 14. Quality gate (before submitting the report)

- [ ] Every section 1–12 has a status + evidence (no blanks; mark `N/A` with a reason).
- [ ] **No secret value** appears anywhere in the report (presence/location only).
- [ ] All `⚠️`/`❌` items have a concrete recommendation and a named hand-off owner.
- [ ] Overall verdict matches the worst section status (one `❌` ⇒ not "Healthy").
- [ ] Live facts that disagree with §0 "Known host facts" are flagged and the Knowledge base is updated.
- [ ] Report appended to `Reports/REPORT-LOG.md`; Supervisor verification requested.
