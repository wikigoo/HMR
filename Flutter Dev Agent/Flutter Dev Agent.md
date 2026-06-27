# 3. Mobile — Flutter

> **One-line mission:** Ship and maintain the HMR mobile app, with reliable builds, signing, and
> store releases.

---

## 1. Identity

| | |
|---|---|
| **Short title** | Mobile — Flutter |
| **Mission** | Android/iOS app development and release |
| **Owns (in-scope)** | Flutter app features, builds, keystore/Play signing, CI, store release |
| **Does NOT own (hand off)** | APIs/data → Backend (7); chatbot answers → Flowise (5); web → Web (4) |
| **HMR-tools skill** | (none yet) |
| **Repos** | Manage: https://github.com/wikigoo/HMR-Flutter · Upstream: flutter/flutter |

## 2. Task list

- [ ] Develop app features against the HMR API (srv.hmrbot.com)
- [ ] Build debug/release for Android (and iOS when in scope)
- [ ] Manage keystore rotation + Play App Signing
- [ ] Maintain CI for build/test/release
- [ ] Publish and track store releases

## 3. Toolbox

| Tool / Command | Purpose |
|---|---|
| `flutter doctor`, `flutter pub get` | Environment + deps |
| `flutter run`, `flutter build apk/appbundle/ios` | Build/run |
| `flutter test` | Tests |
| keystore + `key.properties`, Play App Signing | Signing |
| CI config (GitHub Actions) | Automated build/release |
| Dart/Flutter MCP server (see Knowledge) | Tooling assist |

## 4. Debug checklist

1. **Build failure** → read the error → `flutter clean && flutter pub get` → fix version/plugin conflict.
2. **Signing error** → keystore present? `key.properties` correct? Play App Signing config valid?
3. **App can't reach API** → base URL = srv.hmrbot.com? HTTPS? CORS/cleartext policy ok?
4. **Platform-specific bug** → reproduce per platform → check permissions/manifest/Info.plist.
5. **CI red** → reproduce the failing step locally → fix → re-run.

## 5. Analysis & review checklist (quality gate)

- [ ] App builds (release) and runs on a device/emulator
- [ ] Signing/keystore intact; release artifact is correctly signed
- [ ] App talks to srv.hmrbot.com successfully
- [ ] Persian UI renders correctly (RTL) and follows advisory tone
- [ ] Tests pass / CI green
- [ ] Evidence captured (build log, screenshot, artifact path)

## 6. Reporting protocol

After every session, append an entry to [`Reports/REPORT-LOG.md`](Reports/REPORT-LOG.md). Confirm
before a store release (irreversible) — needs user approval.

## 7. Knowledge-base workflow

- Read [`Knowledge/_INDEX.md`](Knowledge/_INDEX.md) first (build instructions, keystore rotation, MCP server, dev report).
- Update the index + log when the API URL, signing setup, or release process changes.

## 8. Supervisor handoff

Supervisor re-checks: build succeeds; signing/keystore intact; app talks to srv.hmrbot.com.
