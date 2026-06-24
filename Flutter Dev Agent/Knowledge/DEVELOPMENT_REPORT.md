# HMR Chatbot — Development Session Report

**Date:** 2026-06-20
**Project:** HMR (Hamer) — AI Mobile Hardware Advisor
**Platform:** Flutter (Dart)
**Target Market:** Iran

---

## 1. Session Overview

This report documents the end-to-end development and deployment of the HMR chatbot Flutter application. The session covered Flutter SDK installation on Windows, project scaffolding, dependency resolution with regional network constraints, build execution, and a runtime layout bug fix.

---

## 2. Environment Setup

### 2.1 Flutter SDK Installation

| Item | Detail |
|---|---|
| OS | Windows 11 (26H1, Build 28000) |
| Flutter Version | 3.44.2 (stable channel) |
| Installation Path | `C:\src\flutter` |
| PATH Entries Added | `C:\src\flutter\bin`, `C:\src\flutter\bin\cache\dart-sdk\bin` |
| Verification | `flutter doctor` — passed with 1 non-blocking warning (Android SDK absent) |

### 2.2 Network Constraint: pub.dev Blocked

The default `pub.dev` package repository was inaccessible from the user's location (Iran), returning `authorization failed` errors. This was resolved by configuring the Flutter China mirror:

```powershell
PUB_HOSTED_URL = "https://pub.flutter-io.cn"
FLUTTER_STORAGE_BASE_URL = "https://storage.flutter-io.cn"
```

These were set both for the active PowerShell session and persistently at the user environment variable level via `[Environment]::SetEnvironmentVariable(...)`.

---

## 3. Project Architecture

### 3.1 Directory Structure

```
hmr_chatbot/
├── pubspec.yaml
├── analysis_options.yaml
└── lib/
    ├── main.dart
    ├── models/
    │   └── message_model.dart
    ├── services/
    │   └── api_service.dart
    ├── providers/
    │   └── chat_provider.dart
    ├── screens/
    │   └── chat_screen.dart
    └── widgets/
        └── chat_bubble.dart
```

### 3.2 Dependency Graph

| Package | Version | Purpose |
|---|---|---|
| `provider` | ^6.1.1 | State management (ChangeNotifier + Provider) |
| `shared_preferences` | ^2.2.2 | Local chat history persistence |
| `http` | ^1.2.0 | REST API client for Flowise backend |
| `intl` | ^0.20.2 | Internationalization (pinned by flutter_localizations) |
| `flutter_localizations` | SDK | RTL Persian locale support |

### 3.3 Architectural Pattern

The application follows a **Provider-based MVVM-like** architecture:

```
┌─────────────────────────────────────────────────┐
│  UI Layer (screens/, widgets/)                  │
│  ┌───────────┐  ┌──────────────┐               │
│  │ ChatScreen │  │ ChatBubble   │               │
│  └─────┬─────┘  └──────────────┘               │
│        │ watch / read                            │
├────────┼────────────────────────────────────────┤
│  State Layer (providers/)                       │
│  ┌─────┴──────────────────────────┐            │
│  │ ChatProvider (ChangeNotifier)  │            │
│  │ - messages: List<MessageModel> │            │
│  │ - isLoading: bool              │            │
│  │ - sendMessage() / clearHistory │            │
│  └─────┬──────────────────────────┘            │
│        │ calls                                   │
├────────┼────────────────────────────────────────┤
│  Service Layer (services/)                      │
│  ┌─────┴────────────┐                          │
│  │ ApiService        │                          │
│  │ - sendMessage()   │                          │
│  └─────┬────────────┘                          │
│        │ HTTP POST                               │
├────────┼────────────────────────────────────────┤
│  Data Layer (models/)                           │
│  ┌─────┴──────────────────────────┐            │
│  │ MessageModel                    │            │
│  │ - id, text, role, timestamp     │            │
│  │ - toJson() / fromJson()         │            │
│  │ - encodeList() / decodeList()   │            │
│  └────────────────────────────────┘            │
└─────────────────────────────────────────────────┘
```

---

## 4. Key Implementation Details

### 4.1 RTL & Localization (`main.dart`)

- Locale forced to `fa_IR` (Persian/Iran)
- `Directionality` widget wraps the entire app with `TextDirection.rtl`
- `GlobalMaterialLocalizations.delegate` and `GlobalWidgetsLocalizations.delegate` registered
- Material 3 theming with `ColorScheme.fromSeed` (deep blue `#1565C0`)

### 4.2 Message Model (`message_model.dart`)

- `MessageRole` enum: `user` | `ai`
- Factory constructors: `MessageModel.userMessage(text)` and `MessageModel.aiMessage(text)` — auto-generate unique IDs and timestamps
- Full JSON serialization/deserialization for `shared_preferences` persistence
- Static `encodeList()` / `decodeList()` for batch storage

### 4.3 API Service (`api_service.dart`)

- Endpoint: `POST http://{VPS_IP}:3000/api/v1/prediction/{CHATFLOW_ID}`
- Payload: `{"question": "USER_TEXT"}`
- Response parsing: extracts the `text` key from the JSON body
- Timeout: 30 seconds
- Custom `ApiException` class with Persian error messages for:
  - `TimeoutException` — connectivity check prompt
  - `ClientException` — server unreachable
  - `FormatException` — invalid response
  - Non-200 status codes — server error with code
  - Generic catch-all

### 4.4 Chat Provider (`chat_provider.dart`)

- Extends `ChangeNotifier` for reactive UI updates
- `sendMessage(text)`: adds user message → sets loading → calls API → adds AI/error response → persists
- `loadHistory()`: restores messages from `shared_preferences` on app launch
- `clearHistory()`: removes all messages from memory and storage
- History key: `hmr_chat_history`

### 4.5 Chat Screen (`chat_screen.dart`)

- `ListView.builder` with auto-scroll via `ScrollController`
- Empty state with welcome message and icon
- Typing indicator: animated three-dot widget (`_TypingDots`) with `AnimationController` (1200ms loop, staggered opacity)
- **Disclaimer widget**: amber warning bar (`#FFF3E0` background, `#E65100` icon) with the exact Persian text:
  > "همر هیچ‌گونه مسئولیتی در قبال دقت یا منصفانه بودن قیمت‌های اعلام شده ندارد. قیمت‌ها از جستجوی زنده وب استخراج می‌شوند."
- Input bar: rounded `TextField` with send `IconButton`, disabled during loading
- Clear history confirmation dialog (AlertDialog)

### 4.6 Chat Bubble (`chat_bubble.dart`)

- Differentiated styling: user messages use `colorScheme.primary`, AI messages use `colorScheme.surfaceContainerHighest`
- Asymmetric border radii (user: bottom-left rounded, AI: bottom-right rounded)
- Sender label ("شما" for user, "همر" for AI)
- Timestamp formatted as `HH:mm`
- Max width: 78% of screen width

---

## 5. Build & Deployment

### 5.1 Dependency Resolution

| Issue | Resolution |
|---|---|
| `flutter_lints` unavailable on China mirror | Removed from `dev_dependencies` |
| `intl ^0.19.0` conflicts with SDK-pinned `0.20.2` | Updated constraint to `^0.20.2` |
| pub.dev authorization failures | Switched to `pub.flutter-io.cn` mirror |

### 5.2 Build Result

```
flutter pub get  → 48 dependencies resolved successfully
flutter devices  → 3 targets: Windows (desktop), Chrome (web), Edge (web)
flutter run -d chrome → Compiled and launched on Chrome 149
```

### 5.3 Available Targets

| Device | Type | Identifier |
|---|---|---|
| Windows | desktop | `windows-x64` |
| Chrome | web | `web-javascript` |
| Edge | web | `web-javascript` |

---

## 6. Bug Fix: RenderFlex Overflow

### 6.1 Symptom

```
A RenderFlex overflowed by 10 pixels on the right.
Location: chat_screen.dart, _TypingDots widget
```

### 6.2 Root Cause

The `SizedBox` wrapping `_TypingDots` had `width: 20.0`. Each dot is `6px` wide with `2px` horizontal margin on both sides, totaling `10px` per dot. Three dots require `30px`, causing a `10px` overflow.

### 6.3 Fix Applied

```dart
// File: lib/screens/chat_screen.dart, line ~206
// Before:
SizedBox(width: 20, height: 12, child: _TypingDots())

// After:
SizedBox(width: 35, height: 12, child: _TypingDots())
```

---

## 7. Pending Actions

| # | Action | Priority |
|---|---|---|
| 1 | Replace `YOUR_VPS_IP` and `YOUR_CHATFLOW_ID` in `api_service.dart` with actual values | Critical |
| 2 | Add Vazirmatn Persian font to `pubspec.yaml` for improved typography | Low |
| 3 | Install Android SDK / Android Studio for mobile deployment | Medium |
| 4 | Implement proper error state UI (retry button on failed messages) | Medium |
| 5 | Add message timestamp grouping (date separators) | Low |

---

## 8. Session Timeline

| Step | Action | Result |
|---|---|---|
| 1 | Flutter SDK installation guidance | Provided step-by-step Windows instructions |
| 2 | Project scaffolding | Created 7 files across 5 directories |
| 3 | `flutter pub get` | Failed — pub.dev blocked |
| 4 | China mirror configuration | Set `PUB_HOSTED_URL` and `FLUTTER_STORAGE_BASE_URL` |
| 5 | `intl` version fix | Updated `^0.19.0` → `^0.20.2` |
| 6 | `flutter_lints` removal | Removed unavailable dev dependency |
| 7 | `flutter pub get` (retry) | 48 packages resolved |
| 8 | `flutter devices` | 3 targets detected |
| 9 | `flutter run -d chrome` | App launched successfully |
| 10 | RenderFlex overflow fix | `SizedBox` width: 20 → 35 |

---

*Report generated by GitHub Copilot — Senior Flutter Developer session.*