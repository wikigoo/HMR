# Application Health & Accuracy Evaluation Checklist
**Platform:** Android | **Framework:** Flutter | **Infrastructure:** Hetzner VPS | **Application Type:** Flowise AI Chatbot

---

## 1. User Interface

### Layout & Responsiveness
- [ ] Verify all screens render correctly across common Android screen densities (mdpi, hdpi, xhdpi, xxhdpi, xxxhdpi) and aspect ratios (16:9, 18:9, 20:9)
- [ ] Confirm `LayoutBuilder` or `MediaQuery` is used for adaptive layouts rather than hardcoded pixel values
- [ ] Test portrait and landscape orientations; confirm no layout overflow or clipping occurs
- [ ] Validate that the chat message list uses `ListView.builder` (lazy loading) rather than a static `ListView`, per Flutter performance best practices [1](#0-0) 
- [ ] Confirm `const` constructors are used wherever possible in widget trees to minimize unnecessary rebuilds [2](#0-1) 
- [ ] Verify `build()` methods contain no network calls, expensive computations, or side effects [3](#0-2) 

### Interaction & UX
- [ ] Validate all interactive elements (send button, text input, scroll, navigation) respond correctly and without jank
- [ ] Confirm loading/typing indicators are displayed while awaiting AI responses from Flowise
- [ ] Verify error states (network failure, Flowise API timeout, empty response) display user-friendly messages rather than raw exceptions
- [ ] Test keyboard behavior: input field scrolls into view when keyboard appears; no content is obscured
- [ ] Confirm chat history scrolls to the latest message automatically on new responses
- [ ] Validate that long AI responses render correctly without truncation or overflow

### Accessibility (A11Y)
- [ ] Verify text contrast ratio meets the WCAG 4.5:1 minimum for all text elements [4](#0-3) 
- [ ] Confirm all interactive elements have `Semantics` labels for TalkBack screen reader compatibility [5](#0-4) 
- [ ] Test dynamic font scaling up to 200% (`textScaleFactor`) — no layout breakage or text clipping
- [ ] Run Flutter's built-in accessibility audit: `flutter test --tags=a11y`

### Theming
- [ ] Confirm Light and Dark mode (`ThemeMode.system`) are both implemented and visually consistent
- [ ] Verify `ThemeData` with `ColorScheme.fromSeed` is used for consistent Material 3 theming [6](#0-5) 

---

## 2. Backend Evaluation (Flowise + VPS)

### Flowise Service Health
- [ ] Confirm the Flowise process is running and healthy: `systemctl status flowise` or equivalent process manager (PM2, Docker)
- [ ] Verify the Flowise API endpoint (`/api/v1/prediction/<chatflowId>`) returns a valid `200 OK` response with a test payload
- [ ] Validate that the correct chatflow ID is configured in the Flutter app and matches the active Flowise flow
- [ ] Confirm all Flowise nodes (LLM, memory, vector store, tools) in the chatflow are connected and operational
- [ ] Check Flowise logs for recurring errors, failed LLM calls, or broken tool integrations
- [ ] Verify API keys for external LLM providers (OpenAI, Anthropic, etc.) are valid, not expired, and have sufficient quota

### Flutter–Backend Integration
- [ ] Confirm all HTTP requests to the Flowise API use `async`/`await` with proper `try-catch` error handling [7](#0-6) 
- [ ] Verify that JSON parsing of Flowise responses is performed in a separate `Isolate` via `compute()` to avoid blocking the UI thread [8](#0-7) 
- [ ] Validate that API base URLs and keys are stored in environment variables or a secure config, not hardcoded in Dart source
- [ ] Confirm request timeouts are configured (e.g., `HttpClient.connectionTimeout`) to prevent indefinite hangs
- [ ] Test behavior when the Flowise backend is unreachable — the app must not crash; it must surface a recoverable error state

### Data Integrity
- [ ] Verify chat session/conversation IDs are correctly passed to Flowise to maintain conversation memory across turns
- [ ] Confirm response parsing handles all Flowise output formats (text, JSON, streaming SSE if applicable)
- [ ] Validate that no sensitive user input is logged to the console in release builds (`dart:developer log` only in debug) [9](#0-8) 

---

## 3. Performance and Speed Optimization

### Flutter App Performance
- [ ] Build and profile the app in `profile` mode (`flutter run --profile`) using Flutter DevTools — not `debug` mode, which disables AOT compilation [10](#0-9) 
- [ ] Confirm the release APK is built with `flutter build apk --release` (AOT mode, all optimizations enabled) [11](#0-10) 
- [ ] Inspect the Flutter DevTools "Performance" tab for frame rendering — all frames should render within 16ms (60fps) or 8ms (120Hz)
- [ ] Check the "Widget Rebuild" tracker in DevTools for excessive or unnecessary widget rebuilds
- [ ] Verify expensive operations (JSON parsing, message formatting) use `compute()` to run in a background `Isolate` [8](#0-7) 
- [ ] Confirm image assets are appropriately sized and cached; avoid loading full-resolution images for thumbnails
- [ ] Validate app startup time — use `flutter run --trace-startup` and review the startup timeline

### APK Size & Architecture
- [ ] Evaluate APK size using `flutter build apk --analyze-size`
- [ ] Consider building with `--split-per-abi` to produce separate APKs per architecture (arm64-v8a, armeabi-v7a, x86_64), reducing download size [12](#0-11) 
- [ ] Confirm `treeShakeIcons` is enabled to remove unused Material icon glyphs from the release build

### Network Performance
- [ ] Measure Flowise API response latency from the Android device — identify if latency is in the VPS, LLM provider, or network
- [ ] Implement request debouncing on the send button to prevent duplicate API calls
- [ ] Evaluate whether streaming responses (SSE) from Flowise would improve perceived response time vs. waiting for full completion

### VPS Performance
- [ ] Monitor VPS CPU and RAM usage during peak chatbot usage: `htop`, `vmstat`, or a monitoring tool (Netdata, Prometheus)
- [ ] Confirm Flowise is not memory-leaking — check process memory over time
- [ ] Verify disk I/O is not a bottleneck, especially if Flowise uses a local vector store (e.g., Chroma, Faiss)

---

## 4. Technical Debt Assessment

### Code Quality
- [ ] Run `flutter analyze .` and resolve all warnings and errors — zero analyzer issues is the target [13](#0-12) 
- [ ] Confirm `analysis_options.yaml` includes `package:flutter_lints/flutter.yaml` as a baseline [14](#0-13) 
- [ ] Audit for `print()` statements — replace with `dart:developer log()` or remove entirely from production code [15](#0-14) 
- [ ] Verify null safety is fully adopted — no use of the `!` (bang) operator unless the value is provably non-null [16](#0-15) 
- [ ] Confirm no business logic resides inside `build()` methods or widget classes — enforce separation of concerns [17](#0-16) 
- [ ] Review for commented-out dead code blocks and remove them

### Architecture
- [ ] Verify the project follows a layered architecture: Presentation (widgets/screens), Domain (business logic), Data (API clients, models) [18](#0-17) 
- [ ] Confirm state management is consistent — no mixing of `setState`, `ChangeNotifier`, and ad-hoc global variables
- [ ] Audit for hardcoded strings — all user-facing strings should be externalized for future localization support
- [ ] Identify any deprecated Flutter/Dart APIs in use: `flutter pub outdated` and review migration guides

### Dependency Audit
- [ ] Run `flutter pub outdated` and document all outdated dependencies
- [ ] Check for packages with known security advisories on [pub.dev](https://pub.dev) or the Dart advisory database
- [ ] Remove unused dependencies from `pubspec.yaml`
- [ ] Verify `pubspec.lock` is committed to version control for reproducible builds

---

## 5. Security Checks

### Flutter App Security
- [ ] Confirm the release APK is signed with a production keystore — not the debug keystore [19](#0-18) 
- [ ] Verify ProGuard/R8 obfuscation is enabled for the release build to protect Dart code and native libraries
- [ ] Audit all API keys, tokens, and secrets — none should be embedded in Dart source or committed to version control; use `flutter_dotenv` or Android Keystore
- [ ] Confirm HTTPS is enforced for all network communication with the Flowise backend — no plain HTTP in production
- [ ] Verify Android `AndroidManifest.xml` does not declare unnecessary permissions (e.g., `READ_CONTACTS`, `ACCESS_FINE_LOCATION` if not needed)
- [ ] Test for insecure data storage — sensitive data (tokens, session IDs) must use `flutter_secure_storage`, not `SharedPreferences`
- [ ] Validate certificate pinning is implemented if the app communicates with a fixed backend endpoint

### VPS / Infrastructure Security
- [ ] Confirm SSH access uses key-based authentication only — password authentication must be disabled in `/etc/ssh/sshd_config`
- [ ] Verify a firewall (UFW or `iptables`) is active and only exposes required ports (e.g., 443 for HTTPS, 22 for SSH)
- [ ] Confirm Flowise is not exposed directly on a public port without a reverse proxy (Nginx/Caddy) with TLS termination
- [ ] Verify TLS certificates are valid, not expired, and auto-renewing (Let's Encrypt / Certbot)
- [ ] Check that the Flowise admin UI is not publicly accessible — restrict via IP allowlist or authentication
- [ ] Audit VPS user accounts — remove unused accounts; ensure the Flowise process runs as a non-root user
- [ ] Confirm OS-level security patches are applied: `apt list --upgradable` (Debian/Ubuntu)
- [ ] Review Nginx/Caddy access logs for anomalous traffic patterns or brute-force attempts

### API Security
- [ ] Verify the Flowise API endpoint requires authentication (API key header) — unauthenticated public access must be disabled
- [ ] Confirm rate limiting is configured on the reverse proxy to prevent abuse
- [ ] Validate CORS policy on the Flowise server allows only the expected origins

---

## 6. Backup and Update Requirements

### Application Backups
- [ ] Confirm the Flutter source code is in a version-controlled repository (Git) with remote backup (GitHub, GitLab, etc.)
- [ ] Verify Flowise chatflow configurations are exported and backed up (Flowise supports JSON export of flows)
- [ ] Back up all Flowise environment variables and `.env` files to a secure, encrypted location

### Database & Vector Store Backups
- [ ] If Flowise uses a database (SQLite, PostgreSQL, MySQL), confirm automated daily backups are configured
- [ ] Verify vector store data (Chroma, Pinecone, Weaviate, etc.) is backed up or can be re-indexed from source documents
- [ ] Test backup restoration procedures — a backup that has never been restored is unverified

### VPS Backups
- [ ] Enable Hetzner's automated VPS snapshot/backup feature in the Hetzner Cloud Console
- [ ] Confirm snapshots are taken at a regular cadence (daily or weekly) and retained for an adequate period
- [ ] Document the full disaster recovery procedure: how to restore the VPS and Flowise from a snapshot

### Updates
- [ ] Update Flutter SDK to the latest stable channel: `flutter upgrade` [20](#0-19) 
- [ ] Update all `pubspec.yaml` dependencies to their latest compatible versions after regression testing
- [ ] Update Flowise to the latest release — check the [Flowise GitHub releases](https://github.com/FlowiseAI/Flowise/releases) for security patches and new features
- [ ] Apply all pending OS security patches on the Hetzner VPS: `sudo apt update && sudo apt upgrade`
- [ ] Update the reverse proxy (Nginx/Caddy) to the latest stable version

---

## 7. Server Environment Configuration

### Reverse Proxy (Nginx / Caddy)
- [ ] Confirm Nginx/Caddy is configured as a reverse proxy forwarding traffic to the Flowise process (default port `3000`)
- [ ] Verify TLS/SSL is configured with a minimum of TLS 1.2; TLS 1.3 preferred
- [ ] Confirm HTTP → HTTPS redirect is enforced (301 redirect)
- [ ] Set appropriate request timeout values to accommodate LLM response latency (e.g., `proxy_read_timeout 120s` in Nginx)
- [ ] Enable gzip compression for API responses to reduce payload size
- [ ] Configure appropriate HTTP security headers: `Strict-Transport-Security`, `X-Content-Type-Options`, `X-Frame-Options`, `Content-Security-Policy`

### Process Management
- [ ] Confirm Flowise is managed by a process supervisor (PM2, systemd, or Docker) that auto-restarts on crash
- [ ] Verify the process manager is configured to start Flowise automatically on VPS reboot
- [ ] Review PM2/systemd logs for crash history: `pm2 logs` or `journalctl -u flowise`

### Resource Configuration
- [ ] Confirm VPS RAM is sufficient for the Flowise process and any in-memory vector stores — monitor with `free -h`
- [ ] Set Node.js memory limits appropriately for the Flowise process (e.g., `NODE_OPTIONS=--max-old-space-size=2048`)
- [ ] Verify swap space is configured as a safety net for memory spikes
- [ ] Confirm disk space is adequate and monitored: `df -h`; set up alerts for >80% usage
- [ ] Review and tune Nginx `worker_processes` and `worker_connections` for expected concurrency

### Logging & Monitoring
- [ ] Confirm application logs are persisted to disk with log rotation configured (`logrotate`)
- [ ] Set up uptime monitoring for the Flowise endpoint (UptimeRobot, Betterstack, or self-hosted)
- [ ] Configure alerting for VPS CPU >80%, RAM >85%, or disk >80% thresholds
- [ ] Integrate structured logging in Flowise for easier parsing and analysis

---

## 8. Scalability and Stress Testing

### Load Testing
- [ ] Define expected concurrent user count and peak request rate (requests per second to the Flowise API)
- [ ] Conduct load testing using a tool such as `k6`, `Locust`, or `Apache JMeter` against the Flowise endpoint
- [ ] Measure response time degradation under load — establish P50, P95, and P99 latency benchmarks
- [ ] Identify the VPS resource saturation point (CPU, RAM, or network) under simulated load

### Flutter App Stress Testing
- [ ] Test the chat UI with a large number of messages in the conversation history (500+ messages) — verify `ListView.builder` maintains smooth scrolling
- [ ] Simulate rapid successive message sends and verify the app queues or debounces requests correctly
- [ ] Test behavior under poor network conditions using Android's built-in network throttling (Developer Options → Network condition)

### Scalability Planning
- [ ] Document the current single-VPS architecture limitations — identify at what load a second instance or horizontal scaling would be required
- [ ] Evaluate whether Flowise supports stateless horizontal scaling (depends on session/memory storage configuration)
- [ ] If scaling is anticipated, plan for a load balancer (Hetzner Load Balancer) and shared session/vector store backend
- [ ] Assess LLM provider rate limits (tokens per minute, requests per minute) as a potential bottleneck independent of VPS capacity

---

## 9. Documentation and Error Handling

### Error Handling
- [ ] Confirm all `async` functions that call the Flowise API are wrapped in `try-catch` blocks with typed exception handling [21](#0-20) 
- [ ] Verify the app never displays raw stack traces or internal error messages to end users
- [ ] Implement a global Flutter error handler via `FlutterError.onError` and `PlatformDispatcher.instance.onError` to catch and log unhandled exceptions
- [ ] Confirm network errors, HTTP 4xx/5xx responses, and LLM provider errors each have distinct, user-friendly error messages
- [ ] Validate retry logic for transient failures (e.g., exponential backoff for 503 responses from Flowise)
- [ ] Test the "no internet connection" scenario — the app must detect connectivity loss and prompt the user appropriately

### Logging & Crash Reporting
- [ ] Integrate a crash reporting service (Firebase Crashlytics, Sentry) to capture unhandled exceptions in production
- [ ] Confirm that crash reports include sufficient context (device model, OS version, app version, error type) without capturing PII
- [ ] Verify that `dart:developer log()` calls are used for debug logging and are not compiled into release builds

### Developer Documentation
- [ ] Confirm a `README.md` exists with: project overview, local setup instructions, environment variable reference, and deployment steps
- [ ] Document the Flowise chatflow architecture — which LLM, memory type, tools, and vector store are used
- [ ] Document the VPS configuration: OS, Nginx config, process manager setup, and port layout
- [ ] Maintain a `CHANGELOG.md` or equivalent to track version history and breaking changes
- [ ] Document the backup and restore procedure for both the Flutter app and the Flowise backend

### User-Facing Documentation
- [ ] Provide in-app guidance or onboarding for first-time users explaining the chatbot's capabilities and limitations
- [ ] Display clear messaging when the AI is unable to answer a query (hallucination guard, out-of-scope topics)
- [ ] Include a privacy notice informing users what data is sent to the backend and any third-party LLM providers

---

## Summary Severity Matrix

| Category | Priority | Typical Blockers |
|---|---|---|
| Security Checks | Critical | Exposed Flowise API, HTTP-only traffic, hardcoded secrets |
| Backend Evaluation | Critical | Flowise unreachable, broken chatflow, no error handling |
| Backup & Updates | High | No VPS snapshots, outdated Flowise with CVEs |
| Server Configuration | High | No TLS, no process supervisor, no rate limiting |
| Performance | High | Debug build in production, UI thread blocking |
| User Interface | Medium | Layout overflow, no loading states, no error states |
| Error Handling | Medium | Raw exceptions shown to users, no crash reporting |
| Technical Debt | Medium | Outdated deps, analyzer warnings, no linting |
| Scalability | Low (now) | Single-VPS bottleneck at scale |

### Citations

**File:** docs/rules/rules.md (L62-63)
```markdown
* **Code structure:** Adhere to maintainable code structure and separation of
  concerns (e.g., UI logic separate from business logic).
```

**File:** docs/rules/rules.md (L82-82)
```markdown
* **Logging:** Use the `logging` package instead of `print`.
```

**File:** docs/rules/rules.md (L96-97)
```markdown
* **Async/Await:** Ensure proper use of `async`/`await` for asynchronous
  operations with robust error handling.
```

**File:** docs/rules/rules.md (L100-101)
```markdown
* **Null Safety:** Write code that is soundly null-safe. Leverage Dart's null
  safety features. Avoid `!` unless the value is guaranteed to be non-null.
```

**File:** docs/rules/rules.md (L108-110)
```markdown
* **Exception Handling:** Use `try-catch` blocks for handling exceptions, and
  use exceptions appropriate for the type of exception. Use custom exceptions
  for situations specific to your code.
```

**File:** docs/rules/rules.md (L122-123)
```markdown
* **List Performance:** Use `ListView.builder` or `SliverList` for long lists to
  create lazy-loaded lists for performance.
```

**File:** docs/rules/rules.md (L124-125)
```markdown
* **Isolates:** Use `compute()` to run expensive calculations in a separate
  isolate to avoid blocking the UI thread, such as JSON parsing.
```

**File:** docs/rules/rules.md (L126-127)
```markdown
* **Const Constructors:** Use `const` constructors for widgets and in `build()`
  methods whenever possible to reduce rebuilds.
```

**File:** docs/rules/rules.md (L142-146)
```markdown
* **Logical Layers:** Organize the project into logical layers:
    * Presentation (widgets, screens)
    * Domain (business logic classes)
    * Data (model classes, API clients)
    * Core (shared classes, utilities, and extension types)
```

**File:** docs/rules/rules.md (L153-157)
```markdown
Include the package in the `analysis_options.yaml` file. Use the following
`analysis_options.yaml` file as a starting point:

```yaml
include: package:flutter_lints/flutter.yaml
```

**File:** docs/rules/rules_4k.md (L25-25)
```markdown
* **Logging:** Use `dart:developer` `log()` locally. NEVER use `print`.
```

**File:** docs/rules/rules_4k.md (L28-29)
```markdown
* **Build Methods:** Keep pure and fast. No side effects. No network calls.
* **Isolates:** Use `compute()` for heavy tasks like JSON parsing.
```

**File:** docs/rules/rules_4k.md (L57-61)
```markdown
## Visual Design (Material 3)
* **Aesthetics:** Premium, custom look. "Wow" the user. Avoid default blue.
* **Theme:** Use `ThemeData` with `ColorScheme.fromSeed`.
* **Modes:** Support Light & Dark modes (`ThemeMode.system`).
* **Typography:** `google_fonts`. Define a consistent Type Scale.
```

**File:** docs/rules/rules_4k.md (L72-75)
```markdown
* **Contrast:** 4.5:1 minimum for text.
* **Semantics:** Label all interactive elements specifically.
* **Scale:** Test dynamic font sizes (up to 200%).
* **Screen Readers:** Verify with TalkBack/VoiceOver.
```

**File:** docs/rules/rules_4k.md (L80-80)
```markdown
* **Analyze:** `flutter analyze .`
```

**File:** packages/flutter_tools/lib/src/build_info.dart (L462-467)
```dart

  /// Built in AOT mode with some optimizations and a VM service.
  profile,

  /// Built in AOT mode with all optimizations and no VM service.
  release,
```

**File:** packages/flutter_tools/lib/src/commands/build_apk.dart (L38-44)
```dart
      ..addFlag(
        'split-per-abi',
        negatable: false,
        help:
            'Whether to split the APKs per ABIs. '
            'To learn more, see: https://developer.android.com/studio/build/configure-apk-splits#configure-abi-split',
      )
```

**File:** dev/a11y_assessments/android/app/build.gradle (L58-65)
```gradle
   signingConfigs {
       release {
           keyAlias keystoreProperties['keyAlias']
           keyPassword keystoreProperties['keyPassword']
           storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
           storePassword keystoreProperties['storePassword']
       }
   }
```

**File:** docs/roadmap/[Archive]-Old-Roadmaps.md (L307-309)
```markdown
## Release Channels and Cadence

Flutter offers three “channels” from which developers can receive updates: master, beta and stable, with increasing levels of stability and confidence of quality but longer lead times for changes to propagate. We plan to release one beta build each month, typically near the start of the month, and about four stable releases throughout the year. We recommend that you use the stable channel for apps released to end-users. For more details on our release process, see the [Flutter build release channels](../releases/Flutter-build-release-channels.md) wiki page.
```
