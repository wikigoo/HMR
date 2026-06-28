# Plan — Remove the API token from the mobile binary (token hardening)

> **Problem (verified in code):** `lib/services/api_service.dart` sends
> `Authorization: Bearer <HMR_API_TOKEN>` where `HMR_API_TOKEN` is injected at build time via
> `--dart-define`. A `--dart-define` value is compiled into the binary as a string constant, so anyone who
> reverse-engineers the `.apk`/`.aab` can extract it. The token then lets an attacker call the Flowise
> prediction API directly (cost/abuse, and a Google Play "client security" red flag).
>
> **Note on current state:** the APKs built in this session were built **without** `--dart-define=HMR_API_TOKEN`,
> so the shipped binary currently carries an **empty** token. The risk becomes real the moment a real token is
> baked in. Fix the architecture before that happens.
>
> **Owners:** Backend & Data (7) + DevOps (6) own the proxy; Mobile — Flutter (3) owns the client change;
> Web/Cloudflare (4) if the proxy is a Worker. Supervisor coordinates.

---

## Core idea

The mobile client should **never hold the Flowise token**. Put a thin server-side proxy between the app and
Flowise that holds the token, and protect that proxy from abuse with **app attestation** instead of a shared
secret.

```
BEFORE:  app ──(Bearer <token baked in app>)──▶ srv.hmrbot.com (Flowise)
AFTER:   app ──(Firebase App Check / Turnstile token)──▶ proxy ──(Bearer <token, server-side secret>)──▶ Flowise
```

## Recommended approach — Cloudflare Worker proxy + Firebase App Check

The stack already uses **Cloudflare** (Astro frontend on Workers) and **Firebase** (Google Sign-In), so reuse both.

### Server side (Backend/DevOps/Web)
1. Create a Cloudflare Worker (e.g. route `https://api.hmrbot.com/chat`) — or add a route to the existing
   Workers app.
2. Store the Flowise token as a **Worker secret** (`wrangler secret put FLOWISE_TOKEN`) — never in the client,
   never in git.
3. The Worker:
   - verifies an inbound **Firebase App Check** token (Play Integrity-backed) — reject if missing/invalid;
   - optionally rate-limits per IP / per App Check app id;
   - forwards the request to `https://srv.hmrbot.com/api/v1/prediction/<chatflowId>` with
     `Authorization: Bearer <FLOWISE_TOKEN>` added server-side;
   - streams/returns the Flowise response back to the app.
4. Lock down Flowise so it only accepts traffic carrying the token (already the case) and, ideally, only from
   the Worker/origin.

### Client side (Flutter)
5. **Delete** `_apiToken` / the `Authorization: Bearer` header and the `--dart-define=HMR_API_TOKEN` from the
   build & CI (`build-release.yml`).
6. Point `_baseUrl` at the proxy (`https://api.hmrbot.com`) instead of `srv.hmrbot.com`.
7. Add **Firebase App Check** (`firebase_app_check` package, Play Integrity provider) and attach its token to
   each request header (e.g. `X-Firebase-AppCheck`).
8. Rebuild, verify chat still works end-to-end through the proxy.

### Result
- No secret in the binary → reverse engineering yields nothing useful.
- Only genuine, attested installs of the app can reach the backend.
- Removes the Google Play "hardcoded credential" risk.

## Simpler fallbacks (if App Check is too much for v1)

- **B. Worker proxy + Cloudflare Turnstile** instead of App Check (lighter, but weaker than Play Integrity).
- **C. Worker proxy + strict per-IP rate limiting only** (no attestation). Removes the secret from the client
  but leaves the endpoint open to scripted abuse — acceptable only as a short-term step.
- **D. Make the Flowise prediction endpoint public (no token) + rate-limit.** Removes the secret entirely but
  offers the least protection. Not recommended beyond a quick unblock.

> Any fallback still achieves the primary goal — **the token leaves the client** — which is what Google's
> scanners and reverse-engineers care about.

## Action checklist

- [ ] Decide approach (recommended: Worker proxy + App Check).
- [ ] Stand up the Worker proxy with the Flowise token as a server secret.
- [ ] (If App Check) enable App Check in Firebase + Play Integrity; add `firebase_app_check` to the app.
- [ ] Remove `HMR_API_TOKEN` from the app code, the build command, and `build-release.yml`.
- [ ] Repoint `_baseUrl` to the proxy; send the attestation token instead of the bearer token.
- [ ] End-to-end test (chat works; direct calls to the proxy without a valid attestation are rejected).
- [ ] Rotate the Flowise token once the old one is fully off the client.

## What does NOT need doing (verified)

- ❌ **No Git history purge / BFG run.** A full-history search (`git log --all` over `*.jks`, `key.properties`,
  `*.keystore`) found **nothing** — signing material was never committed. The "clean the git history" advice
  does not apply to this repo. The current `hmr-release.jks` is gitignored and was never pushed.
