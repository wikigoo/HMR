# To-Do — Publish the HMR Android App on Cafe Bazaar (کافه‌بازار)

> **Scope:** Cafe Bazaar release of the HMR (همر) Flutter app (`D:\.HMR\HMR-Flutter`, package `ir.hmrbot.app`).
> Companion to `To-do-for-publish-HMR-app-in-googleplay.md`. Cafe Bazaar is the dominant Iranian Android store,
> the recommended **primary** channel for an Iran-targeted app (Google Play has sanction friction — see Play
> checklist §G). **Myket** is a good secondary channel with near-identical steps.
> **Status legend:** `[ ]` not done · `[~]` partial/unverified · `[x]` done. **Created:** 2026-06-27.
> Owner: Mobile — Flutter (3) for build/signing; Growth & Marketing (8) for listing/strategy.

---

## A. Pre-flight — what's already done vs. Bazaar-specific gaps

1. `[x]` **A signed release artifact already exists.** `D:\HMR-release.apk` (≈57 MB, signed with the
   `hmr-release.jks` upload key, built with the real `google-services.json`). Cafe Bazaar accepts a **signed
   APK directly** — it does NOT re-sign like Google Play App Signing, so this APK is the one Bazaar will ship.
   Keep using the **same keystore** for every future update (a different key = Bazaar rejects the update).
2. `[x]` **App basics verified:** `applicationId ir.hmrbot.app` · `versionName/Code 1.0.0+1` · `targetSdk 36` ·
   Persian RTL UI · HTTPS API (`srv.hmrbot.com`) · only the `INTERNET` permission · price disclaimer present.
3. `[ ]` **Bump `versionCode` for every Bazaar upload.** Bazaar (like Play) rejects re-uploads with the same
   `versionCode`. If you also ship to Play, keep version numbers coordinated to avoid confusion.

## B. 🔴 Critical compatibility decision — Google Sign-In vs. Google Play Services on Iranian devices

4. `[ ]` **Resolve the Google Sign-In ↔ GMS dependency before launch.** The app uses `google_sign_in`
   (`lib/providers/auth_provider.dart`), which **requires Google Play Services (GMS) on the device**. Many
   Iranian Android phones (and most users who installed via Bazaar) **do not have working GMS**, so Google
   Sign-In will **fail for a large share of the Bazaar audience**. Options:
   - **(Recommended) Make login optional / add a non-Google path** (guest mode, phone-OTP, or email) so the app
     is fully usable without GMS. The chat already works without auth on the conversations screen.
   - **Or gate Google Sign-In behind a GMS-availability check** and hide it when GMS is absent.
   - **Or accept the limitation** and document that login is GMS-only. ⚠️ Not recommended for the Iran market.
   This is the single biggest Bazaar-specific risk and should be decided with Growth (8) + Backend (7).
5. `[ ]` **Do NOT use Google Play Billing.** If HMR ever monetizes, Bazaar requires **Bazaar's own in-app
   billing (Poolak/Bazaar IAB)** — Google Play Billing is not allowed and won't work on most Iranian devices.
   (Currently the app is free with no IAP — no action needed unless that changes.)

## C. Developer account (Bazaar Pishkhan)

6. `[ ]` **Create / sign in to a Cafe Bazaar developer account** at the developer panel
   (پیشخان توسعه‌دهندگان — `pishkhan.cafebazaar.ir`). Registration is **free** and straightforward for
   Iranian developers (no US$25 fee, no sanction barrier — the key advantage over Google Play).
7. `[ ]` **Complete identity verification** (احراز هویت) — Iranian national ID / the panel's required
   verification. Needed before an app can go public.
8. `[ ]` **Set up the developer profile** (display name, contact email `wikigoo58@gmail.com`, support info).

## D. Create the app & upload the build

9. `[ ]` **Create a new app** in Pishkhan with package `ir.hmrbot.app`.
10. `[ ]` **Upload the signed APK** (`D:\HMR-release.apk`). Bazaar also accepts Android App Bundle, but the
    signed APK is the simplest path and is already built.
11. `[ ]` **Confirm Bazaar's automated checks pass** (signature valid, manifest, target SDK, permissions). Fix
    any flagged issue and re-upload with an incremented `versionCode`.

## E. Store listing (Persian — required)

12. `[ ]` **App title (Persian):** `همر` or `همر | مشاور سخت‌افزار موبایل` (confirm length limits in Pishkhan).
13. `[ ]` **Short description (Persian):**
    > مشاور هوشمند خرید و عیب‌یابی موبایل؛ راهنمای صادق برای بازار ایران.
14. `[ ]` **Full description (Persian)** — reuse the body drafted in `Play-Store-Listing-Assets.md` §3 (five
    pillars + advisory positioning + price disclaimer). Adapt tone for the Iranian-store audience as needed.
15. `[ ]` **Category & tags:** e.g. *خرید* (Shopping) or *ابزارها* (Tools); tags: موبایل، راهنمای خرید، سخت‌افزار.
16. `[ ]` **Graphic assets:**
    - App **icon** (Bazaar spec — typically 512×512 PNG).
    - **Screenshots** (≥3 recommended) captured from the signed build — chat screen, conversations list,
      disclaimer. (The Play emulator screenshot from item 9 can be reused.)
    - **Feature/cover graphic** if Bazaar requests one for featuring.
17. `[ ]` **Changelog / release notes (Persian)** for v1.0.0.

## F. Policy, rating & compliance (Iran-specific)

18. `[ ]` **Age rating / content rating** — complete Bazaar's questionnaire (expected: عمومی / all ages; no
    mature content).
19. `[ ]` **Permissions justification** — only `INTERNET` is requested; declare why (chat with the backend).
20. `[ ]` **Privacy policy** — point to a live Persian privacy page (the app references
    `https://hmrbot.com/privacy`; confirm it is live and covers data collected, the backend AI processing,
    and deletion). Account-deletion path is also expected if login is kept (see §B 4).
21. `[ ]` **Iranian content compliance** — content must comply with Iran's laws/regulations (no prohibited
    content). The advisory, hardware-focused scope is low-risk; the price disclaimer helps.
22. `[ ]` **Honesty / claims** — no fabricated specs or prices in listing or app (HMR global rule); the
    live-price disclaimer must remain visible.

## G. Submit, review & release

23. `[ ]` **Submit for review.** Bazaar runs a human review (functionality + policy). Typical turnaround is a
    few hours to a couple of days.
24. `[ ]` **Respond to any review feedback**, fix, bump `versionCode`, re-submit.
25. `[ ]` **Publish to production** once approved. Consider a soft launch / limited announcement first.
26. `[ ]` **Post-launch monitoring** — watch Bazaar reviews, crash reports (the app ships Sentry; wire a DSN if
    you want crash telemetry), and ratings; plan a quick hotfix path (increment `versionCode`, rebuild, re-upload).

## H. Optional — Bazaar SDK & multi-store hygiene

27. `[ ]` **(Optional) Integrate the Bazaar SDK** for in-app update prompts / review prompts, if desired.
28. `[ ]` **Multi-store coordination** — if also shipping to Google Play and/or Myket:
    - Keep **one source of truth for `versionCode`/`versionName`** across stores.
    - Remember the **signature difference**: the Bazaar build is signed with your `hmr-release.jks`; a Google
      Play build re-signed by Play App Signing has a different key, so a user can't cross-update between stores
      (each installs/updates from one store). Register the relevant SHA-1s in Firebase per channel.
    - Myket setup mirrors this checklist almost exactly.

---

## Quick comparison — Cafe Bazaar vs. Google Play (for HMR)

| Topic | Cafe Bazaar | Google Play |
|---|---|---|
| Iran reach | ✅ Dominant local store | ⚠️ Restricted for IR users |
| Dev account | Free, Iranian ID | US$25 + sanction friction |
| Artifact | Signed APK (or AAB) | AAB only |
| Signing | Your key ships as-is | Re-signed via Play App Signing |
| Google Sign-In | ⚠️ Needs GMS — often missing on IR devices (§B 4) | Works (GMS present) |
| Billing (if any) | Bazaar IAB only | Google Play Billing |
| Best role for HMR | **Primary channel** | Secondary (diaspora) |

**Recommendation:** ship to **Cafe Bazaar first** (lowest friction, the real target market), with **Myket** as a
second channel, and treat Google Play as optional. **Before any Iranian-store launch, resolve the Google
Sign-In / GMS dependency (§B 4)** — it is the highest-impact item on this list.
