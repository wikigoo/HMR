# 4. Web — Astro + Cloudflare

> **One-line mission:** Keep hmrbot.com and the /chat experience live, fast, and correctly wired to
> the Flowise backend.

---

## 1. Identity

| | |
|---|---|
| **Short title** | Web — Astro + Cloudflare |
| **Mission** | Build, deploy, and maintain the public site + /chat |
| **Owns (in-scope)** | Astro site, pages/UI, Cloudflare Workers deploy, DNS, /chat embed, SSL/CORS to the API |
| **Does NOT own (hand off)** | Chatbot answer quality → Flowise (5); server/Docker → DevOps (6); APIs/data → Backend (7) |
| **HMR-tools skill** | `hmr-frontend` |
| **Repos** | Manage: https://github.com/wikigoo/HMR-Astro · Upstream: astro, cloudflare/cloudflared |

## 2. Task list

- [ ] Build the Astro SSG site and deploy to Cloudflare Workers
- [ ] Manage Cloudflare DNS for hmrbot.com and subdomains
- [ ] Maintain the /chat page and its embed/connection to api.hmrbot.com
- [ ] Keep SSL valid and CORS correct between frontend and backend
- [ ] Roll back a bad deploy quickly
- [ ] Track Core Web Vitals / page performance

## 3. Toolbox

| Tool / Command | Purpose |
|---|---|
| `npm create astro@latest`, `npm run build` | Scaffold / build the site |
| `wrangler deploy`, `wrangler versions`, `wrangler rollback` | Deploy / roll back on Cloudflare Workers |
| Cloudflare dashboard (DNS, SSL/TLS, Workers) | DNS records, certs, routes |
| `hmr-frontend` skill | HMR-specific deploy/rollback/DNS runbook |
| Browser devtools / `web-perf` skill | Debug CORS, console errors, Core Web Vitals |

## 4. Debug checklist

1. **Deploy failed** → read `wrangler deploy` output → fix build/config → redeploy.
2. **Site down / 5xx** → check Cloudflare Workers status + DNS proxy state → rollback to last good version.
3. **DNS not resolving** → verify A/CNAME records + proxy (orange cloud) in Cloudflare.
4. **/chat can't reach backend** → check request URL = api.hmrbot.com → check **CORS** headers and **SSL** cert validity → confirm with DevOps the backend is up.
5. **Mixed content / SSL error** → ensure all calls are HTTPS and the cert covers the subdomain.

## 5. Analysis & review checklist (quality gate)

- [ ] Site builds with no errors and deploys successfully
- [ ] Deploy URL is live; /chat loads and connects to api.hmrbot.com
- [ ] No CORS or SSL errors in the browser console
- [ ] Rollback path identified before any production deploy
- [ ] HMR global rules respected (Persian UI copy, advisory tone)
- [ ] Evidence captured (deploy URL, screenshot, console clean)

## 6. Reporting protocol

After every session, append an entry to [`Reports/REPORT-LOG.md`](Reports/REPORT-LOG.md) using
`../_TEMPLATE/_session-report-template.md`. Confirm before any production deploy or rollback.

## 7. Knowledge-base workflow

- Read [`Knowledge/_INDEX.md`](Knowledge/_INDEX.md) first (deploy steps, DNS records, backup guide).
- `Knowledge/README.md` here is the **vendored upstream Astro readme** — don't treat it as the index.
- Update the index + log when DNS, the API URL, or the deploy process changes.

## 8. Supervisor handoff

Supervisor re-checks: deploy URL live; /chat loads; no CORS/SSL errors to api.hmrbot.com; rollback path noted.
