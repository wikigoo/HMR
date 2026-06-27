# To-Do — Publish the HMR Android App on Google Play

> **Scope:** Android release of the HMR (همر) Flutter app (`D:\.HMR\HMR-Flutter`, package `ir.hmrbot.app`).
> **Owner:** Mobile — Flutter (3). **Verified by:** Supervisor.
> **Status legend:** `[ ]` not done · `[~]` partially done / unverified · `[x]` done.
> **Created:** 2026-06-27. Based on a live, evidence-based review of the actual repo (not just docs).

---

## A. Verify build secrets & prerequisites exist (highest priority — current blockers)

1. `[ ]` **Confirm the upload keystore exists and is safely stored.** No `android/key.properties` and no `.jks`
   are present locally (correct — they are gitignored). The release signing only works via CI secrets or a
   developer-supplied keystore. Locate the real upload keystore (`hmr-release.jks`) and store it in a password
   manager / secure vault. **Losing this keystore = you can never update the app again** (unless enrolled in
   Play App Signing with a separate upload key — see §C).
2. `[ ]` **Confirm the GitHub Actions secrets are set** for `build-release.yml`:
   `HMR_KEYSTORE_BASE64`, `HMR_KEY_ALIAS`, `HMR_KEY_PASSWORD`, `HMR_STORE_PASSWORD`,
   `GOOGLE_SERVICES_JSON_BASE64`, `HMR_API_TOKEN`. (Repo → Settings → Secrets and variables → Actions.)
3. `[ ]` **Confirm the Firebase / Google project for Google Sign-In exists.** `google_sign_in` is used in
   `lib/providers/auth_provider.dart`, and the `com.google.gms.google-services` Gradle plugin is applied, so
   `android/app/google-services.json` is **mandatory** — without it the release build fails. Obtain it from the
   Firebase console for package `ir.hmrbot.app` and store as the base64 secret above.
4. `[ ]` **Record the SHA-1 / SHA-256 signing fingerprints** of the upload key in the Firebase project (required
   for Google Sign-In to work in the signed build).
5. `[ ]` **Create a local `android/key.properties`** (for any developer doing local release builds) with
   `storeFile`, `storePassword`, `keyAlias`, `keyPassword`. Confirm it stays gitignored.

## B. Produce and verify a real signed build (no evidence one exists yet)

6. `[ ]` **Run a debug build first** to confirm the project compiles: `flutter pub get` then
   `flutter build apk --debug`. Fix any compile/plugin errors.
7. `[ ]` **Run the release build path** that Play needs — an App Bundle, signed:
   `flutter build appbundle --release` (with keystore + `google-services.json` present, plus
   `--dart-define=HMR_API_TOKEN=...`). Output: `build/app/outputs/bundle/release/app-release.aab`.
8. `[ ]` **Verify the AAB is correctly signed** (e.g. `jarsigner -verify` / `bundletool`), and that
   `versionCode`/`versionName` (currently `1.0.0+1`) are correct and will increment on every upload.
9. `[ ]` **Smoke-test the release artifact on a real device** (build an APK from the bundle with `bundletool`
   or use `flutter build apk --release`): app launches, Persian RTL renders, chat reaches
   `https://srv.hmrbot.com`, Google Sign-In works, and the price disclaimer shows.
10. `[ ]` **Trigger the CI workflow** (`build-release.yml`) on `main` and confirm a green run + uploaded AAB
    artifact. Log the result in `Flutter Dev Agent/Reports/REPORT-LOG.md`.

## C. Google Play Console — account, app, and signing setup

11. `[ ]` **Have a Google Play Developer account** (one-time US$25). ⚠️ See §G for the Iran-distribution caveat
    before paying — this may be a strategic blocker.
12. `[ ]` **Create the app** in Play Console: app name, default language (Persian — `fa-IR`), app/game = App,
    free/paid = Free.
13. `[ ]` **Enroll in Play App Signing** and upload the upload key. This lets Google manage the app signing key
    and protects you if the upload key is ever lost.
14. `[ ]` **Reserve the package name** `ir.hmrbot.app` by uploading the first AAB to an internal testing track.

## D. Store listing assets (all required before review)

15. `[ ]` **App icon** — 512 × 512 PNG (32-bit, with alpha).
16. `[ ]` **Feature graphic** — 1024 × 500 PNG/JPG.
17. `[ ]` **Phone screenshots** — at least 2 (recommended 4–8), 16:9 or 9:16, min 320px side. Capture from the
    real app (chat screen, conversations screen, sign-in, disclaimer).
18. `[ ]` **Short description** (≤ 80 chars, Persian) and **full description** (≤ 4000 chars, Persian).
19. `[ ]` **(Optional) Promo video** (YouTube URL) and 7-inch / 10-inch tablet screenshots if tablet support is
    claimed.
20. `[ ]` **Categorization**: application category (e.g. *Shopping* or *Tools*), tags, and contact details
    (email, optional phone/website).

## E. Policy & compliance forms (mandatory — app will be rejected without them)

21. `[ ]` **Privacy Policy URL** — must be publicly live. The app references `https://hmrbot.com/privacy`;
    confirm that page actually exists and accurately describes data handling.
22. `[ ]` **Data safety form** — declare data collected/shared. The app collects **Google account identity**
    (Google Sign-In) and **chat content** sent to the backend; declare these, their purpose, encryption in
    transit (HTTPS ✅), and whether the user can request deletion.
23. `[ ]` **Account deletion mechanism** — because the app supports account login (Google Sign-In), Play
    **requires** an in-app and/or web path for users to delete their account and associated data. Implement and
    document the URL.
24. `[ ]` **Content rating questionnaire** — complete it to receive an IARC rating.
25. `[ ]` **Target audience & content** — set age groups; if not targeting children, declare so.
26. `[ ]` **Ads declaration** — declare whether the app contains ads (currently appears to be **no ads**).
27. `[ ]` **Government/financial features** — N/A, but confirm the price-advice disclaimer is present (it is:
    the amber disclaimer in `chat_screen`) so the app is clearly **advisory, not transactional**.
28. `[ ]` **Permissions justification** — only `INTERNET` is requested (good); no sensitive permissions to
    justify.

## F. Release & rollout

29. `[ ]` **Upload the AAB to Internal testing**, add testers, and verify install + core flows on real devices.
30. `[ ]` **Promote to Closed/Open testing** (optional) to gather feedback before production.
31. `[ ]` **Complete the "Production" release** with release notes (Persian), then submit for review.
32. `[ ]` **Use a staged rollout** (e.g. 10% → 50% → 100%) and monitor crashes/ANRs in Play Console
    (Android vitals).
33. `[ ]` **Document the rollback/hotfix plan** (increment versionCode, rebuild, re-upload) in the Flutter
    agent's Knowledge base.

## G. ⚠️ Strategic blocker — Iran distribution on Google Play

34. `[ ]` **Decide the distribution channel before investing in Play.** HMR targets the Iranian market, but
    Google Play has **sanction-related restrictions** affecting both developer-account registration and
    distribution/visibility to users in Iran. This can make Google Play impractical as the *primary* channel.
35. `[ ]` **Evaluate Iranian app stores as the primary/secondary channel** — e.g. **Cafe Bazaar** and **Myket**
    — which dominate the Iranian market and accept Flutter AABs/APKs. Route this decision through the
    **Growth & Marketing (8)** agent.
36. `[ ]` **If keeping Google Play**, confirm a viable developer-account path and that target users can actually
    install the app, before spending effort on store-listing work above.

---

## Quick status snapshot (as of 2026-06-27)

**Already in good shape (verified in code):** `applicationId ir.hmrbot.app` · `targetSdk/compileSdk 36` ·
version `1.0.0+1` · release minify/shrink/proguard · HTTPS API (`srv.hmrbot.com`) · real chatflow ID ·
clean manifest (only `INTERNET`, `allowBackup=false`) · custom launcher icon + splash · signed-AAB CI workflow ·
secrets correctly gitignored.

**Hard blockers right now:** keystore + `google-services.json` existence unconfirmed (items 1–3) ·
no evidence of a successful signed build (items 6–10) · all Play Console forms/listing pending (§C–F) ·
Iran-distribution decision unmade (§G).

**Supervisor verdict:** `Needs-work — not ready to publish`. The build/signing foundation is solid; publishing
is gated on verifying secrets, producing one green signed build, completing Console requirements, and resolving
the Iran-distribution question.
