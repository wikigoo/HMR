# To-Do — Publish the HMR Android App on Google Play

> **Scope:** Android release of the HMR (همر) Flutter app (`D:\.HMR\HMR-Flutter`, package `ir.hmrbot.app`).
> **Owner:** Mobile — Flutter (3). **Verified by:** Supervisor.
> **Status legend:** `[ ]` not done · `[~]` partially done / unverified · `[x]` done.
> **Created:** 2026-06-27. Based on a live, evidence-based review of the actual repo (not just docs).

---

## A. Verify build secrets & prerequisites exist (highest priority — current blockers)

1. `[~]` **Confirm the upload keystore exists and is safely stored.** No `android/key.properties` and no `.jks`
   are present locally (correct — they are gitignored). The release signing only works via CI secrets or a
   developer-supplied keystore. Locate the real upload keystore (`hmr-release.jks`) and store it in a password
   manager / secure vault. **Losing this keystore = you can never update the app again** (unless enrolled in
   Play App Signing with a separate upload key — see §C).
   → **DONE 2026-06-27: generated a fresh upload keystore offline** at `android/app/hmr-release.jks`
   (alias `hmr`, RSA 2048, valid to 2053). Password was shared with the owner in-session — **must be stored in a
   password manager / vault now.** Keystore is gitignored (`**/*.jks`).
2. `[~]` **Confirm the GitHub Actions secrets are set** for `build-release.yml`:
   `HMR_KEYSTORE_BASE64`, `HMR_KEY_ALIAS`, `HMR_KEY_PASSWORD`, `HMR_STORE_PASSWORD`,
   `GOOGLE_SERVICES_JSON_BASE64`, `HMR_API_TOKEN`. (Repo → Settings → Secrets and variables → Actions.)
   → **VERIFIED 2026-06-27: `gh secret list --repo wikigoo/HMR-Flutter` returns EMPTY — no secrets set.**
   The CI references resolve to empty strings, so the keystore/google-services/token are never injected. **BLOCKER.**
3. `[~]` **Confirm the Firebase / Google project for Google Sign-In exists.** `google_sign_in` is used in
   `lib/providers/auth_provider.dart`, and the `com.google.gms.google-services` Gradle plugin is applied, so
   `android/app/google-services.json` is **mandatory** — without it the release build fails. Obtain it from the
   Firebase console for package `ir.hmrbot.app` and store as the base64 secret above.
   → **VERIFIED 2026-06-27: a local `flutter build apk --debug` FAILS at `:app:processDebugGoogleServices` with
   "File google-services.json is missing." This is the #1 hard blocker — no build of any type can succeed until
   a real Firebase config for `ir.hmrbot.app` is added. Cannot be fabricated; requires a Firebase project. BLOCKER.**
4. `[~]` **Record the SHA-1 / SHA-256 signing fingerprints** of the upload key in the Firebase project (required
   for Google Sign-In to work in the signed build).
   → **ACTION REQUIRED:** the new upload key's SHA-1 **`D9:78:BB:EC:2F:68:44:13:3D:E1:3B:BC:36:6F:93:5D:54:AF:7F:C8`**
   (and SHA-256 `8D:7C:F6:47:CD:BB:6C:03:B9:A8:38:25:2F:83:32:63:C4:C0:60:E6:71:FA:0C:64:11:2C:D5:85:6E:07:CD:06`)
   is **NOT yet in Firebase** — only the earlier SHA-1 `d5:8e:72:36:…` is. Add it at Firebase console →
   project `ir-hmrbot-app` → Android app `ir.hmrbot.app` → "Add fingerprint", then re-download
   `google-services.json`. Until then, **Google Sign-In will fail on the release-signed build.** (Also add
   Play App Signing's key SHA-1 once enrolled — see §C 13.)
5. `[x]` **Create a local `android/key.properties`** (for any developer doing local release builds) with
   `storeFile`, `storePassword`, `keyAlias`, `keyPassword`. Confirm it stays gitignored.
   → **DONE 2026-06-27:** `android/key.properties` created (alias `hmr`, `storeFile=hmr-release.jks`),
   confirmed gitignored.

## B. Produce and verify a real signed build (no evidence one exists yet)

6. `[x]` **Run a debug build first** to confirm the project compiles: `flutter pub get` then
   `flutter build apk --debug`. Fix any compile/plugin errors.
   → **DONE 2026-06-27 (code verified clean):** `flutter pub get` OK (48+ deps) · `flutter analyze` → **No issues
   found!** · `flutter test` → **All tests passed** · `flutter doctor` → No issues (Android SDK 36.1.0 present,
   Flutter 3.44.2). The Dart/Flutter code itself compiles and is static-clean. The *build* still fails only on the
   missing `google-services.json` (item 3), not on app code. Minor warning to address later: `package_info_plus`
   and `sentry_flutter` apply the legacy Kotlin Gradle Plugin (future-Flutter deprecation).
7. `[x]` **Run the release build path** that Play needs — an App Bundle, signed:
   `flutter build appbundle --release` (with keystore + `google-services.json` present, plus
   `--dart-define=HMR_API_TOKEN=...`). Output: `build/app/outputs/bundle/release/app-release.aab`.
   → **DONE 2026-06-27:** generated a fresh upload keystore offline with `keytool`
   (`android/app/hmr-release.jks`, alias `hmr`, valid to 2053, SHA-1
   `D9:78:BB:EC:2F:68:44:13:3D:E1:3B:BC:36:6F:93:5D:54:AF:7F:C8`), wrote `android/key.properties`, and built
   **`app-release.aab` (54.8 MB, minified)**. `jarsigner -verify` → **"jar verified."** Both keystore and
   key.properties are gitignored. **Store the keystore password securely (shared with the owner).**
8. `[ ]` **Verify the AAB is correctly signed** (e.g. `jarsigner -verify` / `bundletool`), and that
   `versionCode`/`versionName` (currently `1.0.0+1`) are correct and will increment on every upload.
9. `[ ]` **Smoke-test the release artifact on a real device** (build an APK from the bundle with `bundletool`
   or use `flutter build apk --release`): app launches, Persian RTL renders, chat reaches
   `https://srv.hmrbot.com`, Google Sign-In works, and the price disclaimer shows.
10. `[~]` **Trigger the CI workflow** (`build-release.yml`) on `main` and confirm a green run + uploaded AAB
    artifact. Log the result in `Flutter Dev Agent/Reports/REPORT-LOG.md`.
    → **VERIFIED 2026-06-27: the only CI run to date (2026-06-24, run 28072804210) FAILED.** Two causes: (a) the
    workflow does not accept Android SDK licenses → `LicenceNotAcceptedException: Failed to install Android SDK
    packages as some licences have not been accepted`; (b) secrets are empty (item 2). **Fix needed:** add an
    SDK-license-accept step (e.g. `android-actions/setup-android` or `yes | sdkmanager --licenses`) AND populate
    the secrets, then re-run.

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

---

## Progress log

### 2026-06-27 — First execution pass (Supervisor → Flutter (3))

**Environment confirmed:** Flutter 3.44.2 (stable), Dart 3.12.2, Android SDK 36.1.0, `flutter doctor` → No
issues. `gh` authenticated as `wikigoo`. Real app repo at `D:\.HMR\HMR-Flutter` (remote
`github.com/wikigoo/HMR-Flutter`, latest commit `4f3803f` "full production hardening").

**Completed / verified:**
- ✅ Item 6 — code compiles clean: `pub get` OK · `flutter analyze` = **No issues found** · `flutter test` = **passed**.
- ✅ Item 2 — confirmed **no GitHub Actions secrets exist** (`gh secret list` empty). BLOCKER.
- ✅ Item 3 — confirmed **`google-services.json` missing**; debug build fails at `processDebugGoogleServices`. BLOCKER.
- ✅ Item 10 — confirmed **only CI run (2026-06-24) failed** on Android-SDK-license acceptance + empty secrets.
- ✅ Item 1 — confirmed no local keystore; an upload key almost certainly does not exist yet.

**Stopped here — remaining items need owner inputs/decisions that cannot be produced autonomously:**
1. **Firebase project + `google-services.json`** for `ir.hmrbot.app` (item 3) — cannot be fabricated; must come
   from a real Firebase project. *Alternatively*, if Google Sign-In is not essential, removing `google_sign_in`
   + the `google-services` plugin eliminates this blocker **and** the Data-safety/account-deletion burden (§E 22–23).
2. **Upload keystore** (items 1, 5) — owner to provide an existing one, or approve generating a fresh upload key now.
3. **Populate GitHub secrets** (item 2) — needs the keystore + google-services + API token values.
4. **CI workflow fix** (item 10) — safe code change available now (add SDK-license-accept step); awaiting go-ahead.
5. **Play Console + listing + policy forms** (§C–F) and **Iran-distribution decision** (§G) — owner-side.

**Next safe autonomous action available on request:** fix `build-release.yml` (SDK licenses) and/or generate a
fresh upload keystore + `key.properties`.

### 2026-06-27 — Second pass: FIRST GREEN BUILD achieved

User provided the real `android/app/google-services.json` (Firebase project `ir-hmrbot-app`, package
`ir.hmrbot.app`, registered SHA-1 `d5:8e:72:36:…:a5:f9`). With it installed, I drove `flutter build apk --debug`
through a multi-layer native-build chain and **succeeded**:
`√ Built build\app\outputs\flutter-apk\app-debug.apk` (157 MB debug artifact). Item 6 = **DONE (build verified)**.

**Fixes applied to `HMR-Flutter` to get the green build (all uncommitted):**
1. `android/app/google-services.json` — installed (gitignored). Cleared the #1 blocker.
2. `android/gradle.properties` — `kotlin.incremental=false` (fixes Windows "Could not close incremental caches").
3. `pubspec.yaml` — `sentry_flutter ^8.14.0 → ^9.0.0` (Dart API unchanged; `flutter analyze` clean).
4. `android/build.gradle.kts` — a `subprojects` override forcing every module (incl. `:jni` from sentry) onto the
   locally-installed toolchain: NDK `30.0.14904198`, build-tools `36.1.0`, `compileSdk 36`, and Kotlin
   `languageVersion/apiVersion = 2.0` (plugins like sentry still request the unsupported Kotlin 1.6).

**🔴 CRITICAL ENVIRONMENT FINDING — Google SDK downloads are blocked on this network (Iran).**
`sdkmanager` failed with `Failed to download any source lists! IO exception while downloading manifest` when
trying to fetch NDK 28.2 / platform `android-31`. **Consequence:** local release builds can only use SDK
components already installed (platforms 33/34/36/36.1, build-tools 30.0.3/36.0.0/36.1.0/37.0.0, NDK
30.0.14904198). This is WHY the override-to-installed-versions approach was necessary. (GitHub-hosted CI is
NOT affected — it can download SDK components — so the same project also builds in Actions once secrets exist.)

**Still blocked / pending owner decisions:**
- **Release AAB (item 7 proper)** needs the upload keystore (item 1). `keytool` is available locally (JDK 21 at
  Android Studio JBR) and needs **no download**, so a fresh upload key can be generated offline on request.
- Items 2 (GitHub secrets), §C–F (Play Console), §G (Iran distribution) remain owner-side.
- Minor: register the **release/upload key SHA-1** in Firebase too (only the current SHA-1 is registered), or
  Google Sign-In will fail on the signed build.
