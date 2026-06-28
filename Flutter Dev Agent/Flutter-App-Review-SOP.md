# Flutter App Review — SOP (Flutter Dev Agent, agent 3)

> **Purpose:** A single, repeatable procedure for an in-depth, evidence-based review of the **HMR
> Android app** (Flutter) — architecture, code, UI/UX, localization, security, performance, and Google
> Play readiness. The output is a structured report of strengths, blockers, and prioritized
> improvements.
>
> **Owner:** Flutter Dev Agent (3), routed by the Supervisor (1). **Mode:** **READ-ONLY analysis** — this
> SOP inspects and reports; it does **not** push code changes. Proposed fixes are written up and handed
> back to the owner for a separate implementation task.
>
> **Honesty boundary:** review only code/config you can actually see. If a file isn't available, mark
> the related check **`N/A — not provided`** and request it; never infer findings about unseen code.

---

## 1. Scope

- **In scope:** the HMR Flutter mobile **client** (Android primary), repo `github.com/wikigoo/HMR-Flutter`.
- **Out of scope:** Flowise backend (→ agent 5/6) and the Astro web frontend (→ agent 4), **except** the
  app↔backend API contract with `srv.hmrbot.com`, which is in scope here.
- **iOS:** Android is primary (the live target is Google Play). Where a check has an iOS analogue, note
  parity as **optional**; do not block on iOS unless the user asks.

## 2. Obtain the code (read-only)

1. **Preferred — live repo:** read `github.com/wikigoo/HMR-Flutter` (branch `main`). Use raw files
   `raw.githubusercontent.com/wikigoo/HMR-Flutter/main/<path>` and the contents API
   `api.github.com/repos/wikigoo/HMR-Flutter/contents/<path>` to list folders. If the repo is private
   or unreachable, **say so** and ask the user to upload the files.
2. **Fallback — uploaded files.** Minimum set to do a meaningful review:
   `pubspec.yaml`, the `lib/` tree, API/service layer, state-management files, `android/app/build.gradle`,
   `android/app/src/main/AndroidManifest.xml`, and any `l10n`/`intl`/ARB assets.
3. **Optional static-analysis pass** (if a Dart/Flutter toolchain or the Dart MCP is available — see the
   Flutter agent's Knowledge index): run read-only `dart analyze`, `dart format --output=none --set-exit-if-changed .`,
   `flutter pub outdated`, `flutter pub deps`. Capture output as evidence. Do **not** modify files.

## 3. Severity scale (tag every finding)

| Tag | Meaning |
|---|---|
| 🟥 **Blocker** | Must fix before release — crash, data loss, security hole, broken core flow, Play-policy fail |
| 🟧 **Major** | Significant defect or risk — should fix soon; degrades UX, performance, or maintainability |
| 🟨 **Minor** | Real but low-impact; fix when convenient |
| ⬜ **Nit** | Style/polish/preference |

Every finding must carry: severity + a **file:line (or file) reference** + a one-line rationale.

---

## 4. Evaluation steps

Run sections 4.1 → 4.10 in order. For each, record a status (✅ OK / ⚠️ Concerns / ❌ Problem / N/A) plus
findings with severity + evidence.

### 4.1 Architecture & project structure
- Is there a clear, scalable pattern (Clean Architecture / MVVM / feature-first)? Layer separation
  between **UI · business logic · data/services**?
- Folder structure under `lib/` consistent (e.g. `features/`, `core/`, `data/`, `presentation/`)?
- Any god-files, circular deps, or UI widgets calling the network directly?

### 4.2 State management
- Which solution (Provider / Riverpod / Bloc / GetX)? Used consistently?
- Correct lifecycle: controllers/notifiers disposed; no rebuild storms; no **memory leaks** (unclosed
  streams/controllers, listeners not removed).
- Is async state modeled with explicit **loading / data / empty / error** states (not just a spinner)?

### 4.3 Code quality, lints & dependencies
- `analysis_options.yaml` present with a real lint set (e.g. `flutter_lints`/`very_good_analysis`)?
  `dart analyze` clean (record warning/error counts)?
- `pubspec.yaml`: versions pinned sensibly; no **deprecated/abandoned** packages; no obvious bloat or
  duplicate-purpose packages. Run `flutter pub outdated` — list majorly outdated and unmaintained deps.
- Dead code, `print()` left in, `TODO/FIXME` density, package licenses acceptable for distribution.

### 4.4 Backend / API integration (`srv.hmrbot.com`)
- Single source of truth for the **base URL** = `srv.hmrbot.com` (no stray/legacy hosts hardcoded in
  multiple places; use a config/env, not literals scattered across files).
- Requests go through the **token-hiding proxy** (`/predict`, header `x-app-key`) — the OpenRouter / any
  provider key must **never** be embedded in the app binary. Confirm no secrets in source or assets.
- **Error handling:** timeouts, 4xx/5xx, malformed JSON, and **no-internet** all handled with a user
  message + retry path (no silent failures, no raw exceptions on screen).
- Retry/backoff and request cancellation on screen dispose; reasonable timeout values.

### 4.5 Session & chat-history management
- How is the chatflow/session ID created, stored, and reused (to keep conversation context)?
- Where is history persisted (memory only vs local DB/secure storage)? Survives app restart as intended?
- PII in history handled appropriately; clear/delete-history path exists if required.

### 4.6 Localization & RTL (Persian-first)
- **RTL:** layouts, text alignment, icons (directional), and animations fully respect right-to-left.
- **Strings:** externalized via `intl`/ARB (`l10n`) — flag hardcoded Persian strings in widgets.
- **Typography:** a proper Persian font bundled (e.g. Vazirmatn/IRANSans); line-height/letter-spacing
  tuned for chat readability; text scales with system font-size setting.
- **Numerals & dates:** Persian (Eastern Arabic) digits where expected; Jalali calendar/date formatting
  if dates are shown.

### 4.7 UI/UX & product-pillar coverage
- The five HMR domains — **Buying Guide · Fraud Detection · Hardware Troubleshooting · Training ·
  Accessories** — are discoverable/navigable (whether as entry chips, menu, or guided prompts in the
  chat). No dead ends.
- Consistent design system (spacing, colors, components); empty/loading/error states designed, not raw.
- **Accessibility:** sufficient contrast, tap-target sizes, `Semantics` labels, and correct behavior at
  large font scales (no clipped/overflowing Persian text).

### 4.8 Chat rendering & performance
- Long conversations use `ListView.builder` (lazy) — confirm no full-list rebuilds; smooth scroll (no
  jank). Streaming/token-by-token updates don't rebuild the whole list.
- `const` constructors where possible; rebuild scope minimized; images cached
  (`cached_network_image` or equivalent) and sized.
- Startup time, frame drops on scroll, and memory growth over a long session are acceptable (note method
  used: DevTools / profile build).

### 4.9 Security
- **Secrets:** no API keys, tokens, or the keystore password in source, assets, or VCS history.
- **Secure storage:** tokens/sensitive data in `flutter_secure_storage` (not `SharedPreferences`).
- **Signing/keystore:** review `key.properties` handling (presence/location only — never print values).
  ⚠️ **Known issue:** the project's keystore password was exposed (`HmrBot2025!`) with a rotation +
  Play App Signing plan documented in `Knowledge/Keystore Rotation + Play App Signing + CI.md` — confirm
  that rotation is completed and Play App Signing is enabled; treat an un-rotated key as a **🟥 Blocker**.
- **Manifest:** only necessary permissions (`INTERNET`); `usesCleartextTraffic` disabled (HTTPS only);
  no unintended `exported=true` components; deep links validated.

### 4.10 Build, release config & Google Play readiness
- `android/app/build.gradle`: correct `applicationId`, sane `minSdk`/`targetSdk` (meets current Play
  target-API requirement), and `versionCode`/`versionName` bump strategy.
- Release build uses **R8/ProGuard** (shrink + obfuscate); confirm no missing keep-rules break reflection.
- App size reasonable (`flutter build appbundle --analyze-size`); **AAB** (not APK) for Play.
- **Tests/CI:** any unit/widget/integration tests? CI building the signed AAB (see keystore/CI doc)?
- **Play compliance:** Data Safety form matches actual data use; privacy policy linked; permissions
  justified. (Ties to `To do/To-do-for-publish-HMR-app-in-googleplay.md`.)

---

## 5. Report format (produce exactly this)

```markdown
## <YYYY-MM-DD> — HMR Flutter App Review

- **Agent:** Flutter Dev (3) · **Mode:** read-only · **Source:** wikigoo/HMR-Flutter @ <commit/branch>
- **Scope covered:** <sections run> · **Not reviewed:** <N/A items + why>

### 1. Executive summary
<one paragraph: overall health, release-readiness verdict (Ready / Ready-with-fixes / Not ready)>

### 2. Strengths
- <≥3 genuinely good implementations, with file references>

### 3. Critical issues (🟥 Blockers — fix before release)
| # | Issue | Evidence (file:line) | Why it blocks | Suggested fix |
|---|-------|----------------------|---------------|---------------|

### 4. Suggested improvements (🟧/🟨/⬜)
- **Code & performance:** <findings + severity + refs>
- **UX / UI (incl. RTL & localization):** <…>
- **Backend communication (`srv.hmrbot.com`):** <…>

### 5. Area scorecard
| Area | Status | Top finding |
|------|--------|-------------|
| Architecture | ✅/⚠️/❌ | |
| State mgmt | | |
| Code quality/deps | | |
| API integration | | |
| Sessions/history | | |
| Localization/RTL | | |
| UI/UX & pillars | | |
| Performance | | |
| Security | | |
| Build/Play readiness | | |

### 6. Deployment checklist (Android / Google Play)
- [ ] Keystore rotated + **Play App Signing** enabled (no exposed key) — see keystore/CI doc
- [ ] `versionCode`/`versionName` bumped; `targetSdk` meets current Play requirement
- [ ] Release build: R8/shrink + obfuscation on; keep-rules verified
- [ ] Signed **AAB** built (ideally via CI); size checked
- [ ] HTTPS-only; cleartext disabled; only required permissions
- [ ] Data Safety form + privacy policy match actual behavior
- [ ] Smoke test on a real device: chat reaches `srv.hmrbot.com`, RTL/fonts correct, all 5 pillars reachable
- [ ] Open items from `To do/To-do-for-publish-HMR-app-in-googleplay.md` closed

### 7. Supervisor verdict
<Approved | Needs-rework: reason | Pending>
```

---

## 6. Quality gate (before submitting)

- [ ] Every section 4.1–4.10 has a status; unseen areas marked `N/A` with the reason.
- [ ] Every finding has **severity + file reference + rationale** (no vague claims).
- [ ] No secret value is reproduced anywhere (presence/location only).
- [ ] Release verdict matches the worst Blocker (any open 🟥 ⇒ not "Ready").
- [ ] Report appended to `Reports/REPORT-LOG.md`; Supervisor verification requested; Knowledge base
      updated if any durable fact (deps, structure, base URL) changed.
