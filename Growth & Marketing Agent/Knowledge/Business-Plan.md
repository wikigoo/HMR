# Business Plan — HMR (همر / Hamer)
### Specialized Mobile Smart Assistant for the Iranian Market
*Internal document — v2 | Audience: founding & engineering team*

---

## 1. Executive Summary

HMR is an AI conversational assistant that guides Persian-speaking users in Iran across the full mobile lifecycle: choosing a new or used phone, diagnosing hardware faults, learning how a device works, and selecting accessories.

General language models (ChatGPT, Gemini) fail on this job because they have no structured, Persian-language, Iran-specific knowledge. HMR's defensible core is a **curated knowledge base built on official manufacturer manuals (Samsung, Xiaomi, Apple)**, structured and indexed for the Iranian market, plus the conversation data the product accumulates over time.

Pricing information is handled differently and deliberately: HMR does **not** maintain a price database. When asked about prices, it runs a live web search across three reputable Iranian sources and presents the results with an explicit disclaimer — HMR takes no responsibility for the fairness or accuracy of any price. This keeps the product useful on pricing without taking on a data-maintenance burden or liability we cannot defend.

**Strategy:** Build trust first with a free, accurate hardware assistant. Monetize gradually — limited in-app advertising in the early phases, then message caps with an optional user subscription, then a B2B personalized-chatbot subscription for small businesses.

---

## 2. The Problem

**2.1 — General AI has no native data.**
When a user asks ChatGPT "What's a used iPhone 13 worth in Iran?" or "Is the iPhone 14 registry issue resolved?", the answer is wrong, stale, or about another country's market. These confident-but-wrong answers (hallucinations) can cause real financial loss.

**2.2 — The second-hand market runs on distrust.**
Currency pressure and sanctions have pushed mid-range phone prices up several-fold versus a few years ago. More than half of classified-ad listings are for used goods, in a market full of smuggled phones, no-warranty repairs, and counterfeit parts. The seller knows the device's condition; the buyer does not. This information asymmetry is the source of buyer fear — and our opportunity.

**2.3 — Expert advice is expensive and not always available.**
A trustworthy mobile specialist is costly, slow to reach, and effectively unavailable after hours or in smaller cities. Most buyers fall back on Telegram groups and guesswork — neither structured nor reliable.

---

## 3. Solution — The Product

An AI assistant that walks the user through every mobile-related decision, focused on **hardware** in Phase 1, with software/OS topics added later.

### 3.1 — Five product pillars
1. **New phone buying guide** — compare models by budget, needs, and user profile.
2. **Used phone buying guide** — inspection checklist, registry status, and detection of replaced or counterfeit parts. (For price, see §3.3.)
3. **Troubleshooting & fault diagnosis** — user describes the symptom; HMR identifies the likely cause and gives a fix or refers to a reliable repair shop.
4. **Hardware education** — camera, RAM, processor, screen, battery explained in plain language.
5. **Accessories guide** — the right case, screen protector, charger, and headphones for the model and the realities of the Iranian market.

### 3.2 — Honesty boundary and human referral
If an answer is **not** in the knowledge base, HMR says so plainly ("I don't have enough information on that") instead of inventing one, and refers the user to a human expert or a reputable repair shop. This is both quality control and a future referral channel.

### 3.3 — Pricing: live search, no liability
HMR does not store or vouch for prices. On a price question it performs a live web search across three reputable Iranian sources, returns what it finds, and shows a standing disclaimer: *prices are sourced live from third parties; HMR makes no guarantee of their accuracy or fairness, and the final judgment is the user's.* This is a convenience feature, explicitly **not** a guaranteed-accurate data product.

---

## 4. The Core Asset & Differentiation

The single most important asset is the **knowledge base**, accessed via a RAG (retrieval-augmented generation) architecture. The language model is the engine; the knowledge base is the asset.

Be clear internally about *why* this is defensible. The manuals themselves are public — anyone can download them. The moat is not access to manuals; it is:

1. **Curation and structuring.** Hundreds of manuals downloaded, de-duplicated, organized by model/year/language, indexed, and made answerable in Persian for Iranian use cases. This is slow, costly work that competitors would have to redo from scratch.
2. **Accumulated conversation data.** Every interaction produces Persian behavioral and intent data that improves answers over time and can later fine-tune a dedicated model. This compounds and cannot be copied.
3. **Trust and public track record.** In this market, no user — individual or business — pays before they have tested the product and seen a record of accurate answers. Each month of reliable performance is a brick in that record.

Pricing is **not** part of the moat (see §3.3) — and that's a deliberate, honest distinction.

---

## 5. Technical Architecture

| Component | Detail |
|---|---|
| Product name | HMR (همر / Hamer) |
| Main domain | hmrbot.com |
| Landing + chat | hmrbot.com · hmrbot.com/ai |
| Blog | hmrbot.com/blog |
| Admin dashboard | api.hmrbot.com |
| Frontend | Astro (SSG) on Cloudflare |
| Chatbot platform | Flowise (AgentFlow V2) on a Linux VPS |
| Primary AI model | Gemini 3.1 Flash Lite via OpenRouter — 1M context, low-latency, high-volume optimized |
| Server-ops agent | ChatREM, backed by DeepSeek V4 Pro — log review, error prediction, auto-restart |
| Flow design | AgentFlow V2 — explicit node-based orchestration, shared state, human-in-the-loop, SSE streaming |

*Specs above are confirmed current against provider documentation (OpenRouter / Flowise, as of the uploaded data). Confirm live OpenRouter pricing and provider uptime at provisioning time — both move.*

### 5.1 — Knowledge base pipeline
1. Download and categorize official manuals (PDF) on Windows — by model, year, language.
2. Build an index file and a guide map.
3. Use an AI file-management agent to automate processing and upload into the Flowise knowledge base.
4. Final upload to the knowledge base on the VPS.

### 5.2 — Server management
The VPS is supervised by **ChatREM** (on DeepSeek V4 Pro): it checks logs, predicts errors, and performs automatic restarts, minimizing the need for a human admin.

---

## 6. Phasing

**Phase 1 — Three best-selling brands (~3 months).**
Samsung, Xiaomi, Apple, focused on their best-selling models in Iran. Sources: official manuals, specialized Persian forums, and live price lookups (Divar, Sheypoor). Output: a hardware assistant that answers ~80% of common questions on these three brands.

**Phase 2 — Ten popular brands (~3–4 months after Phase 1).**
Adds Nokia, Huawei, Honor, Poco, Redmi, Google Pixel, OnePlus (prioritized by Iranian search volume). Sources: more official manuals, repair videos, community reports. Output: coverage of >95% of models in the Iranian second-hand market.

**Phase 3 — Personalized chatbot for small businesses.**
A white-label version of the HMR engine that businesses (lawyers, dentists, real-estate offices, small shops) install on their site or messenger and feed with their own data — hours, services, prices, contact forms. Requires scalable cloud infrastructure and a central dashboard to manage many tenants.

---

## 7. Market & Competitors

### 7.1 — Sizing (directional)
Mobile penetration in Iran exceeds 100% (well over 160 million active SIMs); used goods make up the majority of classified listings. The widely cited ~$21.65B figure is the **whole Iranian e-commerce market by 2029** — it is a backdrop, not our addressable market. Before fundraising or serious planning, build a proper TAM / SAM / SOM: TAM = annual used-phone transactions, SAM = transactions among online-savvy buyers in target cities, SOM = realistic year-one reach. Treat current numbers as directional only.

### 7.2 — Competitors
- **Divar, Sheypoor** — ad platforms only; no trust layer, no advice.
- **Resale stores (e.g. Nikdiji, Digipoya, StockMobile)** — traditional sellers, not advisory platforms.
- **ChatGPT / general models** — no native Iranian data; unreliable on price and registry.

### 7.3 — The gap
No current player offers a **smart, local, pre-purchase trust layer.** The Iranian buyer is alone at the moment of decision. That gap is the product.

---

## 8. Revenue Model

Monetization is staged to match trust, not to rush it. In **Phases 1 and 2** the product stays free, with revenue coming only from **clean, limited in-app advertising** that does not degrade the experience. Once usage and trust are established, HMR introduces a **message cap on the free tier plus an optional user subscription** that removes the cap and unlocks full access. In **Phase 3**, the primary revenue line becomes a **subscription to the personalized business chatbot**, billed per business. Each step is gated on the previous one proving out — we do not turn on a paywall before the free product has earned the right to charge.

---

## 9. Target Audience

- **Age:** 18–40, with a core of 18–29 (the segment that makes the bulk of online purchases).
- **Behavior:** searches Telegram groups and forums before buying, but the information is scattered, contradictory, and unreliable.
- **Core pain:** *"I'm about to spend a large sum and I don't know if I'm being scammed."*
- **Primary market:** metros first (Tehran, Isfahan, Mashhad, Tabriz, Shiraz), then smaller cities.

---

## 10. Risk Management

| Risk | Mitigation |
|---|---|
| High API cost | Token caps on free users; cache frequent answers; route to the cheapest viable provider on OpenRouter |
| Dependency on a single model/provider | Provider-independent abstraction layer — switch models/APIs without code changes |
| International internet disruption | Cache common answers and serve a read-only mode on the national network |
| Stale knowledge base | Fixed budget and owner for scheduled updates |
| Liability for wrong advice | Standing disclaimer ("based on official documents; the final decision is yours"); human referral on uncertain cases; price results explicitly disclaimed (§3.3) |
| Weak subscription-payment culture | Favor one-time / multi-month packages over recurring auto-debit |

---

## 11. Financial Outlook

A detailed cost model is deliberately omitted at this stage. Phase 1 is intentionally lean: a Linux VPS, domain and SSL, the manual-collection effort for three brands, the Astro site, initial OpenRouter credit, and standing up ChatREM and the file-management agent. Recurring costs are modest and dominated by VPS upkeep, API consumption, knowledge-base maintenance, and content marketing. Early revenue (limited in-app ads) is expected to be small and is not the point in Phases 1–2 — the goal is reach and trust. Meaningful revenue is planned from the subscription and B2B phases. **Action item:** once Phase 1 scope is locked, build a proper unit-economics model — cost per active user, ad RPM, free→paid conversion, and B2B ARPU — before committing to any revenue projection.

---

## 12. Roadmap & KPIs

### 6-month roadmap
| Month | Activity |
|---|---|
| 1 | Download ~50 popular Samsung manuals + build index + stand up VPS and Flowise |
| 2 | Add Xiaomi and Apple manuals + write role prompts + internal testing |
| 3 | Release free beta (web + Telegram) + start feedback collection + start marketing |
| 4 | Turn on limited in-app ads + begin Phase 2 (expand toward 10 brands) |
| 5 | Complete 10-brand KB + improve answer accuracy + add user memory |
| 6 | Begin Phase 3: pilot a personalized version with 2 law firms and 1 real-estate office |

### Key performance indicators
- First-response resolution rate — target **>75%**
- Average response time — target **<3s**
- Manuals uploaded — target **150** by end of Phase 2
- Free → paid conversion — target **8%** within the first 6 months (once subscription is live)
- **Weekly returning-user (retention) rate — target >40%** *(corrected: this should be high, not a bounce rate)*
- CSAT — target **>4.2 / 5**

---

*Open items before this goes external: (1) build TAM/SAM/SOM (§7.1); (2) build the unit-economics model (§11); (3) verify competitor store names and current OpenRouter pricing.*
