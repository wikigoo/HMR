# <N>. <Agent Title>

> **One-line mission:** <what this agent exists to do>
> Copy this file to start a new agent. Keep all 8 sections. Write in English.

---

## 1. Identity

| | |
|---|---|
| **Short title** | <e.g. "Web — Astro + Cloudflare"> |
| **Mission** | <one line> |
| **Owns (in-scope)** | <the work only this agent does> |
| **Does NOT own (hand off)** | <work → the agent it belongs to> |
| **HMR-tools skill** | <related Claude Code skill, if any> |

## 2. Task list

Recurring work this agent is responsible for:

- [ ] <task 1>
- [ ] <task 2>
- [ ] <task 3>

## 3. Toolbox

Exact tools, commands, configs, and credentials this agent uses:

| Tool / Command | Purpose |
|---|---|
| `<cmd>` | <what it does> |

## 4. Debug checklist

Run **in order** when something breaks. Stop at the first failing step and fix it.

1. <symptom> → <check> → <fix>
2. ...

## 5. Analysis & review checklist (quality gate)

Pass **before** writing the report:

- [ ] Goal achieved and matches the original request
- [ ] HMR global rules respected (Persian output, honesty boundary, live-price-only, advisory-not-decisional)
- [ ] No regression introduced (existing behaviour still works)
- [ ] Evidence captured (command output / URL / screenshot / test result)
- [ ] Knowledge base read for relevant facts; updated if any changed
- [ ] <agent-specific check>

## 6. Reporting protocol

After **every** session/action, append an entry to `Reports/REPORT-LOG.md` (newest on top) using `Reports/_session-report-template.md`. Do not close the task until the report is written and the Supervisor verdict is recorded.

## 7. Knowledge-base workflow

- **Read first:** consult `Knowledge/_INDEX.md` for HMR facts before acting — never assume.
- **Update on change:** if a fact changes, edit the relevant `Knowledge/` file, bump its "Last reviewed" date in `_INDEX.md`, and note it under "Knowledge-base updates" in the report.
- **Flag drift:** if repo facts disagree with reality, record it and notify the Supervisor.

## 8. Supervisor handoff

What the Supervisor re-checks for this agent (see `Supervisor Agent.md` → verification matrix):

- <check 1>
- <check 2>
