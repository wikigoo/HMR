
# ۱) راهنمای #1b — Keystore Rotation + Play App Signing + CI

**چرا حیاتی است:** کلید فعلی (`hmr-release.jks` با پسورد متنی `HmrBot2025!`) احتمالاً افشا شده. کلید امضای اپ تنها چیزی است که هویت اپ تو را تضمین می‌کند؛ اگر لو برود، مهاجم می‌تواند آپدیت جعلی امضا کند. راه‌حلِ ریشه‌ای: کلید نو بساز، Play App Signing را فعال کن (تا Google کلید اصلی را نگه دارد و فقط upload key دست تو باشد)، و کلید قدیمی را از history پاک کن.

### Checklist
1. تولید keystore جدید (upload key) با keytool
2. ذخیرهٔ امن آن خارج از مخزن + به‌روزرسانی `key.properties` محلی
3. حذف keystore و key.properties قدیمی از git history (در صورت commit قبلی)
4. چرخش توکن Flowise (چون اگر مخزن عمومی بوده، توکن هم سوخته)
5. تنظیم GitHub Actions secrets برای امضای CI
6. استخراج SHA-1/256 برای #4

---

**قدم ۱ — تولید کلید جدید**
- **Explanation:** یک کلید RSA جدید با اعتبار طولانی می‌سازیم. پسورد را **به‌صورت تعاملی** وارد کن تا در تاریخچهٔ شل و در هیچ فایلی ظاهر نشود.
- **Environment:** PowerShell (Windows)
- **Path:** `C:\Users\wikig\keys\` (یک پوشهٔ امن خارج از پروژه؛ اول بسازش)
- **Code:**
```powershell
mkdir C:\Users\wikig\keys
keytool -genkeypair -v -keystore C:\Users\wikig\keys\hmr-upload.jks -keyalg RSA -keysize 2048 -validity 10000 -alias hmr-upload
```
(keytool همراه JDK اندروید‌استودیو است. اگر شناخته نشد، از `"<AndroidStudio>\jbr\bin\keytool.exe"` استفاده کن.)

**قدم ۲ — اتصال کلید به پروژه (محلی)**
- **Explanation:** `key.properties` محلی به کلید جدید اشاره می‌کند. این فایل gitignore شده؛ هرگز commit نشود. پسوردها همان‌هایی است که در قدم ۱ تعاملی وارد کردی.
- **Environment:** ویرایشگر متن
- **Path:** `C:\Users\wikig\hmr_chatbot\android\key.properties`
- **Code:**
```properties
storePassword=YOUR_STORE_PASSWORD
keyPassword=YOUR_KEY_PASSWORD
keyAlias=hmr-upload
storeFile=C:/Users/wikig/keys/hmr-upload.jks
```

**قدم ۳ — پاک‌سازی history (فقط اگر قبلاً commit شده بود)**
- **Explanation:** اگر `hmr-release.jks` یا `key.properties` قبلاً push شده‌اند، gitignore کافی نیست؛ باید از کل تاریخچه حذف شوند. این کار history را بازنویسی می‌کند و force-push می‌خواهد — هر clone دیگری باید دوباره clone شود.
- **Environment:** PowerShell، در ریشهٔ مخزن
- **Path:** `C:\Users\wikig\hmr_chatbot`
- **Code:**
```powershell
pip install git-filter-repo
git filter-repo --path android/app/hmr-release.jks --path android/key.properties --invert-paths
git push origin --force --all
```
سپس کلید قدیمی `android/app/hmr-release.jks` را از روی دیسک حذف کن.

**قدم ۴ — چرخش توکن Flowise**
- **Explanation:** توکنی که قبلاً در `.env`/dart-define بود، اگر مخزن عمومی بوده، سوخته. در پنل Flowise یک API Key جدید بساز و قدیمی را باطل کن. این توکن جدید فقط روی سرور (در proxi بخش ۲) می‌نشیند.
- **Environment:** مرورگر — `panel.hmrbot.com`
- **Path:** Flowise → Settings → API Keys

**قدم ۵ — فعال‌سازی Play App Signing + استخراج SHA**
- **Explanation:** در Play Console اپ را بساز و AABِ امضاشده با upload key را در یک تست‌ترک آپلود کن؛ Play App Signing به‌صورت پیش‌فرض برای اپ‌های جدید فعال است و Google کلید اصلی را نگه می‌دارد. بعد، fingerprintها را برای #4 بگیر.
- **Environment:** PowerShell + مرورگر (`play.google.com/console`)
- **Path:** Play Console → App integrity → Play App Signing
- **Code:**
```powershell
flutter build appbundle --release
keytool -list -v -keystore C:\Users\wikig\keys\hmr-upload.jks -alias hmr-upload
```
⚠️ **نکتهٔ وابستگی:** keytool بالا SHA-1/256 ِ **upload key** را می‌دهد. بعد از enroll، Play Console یک SHA-1/256 ِ **app signing key** هم نشان می‌دهد. در #4 باید **هر دو** را ثبت کنی.

**قدم ۶ — GitHub Actions secrets (برای build خودکار)**
- **Explanation:** `build.gradle.kts` از قبل آمادهٔ خواندن از env است (`HMR_KEY_ALIAS`, `HMR_KEY_PASSWORD`, `HMR_STORE_PASSWORD`, `HMR_KEYSTORE_PATH`). در CI، keystore را base64 می‌کنی و به‌صورت secret می‌گذاری.
- **Environment:** PowerShell (برای کدگذاری) + GitHub repo settings
- **Path:** GitHub → Settings → Secrets and variables → Actions
- **Code:**
```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("C:\Users\wikig\keys\hmr-upload.jks")) > keystore.b64.txt
```
محتوای `keystore.b64.txt` را به‌عنوان secret `HMR_KEYSTORE_B64` بگذار؛ workflow آن را decode و در runner می‌نویسد و مسیرش را در `HMR_KEYSTORE_PATH` می‌گذارد. (runnerهای GitHub به Google دسترسی دارند — Aliyun mirror لازم نیست.)

---

# ۲) اسپک #2 — VPS Proxy (خروج توکن از کلاینت)

**چرا:** توکن داخل APK قابل استخراج است → هرکس می‌تواند مستقیم به Flowise بزند (هزینهٔ OpenRouter، scrape دانش پایه، دور زدن گاردریل‌ها). راه‌حل ریشه‌ای: یک سرویس واسط روی همان VPS که توکن را **سمت سرور** نگه می‌دارد، rate-limit می‌کند، و قابلیت revoke/log مرکزی می‌دهد (داده‌ٔ گفتگو هم که دارایی اصلی طرح است، اینجا متمرکز می‌شود).

> صداقت مهندسی: هیچ طرح سمت‌کلاینت ۱۰۰٪ نفوذناپذیر نیست. بُرد اصلی این است که **توکن Flowise دیگر ارسال نمی‌شود** و کنترل (نرخ، revoke، لاگ) به سرور منتقل می‌شود.

### Checklist
1. سرویس Node/Express سبک داخل Docker
2. توکن Flowise در `.env` سمت سرور (هرگز در مخزن)
3. Rate-limit + app-key سبک + سقف طول ورودی
4. مسیردهی Nginx به سرویس
5. تغییر `api_service.dart` — **فقط بعد از live شدن proxy**

**قدم ۱ — کد سرویس**
- **Explanation:** یک proxy کوچک که app-key را چک می‌کند، نرخ را محدود می‌کند، و درخواست را با توکن سروری به Flowise فوروارد می‌کند.
- **Environment:** Ubuntu VPS
- **Path:** `/opt/hmr-proxy/server.js`
- **Code:**
```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');

const app = express();
app.set('trust proxy', 1);
app.use(express.json({ limit: '16kb' }));

const { FLOWISE_URL, FLOWISE_TOKEN, APP_KEY, PORT = 8081 } = process.env;

const limiter = rateLimit({ windowMs: 60_000, max: 20 });
app.use('/predict', limiter);

app.post('/predict', async (req, res) => {
  if (req.get('x-app-key') !== APP_KEY) return res.status(401).json({ error: 'unauthorized' });

  const question = (req.body?.question ?? '').toString().trim();
  const sessionId = (req.body?.sessionId ?? '').toString().trim();
  if (!question || question.length > 1000) return res.status(400).json({ error: 'bad_input' });

  try {
    const r = await fetch(FLOWISE_URL, {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${FLOWISE_TOKEN}`, 'Content-Type': 'application/json' },
      body: JSON.stringify({ question, streaming: false, overrideConfig: { sessionId } }),
    });
    const data = await r.json();
    return res.status(r.status).json({ text: data.text ?? '' });
  } catch (e) {
    return res.status(502).json({ error: 'upstream_error' });
  }
});

app.listen(PORT, () => console.log(`hmr-proxy on :${PORT}`));
```

**قدم ۲ — env سمت سرور (هرگز در git)**
- **Path:** `/opt/hmr-proxy/.env`
- **Code:**
```bash
FLOWISE_URL=https://srv.hmrbot.com/api/v1/prediction/843b252b-6ed0-4064-96cd-cb78367bd7b3
FLOWISE_TOKEN=<NEW_FLOWISE_TOKEN_FROM_STEP_1b.4>
APP_KEY=<RANDOM_ROTATABLE_APP_KEY>
PORT=8081
```

**قدم ۳ — Dockerfile + اجرا**
- **Path:** `/opt/hmr-proxy/Dockerfile`
- **Code:**
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package.json ./
RUN npm install --omit=dev
COPY server.js ./
CMD ["node", "server.js"]
```
```bash
cd /opt/hmr-proxy
npm init -y && npm install express express-rate-limit
docker build -t hmr-proxy .
docker run -d --name hmr-proxy --env-file .env -p 127.0.0.1:8081:8081 --restart unless-stopped hmr-proxy
```

**قدم ۴ — مسیر Nginx**
- **Path:** `/etc/nginx/sites-available/hmrbot` (بلوک server موجود `srv.hmrbot.com`)
- **Code:**
```nginx
location /predict {
    proxy_pass http://127.0.0.1:8081/predict;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```
```bash
nginx -t && systemctl reload nginx
```

**قدم ۵ — تغییر اپ (فقط بعد از تست موفق proxy)**
- **Explanation:** توکن Flowise از کلاینت حذف می‌شود؛ به‌جایش endpoint جدید + یک app-key سبکِ قابل‌چرخش. این app-key هم استخراج‌پذیر است اما کم‌ارزش و قابل revoke از سرور — برخلاف توکن اصلی.
- **Path:** `C:\Users\wikig\hmr_chatbot\lib\services\api_service.dart`
- **Code:**
```dart
static const String _predictUrl = 'https://srv.hmrbot.com/predict';
static const String _appKey = String.fromEnvironment('HMR_APP_KEY');
// headers: { 'x-app-key': _appKey, 'Content-Type': 'application/json' }
// body:    { 'question': question, 'sessionId': sessionId }
// response: data['text']
```

---

# ۳) /summarize

```
TITLE: HMR Android App — Audit & 3-Round Remediation (Status)
DATE: 2026-06-23 (UTC; exact session time unavailable)

TOPICS DISCUSSED (≤300 words):
A full security/quality audit of the HMR Flutter Android app (C:\Users\wikig\hmr_chatbot,
applicationId ir.hmrbot.app) was performed by reading the live project on disk, followed by two
autonomous remediation rounds executed by a VS Code agent and verified on disk after each round.
Round 1 closed: hardened .gitignore (secrets/log files), fixed deleteConversation() to delete
SQLite rows + removed dead SharedPreferences read path, set allowBackup=false, pinned SDK levels,
moved MainActivity to ir/hmrbot/app, added a 1000-char input cap, added Semantics labels, split the
app label into en/fa strings.xml, and added Jalali (Shamsi) date labels. Round 2 closed: bumped
compileSdk and targetSdk to 36 (Play 2026 requirement), removed a misleading cross-device "sync"
claim from the drawer, wired a Privacy Policy link via url_launcher (placeholder URL), labeled the
remaining ghost icon buttons, removed the invalid fullBackupContent attribute, and reverted minSdk
to flutter.minSdkVersion (24). The signing config now reads from env vars (CI) or a gitignored
key.properties (local), with no secrets in source. flutter analyze is clean and a release AAB builds
successfully at API 36. A local-only Aliyun Gradle init script works around sqflite_android's
hardcoded google() lookup on Iranian machines (not needed on CI runners).

ESTABLISHED SOLUTIONS (✅ done, verified on disk):
- All identified CODE-LEVEL defects resolved across Rounds 1–2.
- App is code-complete and builds to a release AAB targeting API 36.

CURRENT STATE:
- Branch: fix/hmr-remediation-2. flutter analyze clean.
- Signing wired for env/key.properties; keystore NOT yet rotated.
- API token STILL ships in the client (no proxy yet).

PENDING (USER ACTION):
1. Replace <PRIVACY_POLICY_URL> with a live policy URL (host on hmrbot.com).
2. #1b: rotate keystore, purge git history if committed, enroll Play App Signing.
3. #4: register upload + app-signing SHA-1/256 in Google Cloud (AFTER #1b).
4. #2: deploy VPS proxy; then apply the api_service.dart change.
5. #10: choose Crashlytics or Sentry (not yet integrated).
PUBLISH GATE: items 1 + 2 + 3 + 4.
```

---

**منابع:**
- امضای اندروید Flutter — docs.flutter.dev/deployment/android · Play App Signing — support.google.com/googleplay/android-developer/answer/9842756
- git-filter-repo — github.com/newren/git-filter-repo · Nginx reverse proxy — nginx.org/en/docs/http/ngx_http_proxy_module.html
- express-rate-limit — npmjs.com/package/express-rate-limit · Flowise Prediction API — docs.flowiseai.com

