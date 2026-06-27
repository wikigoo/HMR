# HMR — Google Play Store Listing Assets (draft)

> Companion to `To-do-for-publish-HMR-app-in-googleplay.md` (covers §D store listing + §E policy forms).
> All user-facing copy is in Persian per HMR global rules (advisory, not decisional · honest · live-price-only).
> **Draft for owner review** — fill the blanks marked `⟪…⟫` and confirm before entering into Play Console.
> Package: `ir.hmrbot.app` · App name (from `pubspec`/`@string/app_name`): ⟪confirm exact Persian app name⟫.

---

## 1. App title & category (§D 20)

- **App name (≤ 30 chars):** `همر` — or `همر | مشاور سخت‌افزار موبایل` (confirm it fits 30 chars; Play counts each char).
- **Default language:** Persian (fa-IR).
- **App category:** *Shopping* (primary candidate) or *Tools*. Recommendation: **Shopping** (buying-guide focus).
- **Tags:** mobile phones, buying guide, hardware, advice.
- **Contact:** email `wikigoo58@gmail.com` · website `https://hmrbot.com` · phone ⟪optional⟫.

## 2. Short description (§D 18) — max 80 chars, Persian

> مشاور هوشمند خرید و عیب‌یابی موبایل؛ راهنمای صادق برای بازار ایران.

(73 chars — verify in console; adjust if over 80.)

## 3. Full description (§D 18) — max 4000 chars, Persian (draft)

```
همر (HMR) دستیار هوش مصنوعی شماست برای دنیای موبایل و سخت‌افزار آن — ساخته‌شده برای کاربران ایرانی و به زبان فارسی.

همر تصمیم نمی‌گیرد؛ همر به شما مشاوره می‌دهد تا خودتان آگاهانه تصمیم بگیرید. پاسخ‌ها صادقانه‌اند و اگر چیزی نامشخص باشد، صریح گفته می‌شود.

با همر می‌توانید:
• راهنمای خرید گوشی نو — انتخاب بهترین گزینه متناسب با بودجه و نیاز شما
• راهنمای خرید گوشی دست‌دوم — همراه با نکات تشخیص تقلب و کالای غیراصل
• عیب‌یابی و تشخیص خرابی سخت‌افزار — کمک گام‌به‌گام برای مشکلات رایج
• آموزش سخت‌افزار — یادگیری مفاهیم به زبان ساده
• راهنمای لوازم جانبی — انتخاب درست شارژر، کابل، قاب و موارد دیگر

ویژگی‌ها:
• گفتگوی کاملاً فارسی و راست‌به‌چپ (RTL)
• رابط کاربری ساده و سریع
• تاریخچه گفتگوها روی دستگاه شما

درباره قیمت‌ها: همر هیچ‌گونه مسئولیتی در قبال دقت یا منصفانه بودن قیمت‌های اعلام‌شده ندارد. قیمت‌ها از جستجوی زنده وب استخراج می‌شوند و صرفاً جنبه راهنما دارند.

همر یک ابزار مشاوره‌ای است و جایگزین کارشناس یا تعمیرکار حرفه‌ای نمی‌شود.
```

(~900 chars — well within 4000. Expand with FAQ/examples if desired.)

## 4. Graphic assets checklist (§D 15–19) — owner to produce

| Asset | Spec | Status |
|---|---|---|
| App icon | 512×512 PNG, 32-bit w/ alpha | ⟪needed⟫ (a launcher icon already exists in-app via `flutter_launcher_icons`) |
| Feature graphic | 1024×500 PNG/JPG | ⟪needed⟫ |
| Phone screenshots | ≥2 (4–8 recommended), 9:16, min 320px | ⟪capture: chat screen, conversations list, sign-in, disclaimer⟫ |
| Promo video (optional) | YouTube URL | ⟪optional⟫ |

> Tip: capture screenshots from the signed build on a real device/emulator (see checklist item 9).

## 5. Data Safety form (§E 22) — declared answers (draft)

Based on the actual code (`google_sign_in` in `auth_provider.dart`; chat sent to `https://srv.hmrbot.com`):

| Question | Answer |
|---|---|
| Does the app collect/share user data? | **Yes** |
| Data type — **Personal info → Name / Email** | Collected (via Google Sign-In). Purpose: **Account management / App functionality**. |
| Data type — **Messages → in-app chat content** | Collected (sent to the HMR backend to generate answers). Purpose: **App functionality**. |
| Data type — App info & performance (crash logs) | ⟪if Sentry stays enabled with a DSN: declare **Crash logs → Analytics**; currently no-op⟫ |
| Is data encrypted in transit? | **Yes** (HTTPS to `srv.hmrbot.com`). |
| Can users request data deletion? | **Yes** — provide the account-deletion path (section 6). |
| Is any data shared with third parties? | Google (Sign-In identity) ⟪+ OpenRouter/Gemini processes chat content on the backend — confirm how to disclose backend AI processing⟫. |

## 6. Account deletion (§E 23) — REQUIRED because of Google Sign-In

Play requires a way to delete the account + associated data, reachable both in-app and via a public URL.

- **Public URL:** ⟪create `https://hmrbot.com/delete-account` (or similar) and provide it in Play Console⟫.
- **In-app:** add a "حذف حساب کاربری" action (e.g., in settings/profile) that signs out and triggers backend deletion. ⟪implementation pending — currently not in the app⟫.
- **Suggested Persian copy:**
  > برای حذف حساب کاربری و داده‌های مرتبط، از بخش تنظیمات گزینه «حذف حساب کاربری» را انتخاب کنید یا به ⟪آدرس⟫ مراجعه کنید. پس از تأیید، حساب و تاریخچه گفتگوهای شما حذف می‌شود.

## 7. Privacy Policy (§E 21) — must be live at the referenced URL

The app references `https://hmrbot.com/privacy` (`conversations_screen.dart`). Confirm that page is **live** and covers:
- what is collected (Google account identity, chat content), why, and the legal basis;
- third parties (Google Sign-In; backend AI provider processing chat);
- data retention + the deletion mechanism (section 6);
- contact: `wikigoo58@gmail.com`.

## 8. Content rating (§E 24) & target audience (§E 25)

- Complete the IARC questionnaire → expected rating: **Everyone / 3+** (no violence, gambling, mature content).
- Target audience: **adults / 18+ or 13+** (not directed at children) — declare accordingly.
- Ads: **No ads** (confirm — none in code). Declare "contains ads = No".

---

## Open items for the owner
1. Confirm the exact Persian **app name** and that it fits Play's 30-char limit.
2. Produce **icon 512, feature graphic, and screenshots**.
3. Create and publish the **account-deletion URL** + add the in-app deletion action.
4. Confirm the **privacy policy page is live** and complete.
5. Decide how to disclose **backend AI processing** (OpenRouter/Gemini) in Data Safety + privacy policy.
