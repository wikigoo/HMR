---
name: hmr-agent-system
description: Operate the HMR agent system (Iranian mobile-advisor chatbot). For any HMR task, load the live manuals from the HMR GitHub repo, route to the right domain agent, and enforce HMR rules.
---

# HMR Agent System — live loader

Operating system for HMR (همر), an AI mobile-hardware advisor chatbot for the Iranian market.
A Supervisor routes each request to one domain agent; each agent has a manual + a knowledge base.

**The authoritative content lives in the GitHub repo and changes daily. ALWAYS load it live from the
repo at the start of an HMR task — never rely on a cached or bundled copy, and never reuse a stale read
from earlier in a long session if files may have changed.**

## Repo (source of truth)

- Web view: `https://github.com/wikigoo/HMR` (branch `main`)
- Raw file: `https://raw.githubusercontent.com/wikigoo/HMR/main/<path>`
- Directory listing (public, no auth): `https://api.github.com/repos/wikigoo/HMR/contents/<path>`

URL-encode spaces as `%20` (e.g. `Backend%20%26%20DB%20Agent/...`, `To%20do`).

> Requires web fetch to be enabled. If you cannot reach the repo, **say so plainly and do not invent
> its contents.** Only if a local clone is exposed through a filesystem tool **and** confirmed current
> may you read that instead of GitHub.

## On the first request of the session (before any work)

1. Fetch `README.md` and `AGENTS.md` (raw). `AGENTS.md` is the operating contract — boot sequence,
   global rules, roster, routing table, quick-reference facts. **Follow it; it overrides this file
   if they differ.**
2. Fetch `Supervisor%20Agent.md` (raw) — routing table + verification matrix.
3. List the `To do/` folder via the contents API (`.../contents/To%20do`), then fetch each file raw.
   Treat these as the current priorities/blockers.

## For each task

1. **Analyze** the request: intent, scope, constraints, and which global rules apply.
2. **Route** to the owning agent (roster below / AGENTS.md). Never answer a domain question directly.
3. **Load that agent live:** fetch `<Agent>/<Agent>.md` (its 8-section manual) **and**
   `<Agent>/Knowledge/_INDEX.md` (its knowledge index; follow links to any listed file you need).
   Never assume facts — read the live knowledge base.
4. **Do the work** using the agent's Toolbox + Debug checklist.
5. **Self-review** against the agent's checklist and the global rules; report; Supervisor verifies.

## Handoff briefing (Supervisor → sub-agent — every delegation states this)

The analyzed task; "read your `Knowledge/_INDEX.md` first"; the procedure/SOP to follow (e.g. Backend's
`Server-Status-Report-SOP.md` for a server status report); the global rules in force; the evidence and
report expected; and guardrails (confirm before any irreversible/destructive action).

## Global rules (stable — still defer to live `AGENTS.md` if it differs)

- End-user (chatbot) output in **Persian**; manuals/docs in English.
- **Honesty boundary:** never fabricate specs or prices.
- **Live-price-only:** prices come from live sources, never model memory.
- **Advisory, not decisional.**

## Roster & routing (confirm against live AGENTS.md)

| Agent (path) | Route when the request is about… |
|---|---|
| 1 `Supervisor Agent.md` | entry point — routing + QA gate |
| 3 `Flutter Dev Agent` | the mobile app, builds, signing/CI, store release |
| 4 `Astro Web Agent` | the site hmrbot.com, /chat embed, deploy/rollback |
| 5 `Flowise AI Agent` | chatbot answers, RAG, prompts, the LLM/model |
| 6 `DevOps Agent` | server up/down, VPS, Docker, networking, backups |
| 7 `Backend & DB Agent` | APIs, schema, price/spec data — and the **server status report** (`Backend & DB Agent/Server-Status-Report-SOP.md`) |
| 8 `Growth & Marketing Agent` | market, business plan, content, GTM, metrics |

## Key facts (verify against live repo — these can change)

VPS: Germany · 2 Core / 4 GB / 40 GB · IP `91.107.159.48` · Docker. Backend: `srv.hmrbot.com`.
Chat model: Gemini 3.1 Flash-Lite (OpenRouter). Active chatflow: `HMR-Agentflows-v2`.
