# HMR

Admin workspace for HMR (همر) — an AI mobile-hardware advisor chatbot for the Iranian market. Persian-speaking users; advisor, not decision-maker.

## Mobile AI Chatbot Project

This project is an AI chatbot specialized in mobile technology.

* **Domain:** [hmrbot.com](http://HMRBOT.COM)
* **Chatbot Engine:** Powered by [Flowise](https://github.com/FlowiseAI/Flowise)
* **Website:** Built with [Astro](https://github.com/withastro/astro) and deployed on [Cloudflare](https://github.com/cloudflare)
* **Mobile App:** Developed with [Flutter](https://github.com/flutter/flutter)

## The Five Product Pillars
  
1. New-phone buying guide
2. Used-phone buying guide + fraud / counterfeit detection
3. Hardware troubleshooting & fault diagnosis
4. Hardware education
5. Accessories guidance

## Agent System

Work is organized as a **multi-agent operating manual**: one Supervisor routes and verifies, six
domain agents execute. Start at **[AGENTS.md](AGENTS.md)** for the roster, request lifecycle, and the
global rules every agent follows. See **[CONTRIBUTING.md](CONTRIBUTING.md)** for the workflow and
**[`_TEMPLATE/`](_TEMPLATE/)** for the standard agent/report/knowledge templates.

Each agent folder contains: its manual (`<Agent>.md`), a `Knowledge/` base (`_INDEX.md`), and a
`Reports/` log written after every session.

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
