# HMR Agent System

This repo is the **operating manual** for HMR (همر) — an AI mobile-hardware advisor chatbot for the
Iranian market. Work is split across one **Supervisor** and six **domain agents**. Each agent is a
folder with a standardized manual (`<Agent>.md`), a `Knowledge/` base, and a `Reports/` log.

A human or an AI assistant can pick up any agent role by reading its manual — it defines the scope,
tasks, tools, checklists, and where to log the work.

## How the system works

```
            ┌─────────────────────────────┐
   user ──▶ │  Supervisor — Router & QA   │
            └──────────────┬──────────────┘
        classify + route   │   verify + consolidate
            ┌──────────────┴──────────────┐
            ▼      ▼      ▼      ▼      ▼   ▼
          Web   Backend  DevOps Flowise Flutter Growth
            │      │      │      │      │     │
            └──────┴── each writes Reports/REPORT-LOG.md ──┘
                          │
                          ▼
                Supervisor consolidates → Reports/SESSION-SUMMARY.md
```

1. **Supervisor** classifies the request and routes it to the right agent(s).
2. The agent does the work, runs its **review checklist**, and writes a **report**.
3. The Supervisor **verifies** the result (verification matrix) and records a verdict.
4. Facts learned go back into the agent's **Knowledge base**.

## Agent roster

| # | Folder | Short title | Mission |
|---|---|---|---|
| 1 | [`Supervisor Agent.md`](Supervisor%20Agent.md) | **Supervisor — Router & QA Gate** | Route requests, verify sub-agent output, consolidate reports |
| 3 | [`Flutter Dev Agent`](Flutter%20Dev%20Agent/Flutter%20Dev%20Agent.md) | **Mobile — Flutter** | Android/iOS app, signing/CI, store release |
| 4 | [`Astro Web Agent`](Astro%20Web%20Agent/Astro%20Web%20Agent.md) | **Web — Astro + Cloudflare** | Site at hmrbot.com, /chat embed, deploy/rollback |
| 5 | [`Flowise AI Agent`](Flowise%20AI%20Agent/Flowise%20AI%20Agent.md) | **Conversational AI / Chatflow** | AgentFlow V2, RAG, prompts, OpenRouter, flow JSON |
| 6 | [`DevOps Agent`](DevOps%20Agent/DevOps%20Agent.md) | **Infrastructure & Ops** | VPS, Docker, Flowise uptime, networking, backups |
| 7 | [`Backend & DB Agent`](Backend%20&%20DB%20Agent/Backend%20&%20DB%20Agent.md) | **Backend & Data** | APIs, data models, price/spec data, integrations |
| 8 | [`Growth & Marketing Agent`](Growth%20&%20Marketing%20Agent/Growth%20&%20Marketing%20Agent.md) | **Growth & Marketing** | Market, business plan, content, launch/GTM |

> Numbers 1–8 are the historical agent IDs (2 is unused). Keep them stable so reports cross-reference.

## HMR global rules (every agent must respect)

- **Persian output** — user-facing answers are in Persian (فارسی); technical terms/commands stay in English.
- **Honesty boundary** — never fabricate specs, prices, or facts. Say what is unknown.
- **Live-price-only** — quote prices only from a live/approved source, never from memory.
- **Advisory, not decisional** — HMR guides the user; it does not decide for them.

## Standard manual structure (all agents)

1. Identity (scope / hand-offs) · 2. Task list · 3. Toolbox · 4. Debug checklist ·
5. Analysis & review checklist · 6. Reporting protocol · 7. Knowledge-base workflow · 8. Supervisor handoff.

Start a new agent from [`_TEMPLATE/Agent-Template.md`](_TEMPLATE/Agent-Template.md). See
[`CONTRIBUTING.md`](CONTRIBUTING.md) for the request-flow and how to add an agent.

## Project quick reference

| Item | Value |
|------|-------|
| Live site | hmrbot.com |
| Chat | hmrbot.com/chat |
| Flowise backend | srv.hmrbot.com / api.hmrbot.com |
| VPS | Germany (Nuremberg) · Ubuntu 24.04 · 2 Core · 4 GB RAM · 40 GB · IP 91.107.159.48 |
| AI model | Gemini 3.1 Flash-Lite via OpenRouter |
| Frontend | Astro SSG on Cloudflare Workers |
| Active chatflow | HMR-MultiAgent-V6 (latest in repo) |

> If any value above disagrees with reality, the owning agent must fix it and log the change.
