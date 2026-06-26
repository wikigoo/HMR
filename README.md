# HMR

HMR (همر) — an **AI mobile-hardware advisor chatbot for the Iranian market**. Persian-speaking users;
it **advises**, it does not decide for them. This repository is the project's **operating manual**:
one Supervisor agent routes and verifies work across six domain agents.

> 🤖 **AI assistants start here:** read **[`AGENTS.md`](AGENTS.md)** and follow it. It is the operating
> contract — boot sequence, global rules, agent roster, and the request lifecycle all live there.
> Typical prompt: *"Read AGENTS.md and start operating HMR. Task: …"*

## What's in this repo

| Path | What it is |
|---|---|
| [`AGENTS.md`](AGENTS.md) | **The operating contract** — boot sequence, global rules, roster, lifecycle |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Request lifecycle + how to add an agent |
| [`Supervisor Agent.md`](Supervisor%20Agent.md) | Routing table + verification protocol |
| `<Agent>/<Agent>.md` | The 8-section manual for each of the 6 domain agents |
| `<Agent>/Knowledge/_INDEX.md` · `<Agent>/Reports/` | Per-agent knowledge base + session logs |
| [`_TEMPLATE/`](_TEMPLATE/) | Templates for new agents, reports, knowledge indexes |

## The Five Product Pillars

1. New-phone buying guide
2. Used-phone buying guide + fraud / counterfeit detection
3. Hardware troubleshooting & fault diagnosis
4. Hardware education
5. Accessories guidance

## Quick Reference

| Item | Value |
|------|-------|
| Live site | hmrbot.com |
| Chat | hmrbot.com/chat |
| Flowise backend | srv.hmrbot.com / api.hmrbot.com |
| VPS | Germany (Nuremberg) · Ubuntu 24.04 · 2 Core · 4 GB RAM · 40 GB · IP 91.107.159.48 |
| AI model | [Gemini 3.1 Flash-Lite](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/gemini/3-1-flash-lite) via OpenRouter |
| Frontend | Astro SSG on Cloudflare Workers |
| Active chatflow | HMR-MultiAgent-V6 (latest in repo) |

> The global rules (Persian output · honesty · live-price-only · advisory-not-decisional) and the full
> agent workflow are defined in [`AGENTS.md`](AGENTS.md) — the single source of truth.
