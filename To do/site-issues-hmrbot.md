# مشکلات سایت hmrbot.com
**تاریخ شناسایی:** ۲۰۲۶-۰۶-۲۸  
**منبع:** گزارش آنالیز کامل سایت — DevOps Agent

---

## 🔴 اولویت بالا

- [ ] **۱ — HTTP → HTTPS اجباری نیست**  
  `http://hmrbot.com` به جای redirect، مستقیم HTTP 200 برمی‌گرداند.  
  **رفع:** Cloudflare Dashboard → SSL/TLS → Edge Certificates → **Always Use HTTPS** را روشن کنید.

- [ ] **۲ — www.hmrbot.com ریدایرکت 301 ندارد**  
  `www.hmrbot.com` مستقیم HTTP 200 برمی‌گرداند، باید به `https://hmrbot.com` ریدایرکت کند.  
  **رفع:** Cloudflare Dashboard → Redirects → Bulk Redirects → اضافه کردن: `www.hmrbot.com/*` → `https://hmrbot.com/$1` (301, Permanent)

- [ ] **۳ — هدرهای امنیتی وجود ندارند**  
  همه ۶ هدر امنیتی استاندارد غایبند: `Strict-Transport-Security`, `X-Frame-Options`, `X-Content-Type-Options`, `Content-Security-Policy`, `Referrer-Policy`, `Permissions-Policy`  
  **رفع:** Cloudflare Dashboard → Rules → Transform Rules → Modify Response Header → اضافه کردن هدرها برای همه مسیرها:
  ```
  Strict-Transport-Security: max-age=31536000; includeSubDomains
  X-Frame-Options: SAMEORIGIN
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=()
  ```

---

## 🟡 اولویت متوسط

- [ ] **۴ — ایمیل‌های دامنه ساخته نشده‌اند**  
  سه آدرس در سایت و اپ استفاده شده ولی وجود ندارند:
  - `info@hmrbot.com` ← صفحه تماس
  - `support@hmrbot.com` ← اپ Flutter (حذف حساب)
  - `privacy@hmrbot.com` ← صفحه حریم خصوصی  
  
  **رفع:** Cloudflare Dashboard → Email Routing → Add address → forward هر سه به `wikigoo58@gmail.com`

- [ ] **۵ — زیردامنه‌های مرده لینک دارند**  
  `chat.hmrbot.com` در صفحه تماس لینک دارد ← Connection refused  
  `blog.hmrbot.com` در footer لینک دارد ← Connection refused  
  **رفع کوتاه‌مدت:** حذف لینک‌ها از `contact.astro` و layout footer  
  **رفع بلندمدت:** راه‌اندازی یا حذف DNS record از Cloudflare

---

## 🟢 اولویت پایین

- [ ] **۶ — `FLOWISE_API_KEY = undefined` در سورس صفحه**  
  متغیر محیطی در build-time تنظیم نشده و مقدار `undefined` در JS سورس صفحه `/ai` دیده می‌شود.  
  **تأثیر واقعی:** صفر — nginx توکن Bearer را inject می‌کند و احراز هویت از آن طریق انجام می‌شود.  
  **رفع اختیاری:** Cloudflare Pages → Settings → Environment Variables → حذف یا خالی گذاشتن `FLOWISE_API_KEY`

---

## یادداشت

گزارش کامل در:  
`DevOps Agent/Reports/2026-06-28-site-analysis-hmrbot.md`
