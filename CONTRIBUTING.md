# Contributing & Workflow

How work flows through the HMR agent system, and how to extend it.

## Request lifecycle

1. **Intake (Supervisor).** A user/task arrives. The Supervisor classifies intent and picks the
   owning agent(s) using the routing table in [`Supervisor Agent.md`](Supervisor%20Agent.md).
2. **Execute (Domain agent).** The agent reads its `Knowledge/_INDEX.md`, does the work using its
   Toolbox, and runs its Debug checklist if anything breaks.
3. **Self-review (Domain agent).** The agent completes its section-5 review checklist — including the
   HMR global rules — and captures evidence.
4. **Report (Domain agent).** The agent appends an entry to its `Reports/REPORT-LOG.md` using
   `_TEMPLATE/_session-report-template.md`.
5. **Verify (Supervisor).** The Supervisor runs the verification matrix, records `Approved` or
   `Needs-rework`, and updates `Reports/SESSION-SUMMARY.md`.
6. **Learn.** Any new/changed HMR fact is written back into the relevant `Knowledge/` file.

## Adding a new agent

1. Copy [`_TEMPLATE/Agent-Template.md`](_TEMPLATE/Agent-Template.md) to `New Agent/New Agent.md`.
2. Create `New Agent/Knowledge/_INDEX.md` from `_TEMPLATE/Knowledge-_INDEX-template.md`.
3. Create `New Agent/Reports/REPORT-LOG.md` and copy `_TEMPLATE/_session-report-template.md` into it.
4. Fill all 8 sections. Pick the next free agent ID.
5. Add a row to the roster in [`AGENTS.md`](AGENTS.md) and a routing rule + verification row in the
   Supervisor manual.

## Conventions

- **Language:** English for all manuals/docs. (User-facing chatbot output is Persian — that's a
  product rule, not a docs rule.)
- **Folder names:** keep the current spaced names (e.g. `Backend & DB Agent`) to avoid breaking
  existing links. Do not rename without updating every link.
- **Knowledge index file:** always `_INDEX.md`. A `README.md` inside `Knowledge/` may be a vendored
  upstream readme — never overwrite it; the index is separate.
- **Reports are append-only:** newest entry on top; never delete history.
- **Single source of truth:** project-wide facts live in `AGENTS.md` quick reference; agent-specific
  facts live in that agent's `Knowledge/`.

## Definition of done (per task)

- [ ] Work complete and matches the request
- [ ] Review checklist passed (incl. HMR global rules)
- [ ] Evidence captured
- [ ] Report entry written
- [ ] Supervisor verdict recorded
- [ ] Knowledge base updated if any fact changed
