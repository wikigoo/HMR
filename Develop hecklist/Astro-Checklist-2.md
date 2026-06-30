---

# Astro + Cloudflare + FlowiseAI + Persian Blog — Site Health Checklist

---

## 1. HOSTING & DEPLOYMENT (Astro + Cloudflare)

**1.1**
- **Check**: `astro.config.mjs` uses `@astrojs/cloudflare` adapter with correct `output` mode (`'server'` or `'hybrid'`).
- **How**: Read `astro.config.mjs`; confirm `adapter: cloudflare()` is present and `output` matches the site's SSR/static split.
- **Pass criteria**: Adapter imported from `@astrojs/cloudflare`; `output` is `'server'` or `'hybrid'` (not `'static'` if any SSR routes exist, e.g., chatbot API proxy).
- **Severity**: CRITICAL

**1.2**
- **Check**: Cloudflare adapter targets Workers (not legacy Pages Functions). Adapter version ≥ 13 drops Pages support.
- **How**: Check `package.json` for `@astrojs/cloudflare` version; check `wrangler.jsonc` for `"main": "@astrojs/cloudflare/entrypoints/server"`.
- **Pass criteria**: `wrangler.jsonc` `main` field points to `@astrojs/cloudflare/entrypoints/server`; no legacy `dist/_worker.js/index.js` reference.
- **Severity**: CRITICAL [1](#0-0) 

**1.3**
- **Check**: Environment variables/secrets are not stored in `process.env` at runtime; accessed via `Astro.locals.runtime.env` or `context.locals.runtime.env`.
- **How**: `grep -r "process\.env" src/` — any hit in server-side code is a bug on Cloudflare Workers.
- **Pass criteria**: Zero runtime `process.env` references in SSR routes/middleware; secrets accessed through `locals.runtime.env`.
- **Severity**: CRITICAL [2](#0-1) 

**1.4**
- **Check**: `.dev.vars` file exists for local dev secrets and is listed in `.gitignore`.
- **How**: `ls -la .dev.vars`; `grep ".dev.vars" .gitignore`.
- **Pass criteria**: File exists locally; `.gitignore` contains `.dev.vars`; file is absent from the GitHub repo.
- **Severity**: CRITICAL

**1.5**
- **Check**: GitHub → Cloudflare deploy pipeline is active and the last deployment succeeded.
- **How**: Cloudflare Dashboard → Workers & Pages → Deployments tab; check last deployment status and build log.
- **Pass criteria**: Last deployment status is "Success"; build log shows no errors; `wrangler deploy` exit code 0.
- **Severity**: CRITICAL

**1.6**
- **Check**: `compatibility_date` in `wrangler.jsonc` is set and not stale (Cloudflare flags behavior changes by date).
- **How**: Read `wrangler.jsonc`; compare `compatibility_date` against Cloudflare's compatibility changelog.
- **Pass criteria**: `compatibility_date` is present and within the last 12 months, or intentionally pinned with a documented reason.
- **Severity**: HIGH

**1.7**
- **Check**: Cloudflare KV, D1, R2, or Durable Object bindings (if used) are declared in `wrangler.jsonc` and provisioned in the Cloudflare dashboard.
- **How**: Cross-reference `wrangler.jsonc` bindings with Cloudflare Dashboard → Workers → KV/D1/R2 namespaces.
- **Pass criteria**: Every binding in `wrangler.jsonc` has a corresponding provisioned resource; no "binding not found" errors in Worker logs.
- **Severity**: CRITICAL

**1.8**
- **Check**: Build output is clean — no leftover `_worker.js` directory for fully static builds.
- **How**: Run `astro build`; inspect `dist/` — if all routes are prerendered, `dist/_worker.js` should not exist.
- **Pass criteria**: Build completes without errors; output structure matches the expected static/server split.
- **Severity**: HIGH

**1.9**
- **Check**: `astro preview` runs against the workerd runtime (not Node) for accurate pre-deploy testing.
- **How**: Run `astro preview`; confirm the terminal output references `workerd` or Cloudflare's local runtime.
- **Pass criteria**: Preview server starts using workerd; chatbot proxy routes and dynamic blog routes respond correctly.
- **Severity**: MEDIUM

**1.10**
- **Check**: Cloudflare cache rules / `Cache-Control` headers are configured for static assets (`/_astro/*`).
- **How**: Cloudflare Dashboard → Caching → Cache Rules; or `curl -I https://example.com/_astro/main.abc123.css` and inspect `cf-cache-status`.
- **Pass criteria**: `cf-cache-status: HIT` on second request for `/_astro/*` assets; `Cache-Control: public, max-age=31536000, immutable` present.
- **Severity**: HIGH

---

## 2. DNS & NETWORK

**2.1**
- **Check**: DNS A/AAAA or CNAME records point to Cloudflare and are proxied (orange cloud).
- **How**: `dig +short example.com`; Cloudflare Dashboard → DNS → verify proxy status.
- **Pass criteria**: Records resolve to Cloudflare anycast IPs; proxy is enabled (not DNS-only).
- **Severity**: CRITICAL

**2.2**
- **Check**: SSL/TLS mode is set to "Full (strict)" in Cloudflare.
- **How**: Cloudflare Dashboard → SSL/TLS → Overview.
- **Pass criteria**: Mode is "Full (strict)"; not "Flexible" (which allows plaintext between Cloudflare and origin).
- **Severity**: CRITICAL

**2.3**
- **Check**: HTTPS is enforced — HTTP requests redirect to HTTPS.
- **How**: `curl -I http://example.com` — expect `301` or `308` to `https://`.
- **Pass criteria**: HTTP returns a redirect to HTTPS; no mixed-content warnings in browser console.
- **Severity**: CRITICAL

**2.4**
- **Check**: `www` and apex domain both resolve and redirect consistently (one canonical form).
- **How**: `curl -I https://www.example.com` and `curl -I https://example.com`; one should redirect to the other.
- **Pass criteria**: One form is canonical; the other 301-redirects; no redirect loop.
- **Severity**: HIGH

**2.5**
- **Check**: HSTS header is present with adequate `max-age`.
- **How**: `curl -I https://example.com | grep -i strict-transport`.
- **Pass criteria**: `Strict-Transport-Security: max-age=31536000` (or higher); optionally `includeSubDomains; preload`.
- **Severity**: HIGH

**2.6**
- **Check**: DNS propagation is complete globally.
- **How**: [dnschecker.org](https://dnschecker.org) or `dig @8.8.8.8 example.com` vs `dig @1.1.1.1 example.com`.
- **Pass criteria**: All checked resolvers return the same Cloudflare IPs.
- **Severity**: MEDIUM

---

## 3. PAGES & FUNCTIONALITY

**3.1**
- **Check**: All defined routes return HTTP 200 (no 404 or 500).
- **How**: `npx broken-link-checker https://example.com --recursive` or Screaming Frog crawl; check status codes.
- **Pass criteria**: Zero 4xx/5xx responses on any page route; custom `404.astro` and `500.astro` pages exist and render correctly.
- **Severity**: CRITICAL

**3.2**
- **Check**: All internal links resolve (no broken anchors or dead hrefs).
- **How**: Screaming Frog → "Response Codes" filter 4xx; or `linkinator https://example.com --recurse`.
- **Pass criteria**: Zero broken internal links.
- **Severity**: HIGH

**3.3**
- **Check**: Dynamic blog routes (e.g., `/blog/[slug]`) resolve for all published posts.
- **How**: Fetch the sitemap (`/sitemap-index.xml` → `/sitemap-0.xml`); request each blog URL; check for 200.
- **Pass criteria**: Every URL in the sitemap returns 200; no slug collision or missing route.
- **Severity**: CRITICAL

**3.4**
- **Check**: Static assets (images, fonts, CSS, JS) load without 404.
- **How**: Open browser DevTools → Network tab → filter by 4xx; or run Lighthouse and check "Avoid serving legacy JavaScript" / resource errors.
- **Pass criteria**: Zero asset 404s on any page.
- **Severity**: HIGH

**3.5**
- **Check**: Navigation menu links are correct and functional across all breakpoints.
- **How**: Manual click-through on desktop (1440px), tablet (768px), and mobile (375px) using browser DevTools device emulation.
- **Pass criteria**: All nav links navigate to the correct page; hamburger/mobile menu opens and closes; no overlapping elements.
- **Severity**: HIGH

---

## 4. CHATBOT (FlowiseAI / Agentflow)

**4.1**
- **Check**: FlowiseAI API endpoint is reachable from the Cloudflare Worker.
- **How**: `curl -X POST https://your-flowise-host/api/v1/prediction/<flow-id> -H "Content-Type: application/json" -d '{"question":"test"}'`; check HTTP status.
- **Pass criteria**: Returns HTTP 200 with a non-empty `text` field; response time < 30 s.
- **Severity**: CRITICAL

**4.2**
- **Check**: FlowiseAI API key is stored as a Cloudflare Worker secret (not hardcoded in source).
- **How**: `grep -r "FLOWISE\|flowise_key\|apiKey" src/`; check Cloudflare Dashboard → Workers → Settings → Variables.
- **Pass criteria**: No API key literal in source; key is a Cloudflare secret accessed via `locals.runtime.env`.
- **Severity**: CRITICAL

**4.3**
- **Check**: Chatbot send/receive round-trip works end-to-end in production.
- **How**: Open the chatbot UI; send a test message; verify a response appears within the timeout window.
- **Pass criteria**: Response rendered in UI within configured timeout; no console errors; no CORS errors.
- **Severity**: CRITICAL

**4.4**
- **Check**: Chatbot handles FlowiseAI backend timeout or LLM failure gracefully (no blank UI or unhandled exception).
- **How**: Temporarily point the API URL to an invalid endpoint; send a message; observe UI behavior.
- **Pass criteria**: User sees a localized error message (in Persian); UI remains interactive; no JS stack trace exposed.
- **Severity**: HIGH

**4.5**
- **Check**: Context/session retention works across multiple turns in the same conversation.
- **How**: Send a multi-turn conversation (e.g., "My name is Ali" → "What is my name?"); verify the bot recalls context.
- **Pass criteria**: Bot correctly references prior context; `sessionId` or equivalent is passed consistently.
- **Severity**: HIGH

**4.6**
- **Check**: Rate limiting is applied to the chatbot API proxy endpoint.
- **How**: Send 50+ rapid requests to the chatbot endpoint; check if Cloudflare WAF or application-level rate limiting triggers.
- **Pass criteria**: Requests beyond the threshold return 429; legitimate users are not permanently blocked.
- **Severity**: HIGH

**4.7**
- **Check**: Chatbot input is sanitized before forwarding to FlowiseAI (no prompt injection or XSS passthrough).
- **How**: Send `<script>alert(1)</script>` and `Ignore previous instructions and...` as chat input; inspect what is forwarded to the API and what is rendered.
- **Pass criteria**: HTML is escaped in the UI; prompt injection attempts do not alter system behavior; no raw HTML rendered.
- **Severity**: HIGH

**4.8**
- **Check**: CORS policy on the chatbot proxy endpoint restricts origins to the site's own domain.
- **How**: `curl -H "Origin: https://evil.com" https://example.com/api/chat` — inspect `Access-Control-Allow-Origin`.
- **Pass criteria**: Response does not include `Access-Control-Allow-Origin: *`; only the site's own origin is allowed.
- **Severity**: HIGH

---

## 5. AI BLOG PIPELINE (generation → publish → Telegram)

**5.1**
- **Check**: Blog post generation trigger (cron, webhook, or manual) fires reliably on schedule.
- **How**: Review the trigger mechanism (GitHub Actions cron, Cloudflare Cron Trigger, or external scheduler); check last 10 execution logs.
- **Pass criteria**: Trigger fires at the configured interval with zero missed runs in the last 7 days; failures are logged and alerted.
- **Severity**: CRITICAL

**5.2**
- **Check**: Generated posts are successfully committed/pushed to the GitHub repo and trigger a Cloudflare deployment.
- **How**: Check GitHub Actions run history or Cloudflare deployment log; verify the last auto-published post appears on the live site.
- **Pass criteria**: Each generation run results in a new commit; Cloudflare deployment completes successfully; post is live within the expected window.
- **Severity**: CRITICAL

**5.3**
- **Check**: Duplicate post prevention is in place (same topic/slug not published twice).
- **How**: Review the generation script for deduplication logic (slug registry, content hash, or title check); attempt to trigger generation for an already-published topic.
- **Pass criteria**: Duplicate detection fires; no duplicate slugs exist in the content directory; `git log` shows no repeated filenames.
- **Severity**: HIGH

**5.4**
- **Check**: A content QA gate exists before autonomous publish (human review step, automated quality score, or at minimum a length/coherence check).
- **How**: Review the pipeline code/workflow for any validation step between generation and commit.
- **Pass criteria**: At least one automated check (minimum word count, no empty sections, language detection confirms Persian) runs before publish. **Flag as HIGH risk if publishing is fully autonomous with zero validation.**
- **Severity**: HIGH

**5.5**
- **Check**: Telegram bot delivery succeeds for each new post.
- **How**: Check Telegram Bot API logs or the pipeline script's output; send a test post manually via `curl https://api.telegram.org/bot<TOKEN>/sendMessage`.
- **Pass criteria**: Each published post triggers a Telegram message; delivery confirmed by Telegram API `ok: true` response.
- **Severity**: CRITICAL

**5.6**
- **Check**: Telegram delivery has retry logic for transient API failures.
- **How**: Review the pipeline script for retry/backoff on Telegram API errors (429, 5xx).
- **Pass criteria**: Script retries at least 3 times with exponential backoff before marking delivery as failed; failures are logged.
- **Severity**: HIGH

**5.7**
- **Check**: Telegram message formatting is correct — Persian text renders RTL, links are clickable, no truncation.
- **How**: Open the Telegram channel/bot on mobile and desktop; verify the last 5 posts display correctly.
- **Pass criteria**: Text is right-aligned; links open the correct blog URL; no `[object Object]` or raw Markdown artifacts; messages are not cut off at Telegram's 4096-character limit.
- **Severity**: HIGH

**5.8**
- **Check**: Telegram Bot token is stored as a secret (not in source code or public config).
- **How**: `grep -r "bot[0-9]\{9\}:" .` (Telegram token pattern); check GitHub repo for accidental commits.
- **Pass criteria**: Token not found in any tracked file; stored as a GitHub Actions secret or Cloudflare Worker secret.
- **Severity**: CRITICAL

**5.9**
- **Check**: Pipeline failure (generation error, API outage, commit failure) sends an alert to the operator.
- **How**: Review GitHub Actions workflow for `on: failure` notification step or external alerting (email, Telegram, PagerDuty).
- **Pass criteria**: Any pipeline step failure triggers an operator notification within 15 minutes.
- **Severity**: HIGH

---

## 6. PERSIAN / RTL CORRECTNESS

**6.1**
- **Check**: `<html lang="fa" dir="rtl">` is set on every page.
- **How**: `curl https://example.com | grep -i '<html'`; or browser DevTools → Elements → inspect `<html>` tag.
- **Pass criteria**: `lang="fa"` and `dir="rtl"` both present on the root `<html>` element of every page template.
- **Severity**: CRITICAL

**6.2**
- **Check**: CSS layout does not break under RTL — no mirrored icons, misaligned flex/grid, or LTR-only padding.
- **How**: Open each page in Firefox with `about:config → layout.writing-mode.enabled = true`; or use Chrome DevTools → Rendering → Emulate RTL. Visually inspect navigation, cards, and blog post layout.
- **Pass criteria**: All layout elements mirror correctly; no text overflow; icons that should not mirror (e.g., play buttons) are excluded via `unicode-bidi` or CSS `transform: scaleX(-1)` only where appropriate.
- **Severity**: HIGH

**6.3**
- **Check**: Mixed LTR/RTL content (e.g., English code snippets, URLs, numbers inside Persian text) renders with correct bidi isolation.
- **How**: Open a blog post containing English words or URLs; inspect rendering in Chrome and Firefox.
- **Pass criteria**: LTR segments are wrapped in `<span dir="ltr">` or use CSS `unicode-bidi: isolate`; no character reordering artifacts.
- **Severity**: HIGH

**6.4**
- **Check**: Persian font (e.g., Vazirmatn, IRANSans, or similar) loads correctly with a system fallback.
- **How**: Browser DevTools → Network → filter "Font"; verify the Persian font file loads (200); disable the font in DevTools and confirm fallback renders legibly.
- **Pass criteria**: Persian font loads within 2 s; `font-display: swap` or `optional` is set; fallback is a system Persian/Arabic font (not a Latin-only font).
- **Severity**: HIGH

**6.5**
- **Check**: Persian numerals vs. Western Arabic numerals are used consistently per the site's style decision.
- **How**: Inspect blog post dates, pagination, and numbered lists; check CSS `font-variant-numeric` or JS number formatting.
- **Pass criteria**: Numeral style is consistent throughout; if Persian numerals (۱۲۳) are used, they appear everywhere; no mixed numeral styles on the same page.
- **Severity**: MEDIUM

**6.6**
- **Check**: Persian punctuation (،  ؟  ؛) is used correctly, not replaced with Latin equivalents.
- **How**: Read 5 AI-generated posts; search for `,` and `?` in Persian-language sentences.
- **Pass criteria**: Persian comma (،) and question mark (؟) used in Persian text; no Latin punctuation in Persian prose.
- **Severity**: MEDIUM

---

## 7. CONTENT QUALITY (AI-generated Persian)

**7.1**
- **Check**: Generated posts are grammatically fluent Persian (no machine-translation artifacts, word salad, or code-switching).
- **How**: Have a native Persian speaker review the last 5 posts; or run text through a Persian grammar checker (e.g., Virastyar).
- **Pass criteria**: No obvious grammatical errors; sentences are natural; no unexplained English words mid-sentence.
- **Severity**: HIGH

**7.2**
- **Check**: Posts do not contain hallucinated facts (fabricated statistics, fake citations, incorrect dates).
- **How**: Spot-check factual claims in 5 posts against authoritative sources; verify any cited URLs exist and match the claim.
- **Pass criteria**: All verifiable facts are accurate; no dead citation links; no fabricated statistics.
- **Severity**: HIGH

**7.3**
- **Check**: No post is empty, truncated, or contains only a title/heading with no body.
- **How**: `grep -rL "." src/content/blog/*.md` (files with no content); or script: check word count > 200 for each post file.
- **Pass criteria**: Every published post has ≥ 200 words of body content; no placeholder text (e.g., "Lorem ipsum" or `[CONTENT HERE]`).
- **Severity**: CRITICAL

**7.4**
- **Check**: Post frontmatter is complete and valid — `title`, `description`, `pubDate`, `slug` (or equivalent) are all present.
- **How**: `astro check` or a custom script that reads each content file's frontmatter and validates required fields.
- **Pass criteria**: Zero posts with missing required frontmatter fields; `pubDate` is a valid ISO date; `description` is non-empty.
- **Severity**: HIGH

**7.5**
- **Check**: Formatting is consistent across posts — headings, lists, and code blocks render correctly in Markdown.
- **How**: Open 5 posts in the browser; inspect rendered HTML for correct `<h2>`, `<ul>`, `<pre>` elements.
- **Pass criteria**: No raw Markdown symbols visible in rendered output; heading hierarchy is logical (no skipped levels); code blocks have syntax highlighting if configured.
- **Severity**: MEDIUM

---

## 8. SEO & DISCOVERABILITY

**8.1**
- **Check**: Every page has a unique, non-empty `<title>` and `<meta name="description">` in Persian.
- **How**: Screaming Frog → "Page Titles" and "Meta Description" tabs; filter for duplicates or missing values.
- **Pass criteria**: Zero pages with missing or duplicate titles/descriptions; title length 30–60 chars; description 120–160 chars.
- **Severity**: HIGH

**8.2**
- **Check**: Open Graph tags (`og:title`, `og:description`, `og:image`, `og:url`) are present on all pages.
- **How**: `curl https://example.com/blog/some-post | grep 'og:'`; or use [opengraph.xyz](https://www.opengraph.xyz).
- **Pass criteria**: All four OG tags present; `og:image` resolves to a valid image URL; image dimensions ≥ 1200×630 px.
- **Severity**: HIGH [3](#0-2) 

**8.3**
- **Check**: `sitemap-index.xml` and `sitemap-0.xml` are generated and accessible.
- **How**: `curl https://example.com/sitemap-index.xml`; validate XML structure; check all blog post URLs are included.
- **Pass criteria**: Sitemap returns 200; all published post URLs present; `<lastmod>` dates are accurate; sitemap is referenced in `robots.txt`.
- **Severity**: HIGH

**8.4**
- **Check**: `robots.txt` exists, allows crawling of public pages, and disallows any private/API routes.
- **How**: `curl https://example.com/robots.txt`; verify `Sitemap:` directive points to `sitemap-index.xml`.
- **Pass criteria**: `robots.txt` returns 200; `User-agent: *` with `Allow: /`; API/chatbot proxy routes are `Disallow`ed; `Sitemap:` line present.
- **Severity**: HIGH

**8.5**
- **Check**: Canonical tags are present and correct on all pages (no self-referencing canonicals pointing to wrong URLs).
- **How**: `curl https://example.com/blog/some-post | grep 'canonical'`; verify the `href` matches the page's actual URL.
- **Pass criteria**: Every page has exactly one `<link rel="canonical">`; href matches the page URL exactly (including trailing slash policy).
- **Severity**: HIGH

**8.6**
- **Check**: Structured data (JSON-LD `Article` or `BlogPosting`) is present on blog post pages.
- **How**: [Google Rich Results Test](https://search.google.com/test/rich-results) for a blog post URL; or `curl https://example.com/blog/post | grep 'application/ld+json'`.
- **Pass criteria**: Valid `BlogPosting` schema with `headline`, `datePublished`, `author`, `inLanguage: "fa"` fields; no validation errors.
- **Severity**: MEDIUM

**8.7**
- **Check**: Pages are indexable — no `<meta name="robots" content="noindex">` accidentally left on production pages.
- **How**: Screaming Frog → "Directives" → filter "noindex"; or `curl https://example.com | grep 'noindex'`.
- **Pass criteria**: Zero production pages with `noindex`; Cloudflare headers do not add `X-Robots-Tag: noindex`.
- **Severity**: HIGH

**8.8**
- **Check**: Persian-specific SEO — `hreflang` is set if any content exists in multiple languages; `lang="fa"` on `<html>` signals language to search engines.
- **How**: Check `<head>` for `<link rel="alternate" hreflang="fa">` if multilingual; verify Google Search Console shows the site indexed under Persian language.
- **Pass criteria**: `lang="fa"` on `<html>`; if multilingual, `hreflang` tags are present and reciprocal.
- **Severity**: MEDIUM

---

## 9. PERFORMANCE & SPEED

**9.1**
- **Check**: Lighthouse performance score ≥ 80 on mobile for the homepage and a representative blog post.
- **How**: `npx lighthouse https://example.com --output=json --preset=mobile | jq '.categories.performance.score'`; or PageSpeed Insights.
- **Pass criteria**: Score ≥ 80 on mobile; ≥ 90 on desktop.
- **Severity**: HIGH

**9.2**
- **Check**: Core Web Vitals — LCP < 2.5 s, CLS < 0.1, INP < 200 ms.
- **How**: Chrome DevTools → Lighthouse → "Core Web Vitals"; or CrUX data in Google Search Console.
- **Pass criteria**: All three metrics in the "Good" range.
- **Severity**: HIGH

**9.3**
- **Check**: Images use Astro's `<Image>` or `<Picture>` component (not raw `<img>`) for automatic optimization.
- **How**: Astro Dev Toolbar audit (flags `img` tags not using the Image component); or `grep -r '<img ' src/` for non-optimized images.
- **Pass criteria**: All content images use `<Image>`/`<Picture>`; `alt` attributes present on all images; `loading="lazy"` on below-fold images.
- **Severity**: HIGH [4](#0-3) 

**9.4**
- **Check**: Cloudflare image service is configured for runtime image optimization (if using SSR image transforms).
- **How**: Check `astro.config.mjs` for `imageService: { build: 'compile', runtime: 'cloudflare-binding' }` in the Cloudflare adapter options.
- **Pass criteria**: Image service config matches the deployment mode; no Sharp runtime errors in Worker logs.
- **Severity**: MEDIUM [5](#0-4) 

**9.5**
- **Check**: Astro islands (`client:load`, `client:idle`, `client:visible`) are used only where interactivity is required; static components are not hydrated unnecessarily.
- **How**: `grep -r "client:" src/` — review each usage; confirm chatbot widget uses `client:load` and non-interactive components have no `client:` directive.
- **Pass criteria**: No `client:load` on components that don't need immediate interactivity; chatbot and interactive widgets correctly hydrated.
- **Severity**: MEDIUM

**9.6**
- **Check**: Cloudflare cache hit ratio for static assets is ≥ 90%.
- **How**: Cloudflare Dashboard → Analytics → Cache → "Cache Hit Rate" for the last 7 days.
- **Pass criteria**: Cache hit rate ≥ 90% for static asset requests; `/_astro/*` paths show `cf-cache-status: HIT`.
- **Severity**: MEDIUM

**9.7**
- **Check**: Total page weight (HTML + CSS + JS + fonts) is ≤ 1 MB for the homepage.
- **How**: Chrome DevTools → Network → disable cache → reload; check "Transferred" total.
- **Pass criteria**: Homepage transferred size ≤ 1 MB; no single JS bundle > 300 KB uncompressed.
- **Severity**: MEDIUM

---

## 10. UI / BRANDING

**10.1**
- **Check**: Logo and favicon render correctly at all sizes (16px, 32px, 180px Apple touch icon).
- **How**: Open the site; inspect `<link rel="icon">` and `<link rel="apple-touch-icon">` in `<head>`; verify files exist and load.
- **Pass criteria**: Favicon visible in browser tab; Apple touch icon present; logo not pixelated or stretched.
- **Severity**: MEDIUM

**10.2**
- **Check**: Color contrast meets WCAG AA — text contrast ratio ≥ 4.5:1 (normal text) and ≥ 3:1 (large text).
- **How**: Lighthouse → Accessibility → "Background and foreground colors do not have a sufficient contrast ratio"; or axe DevTools browser extension.
- **Pass criteria**: Zero contrast failures in Lighthouse accessibility audit.
- **Severity**: HIGH

**10.3**
- **Check**: All images have non-empty, descriptive `alt` text in Persian.
- **How**: Lighthouse → Accessibility → "Image elements do not have [alt] attributes"; or `grep -r 'alt=""' src/`.
- **Pass criteria**: Zero images with missing or empty `alt`; decorative images use `alt=""` with `role="presentation"`.
- **Severity**: HIGH

**10.4**
- **Check**: Responsive design is correct at 375px (mobile), 768px (tablet), and 1440px (desktop).
- **How**: Chrome DevTools → Device Toolbar; test each breakpoint on homepage, blog list, blog post, and chatbot page.
- **Pass criteria**: No horizontal scroll; no overlapping elements; text is legible; chatbot widget is usable on mobile.
- **Severity**: HIGH

**10.5**
- **Check**: Keyboard navigation and focus indicators are functional (tab order is logical; focus ring is visible).
- **How**: Tab through the page without a mouse; verify focus ring is visible on all interactive elements.
- **Pass criteria**: All interactive elements reachable by keyboard; focus ring visible; no focus traps outside modals.
- **Severity**: MEDIUM

**10.6**
- **Check**: Visual consistency — typography scale, spacing, and color palette are uniform across all pages.
- **How**: Compare homepage, blog list, blog post, and chatbot page side-by-side; check for inconsistent font sizes or button styles.
- **Pass criteria**: No rogue font sizes or colors; design tokens (CSS variables) used consistently.
- **Severity**: LOW

---

## 11. SECURITY

**11.1**
- **Check**: No API keys, tokens, or secrets are present in the public GitHub repository (current or historical commits).
- **How**: `git log -p | grep -E "(api_key|token|secret|password|FLOWISE|bot[0-9]{9}:)"` on the full history; or use `truffleHog` / `gitleaks`.
- **Pass criteria**: Zero secret patterns found in any commit; if found, rotate all exposed credentials immediately.
- **Severity**: CRITICAL

**11.2**
- **Check**: Content Security Policy (CSP) header is set and restricts script/style sources.
- **How**: `curl -I https://example.com | grep -i content-security-policy`.
- **Pass criteria**: CSP header present; `script-src` does not include `'unsafe-eval'` or `'unsafe-inline'` without a nonce/hash; FlowiseAI API origin is explicitly allowed in `connect-src`.
- **Severity**: HIGH

**11.3**
- **Check**: Cloudflare WAF is enabled with at least the "Managed Rules" ruleset active.
- **How**: Cloudflare Dashboard → Security → WAF → Managed Rules.
- **Pass criteria**: Managed Rules are enabled; Bot Fight Mode or Super Bot Fight Mode is active; no rules are globally disabled.
- **Severity**: HIGH

**11.4**
- **Check**: Security headers are present: `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`.
- **How**: `curl -I https://example.com | grep -iE "(x-content-type|x-frame|referrer-policy)"`.
- **Pass criteria**: `X-Content-Type-Options: nosniff`; `X-Frame-Options: SAMEORIGIN` (or CSP `frame-ancestors`); `Referrer-Policy: strict-origin-when-cross-origin`.
- **Severity**: HIGH

**11.5**
- **Check**: Chatbot input length is capped server-side to prevent oversized payloads.
- **How**: Send a 100,000-character string to the chatbot API endpoint; check response.
- **Pass criteria**: Server rejects payloads above the configured limit with 413 or 400; no Worker timeout or memory spike.
- **Severity**: HIGH

**11.6**
- **Check**: Cloudflare DDoS protection is enabled (automatic for all zones, but verify it is not disabled).
- **How**: Cloudflare Dashboard → Security → DDoS → verify "HTTP DDoS attack protection" is enabled.
- **Pass criteria**: DDoS protection is in "Enabled" state; no custom rules that bypass it for the chatbot endpoint.
- **Severity**: HIGH

---

## 12. BACKUP & RECOVERY

**12.1**
- **Check**: The GitHub repository is the source of truth and is not the only copy — at least one off-site backup exists.
- **How**: Verify a secondary remote (e.g., GitLab mirror, local bare clone, or GitHub repo export) exists and is current.
- **Pass criteria**: At least one off-site backup updated within the last 7 days.
- **Severity**: HIGH

**12.2**
- **Check**: FlowiseAI flows and knowledge bases are exported and backed up.
- **How**: FlowiseAI UI → Export Flow (JSON); verify the export file is stored in a versioned location (S3, GitHub private repo, etc.).
- **Pass criteria**: Flow export exists; last backup is ≤ 7 days old; export can be re-imported to a fresh FlowiseAI instance.
- **Severity**: HIGH

**12.3**
- **Check**: Cloudflare KV/D1 data (if used for sessions or content) is backed up.
- **How**: Use `wrangler kv:bulk export` or `wrangler d1 export` to dump data; verify the dump is stored off-site.
- **Pass criteria**: Backup exists; last backup ≤ 24 hours old for frequently-written data.
- **Severity**: HIGH

**12.4**
- **Check**: A restore procedure has been tested — not just backed up.
- **How**: Document the last date a restore was performed; attempt a restore to a staging environment.
- **Pass criteria**: Restore procedure is documented; a test restore was completed within the last 90 days.
- **Severity**: HIGH

**12.5**
- **Check**: Cloudflare Worker secrets and environment variable values are documented in a secure vault (not just in the dashboard).
- **How**: Check a secrets manager (1Password, Bitwarden, HashiCorp Vault, etc.) for all production secrets.
- **Pass criteria**: All secrets documented in a vault; no secret exists only in the Cloudflare dashboard with no other record.
- **Severity**: HIGH

---

## 13. OPTIMIZATION & TECHNICAL DEBT

**13.1**
- **Check**: Astro and all integrations (`@astrojs/cloudflare`, `@astrojs/sitemap`, etc.) are on current major versions; check for deprecated APIs.
- **How**: `npx npm-check-updates`; compare installed versions against the latest on npm; review each package's CHANGELOG for deprecation notices.
- **Pass criteria**: No packages more than one major version behind; no deprecated APIs in use (e.g., `cloudflareModules`, `mode: 'directory'`, `functionPerRoute`).
- **Severity**: MEDIUM [6](#0-5) 

**13.2**
- **Check**: `process.env` is not used anywhere in server-side code (replaced by `astro:env` or `locals.runtime.env`).
- **How**: `grep -rn "process\.env" src/` — any hit in `.astro`, `.ts`, or `.js` files used server-side.
- **Pass criteria**: Zero `process.env` references in server-side code.
- **Severity**: HIGH

**13.3**
- **Check**: Deprecated `wrangler.toml` is not used alongside `wrangler.jsonc` (only one config file should exist).
- **How**: `ls wrangler.*` in the project root.
- **Pass criteria**: Exactly one Wrangler config file exists; no duplicate `wrangler.toml` + `wrangler.jsonc`.
- **Severity**: MEDIUM

**13.4**
- **Check**: Astro's `astro:env` module is used for type-safe environment variable access (available since Astro 5).
- **How**: Check `astro.config.mjs` for `env` configuration; check if `import { ... } from 'astro:env/server'` is used.
- **Pass criteria**: Secrets accessed via `astro:env` with schema validation; no untyped `process.env` fallbacks.
- **Severity**: LOW

**13.5**
- **Check**: Cloudflare's `nodejs_compat` compatibility flag is enabled if any Node.js built-in APIs are used.
- **How**: `grep -r "node:" src/`; check `wrangler.jsonc` for `compatibility_flags: ["nodejs_compat"]`.
- **Pass criteria**: If Node.js APIs are used, `nodejs_compat` flag is present; no runtime "not implemented" errors in Worker logs.
- **Severity**: HIGH

**13.6**
- **Check**: Dead or unused Astro integrations are not installed (increases build time and bundle size).
- **How**: Compare `astro.config.mjs` `integrations` array against `package.json` dependencies; check for integrations listed in `package.json` but not in the config.
- **Pass criteria**: No unused integrations installed; no deprecated integrations (e.g., `@astrojs/prefetch` — replaced by built-in `prefetch`).
- **Severity**: LOW

**13.7**
- **Check**: Evaluate Cloudflare Workers AI as a potential replacement or fallback for the FlowiseAI/LLM backend (reviewer-run assessment).
- **How**: Review Cloudflare Workers AI model catalog; compare latency and cost against current FlowiseAI setup.
- **Pass criteria**: This is an evaluation item — document findings; no pass/fail.
- **Severity**: LOW

---

## 14. MONITORING & ERROR HANDLING

**14.1**
- **Check**: Uptime monitoring is configured for the site's homepage and chatbot endpoint.
- **How**: Check for an active monitor in UptimeRobot, Better Uptime, Cloudflare Health Checks, or equivalent; verify alert contacts are current.
- **Pass criteria**: Monitor checks at ≤ 5-minute intervals; alert fires within 10 minutes of downtime; alert goes to an active contact.
- **Severity**: CRITICAL

**14.2**
- **Check**: Cloudflare Worker error logs are accessible and reviewed regularly.
- **How**: Cloudflare Dashboard → Workers → your Worker → Logs (real-time) or Logpush to R2/S3.
- **Pass criteria**: Logs are enabled; last 24 hours show no unhandled exceptions; error rate < 0.1%.
- **Severity**: HIGH

**14.3**
- **Check**: Blog pipeline failures (generation, commit, Telegram delivery) trigger an operator alert.
- **How**: Review GitHub Actions workflow for `on: failure` notification; or check if a monitoring webhook is configured.
- **Pass criteria**: Any pipeline failure sends an alert (email, Telegram, Slack) within 15 minutes.
- **Severity**: HIGH

**14.4**
- **Check**: The site has a custom `404.astro` and `500.astro` page in Persian with navigation back to the homepage.
- **How**: Request `https://example.com/nonexistent-page` and `https://example.com/500` (if a test route exists); inspect the rendered page.
- **Pass criteria**: Custom error pages render in Persian; include a link back to the homepage; do not expose stack traces or internal paths.
- **Severity**: HIGH

**14.5**
- **Check**: Chatbot errors are logged server-side with enough context to diagnose (request ID, error type, timestamp).
- **How**: Trigger a chatbot error (invalid input, backend timeout); check Worker logs for a structured error entry.
- **Pass criteria**: Error log entry includes timestamp, error type, and a request identifier; no sensitive user input is logged.
- **Severity**: MEDIUM

**14.6**
- **Check**: Cloudflare Analytics or a privacy-respecting alternative (Plausible, Fathom) is configured to track traffic and error rates.
- **How**: Cloudflare Dashboard → Analytics → Web Analytics; or check for analytics script in page `<head>`.
- **Pass criteria**: Analytics data is being collected; dashboard shows traffic for the last 7 days; no analytics script blocked by CSP.
- **Severity**: MEDIUM

---

## PRIORITIZED SUMMARY — CRITICAL & HIGH ITEMS (First-Pass Order)

**CRITICAL**

| # | Item | Category |
|---|------|----------|
| C1 | Cloudflare adapter targets Workers; `wrangler.jsonc` uses correct entrypoint | Hosting |
| C2 | No API keys/tokens in GitHub repo (current or historical) | Security |
| C3 | Telegram Bot token stored as secret, not in source | Blog Pipeline |
| C4 | FlowiseAI API key stored as Cloudflare Worker secret | Chatbot |
| C5 | All routes return 200; no 404/500 on live pages | Pages |
| C6 | Blog post generation trigger fires reliably; last 7 days zero missed runs | Blog Pipeline |
| C7 | Telegram delivery succeeds for each new post | Blog Pipeline |
| C8 | No published post is empty or truncated (≥ 200 words) | Content Quality |
| C9 | Runtime env vars accessed via `locals.runtime.env`, not `process.env` | Hosting |
| C10 | Uptime monitor active with ≤ 5-min interval and alert contacts | Monitoring |
| C11 | HTTPS enforced; SSL mode is "Full (strict)" | DNS & Network |
| C12 | KV/D1/R2 bindings provisioned and matching `wrangler.jsonc` | Hosting |

**HIGH**

| # | Item | Category |
|---|------|----------|
| H1 | `<html lang="fa" dir="rtl">` on every page | RTL |
| H2 | Persian font loads with `font-display: swap` and system fallback | RTL |
| H3 | RTL layout correct — no mirrored icons, no broken flex/grid | RTL |
| H4 | Chatbot handles backend timeout gracefully with Persian error message | Chatbot |
| H5 | Chatbot rate limiting active (429 on abuse) | Chatbot |
| H6 | Chatbot input sanitized (XSS escape + prompt injection mitigation) | Chatbot / Security |
| H7 | CORS on chatbot proxy restricted to own domain | Chatbot / Security |
| H8 | Content QA gate before autonomous publish | Blog Pipeline |
| H9 | Telegram retry logic (3× with backoff) | Blog Pipeline |
| H10 | Duplicate post prevention in pipeline | Blog Pipeline |
| H11 | Pipeline failure triggers operator alert within 15 min | Blog Pipeline / Monitoring |
| H12 | AI-generated posts: no hallucinated facts; native speaker review | Content Quality |
| H13 | Post frontmatter complete (`title`, `description`, `pubDate`) | Content Quality |
| H14 | Sitemap generated and all blog URLs included | SEO |
| H15 | `robots.txt` present with `Sitemap:` directive | SEO |
| H16 | Canonical tags correct on all pages | SEO |
| H17 | No `noindex` on production pages | SEO |
| H18 | OG tags present on all pages | SEO |
| H19 | Lighthouse mobile score ≥ 80; Core Web Vitals in "Good" range | Performance |
| H20 | All images use `<Image>`/`<Picture>` with `alt` text | Performance / Accessibility |
| H21 | Color contrast ≥ 4.5:1 (WCAG AA) | UI |
| H22 | Responsive design correct at 375/768/1440 px | UI |
| H23 | CSP header present; no `unsafe-eval` without nonce | Security |
| H24 | Cloudflare WAF Managed Rules enabled | Security |
| H25 | Security headers: `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy` | Security |
| H26 | Chatbot payload size capped server-side | Security |
| H27 | FlowiseAI flows backed up (≤ 7 days old) | Backup |
| H28 | Restore procedure tested within last 90 days | Backup |
| H29 | Worker error logs enabled; error rate < 0.1% | Monitoring |
| H30 | Custom 404/500 pages in Persian with homepage link | Monitoring |
| H31 | `nodejs_compat` flag set if Node.js APIs used | Tech Debt |
| H32 | `compatibility_date` in `wrangler.jsonc` is current | Hosting |
| H33 | Cloudflare cache hit rate ≥ 90% for static assets | Performance |

### Citations

**File:** packages/integrations/cloudflare/CHANGELOG.md (L695-697)
```markdown
- [#15480](https://github.com/withastro/astro/pull/15480) [`e118214`](https://github.com/withastro/astro/commit/e118214eeaaa27384528c882af8b68e8daa61e2c) Thanks [@alexanderniebuhr](https://github.com/alexanderniebuhr)! - Drops official support for Cloudflare Pages in favor of Cloudflare Workers

  The Astro Cloudflare adapter now only supports deployment to Cloudflare Workers by default in order to comply with Cloudflare's recommendations for new projects. If you are currently deploying to Cloudflare Pages, consider [migrating to Workers by following the Cloudflare guide](https://developers.cloudflare.com/workers/static-assets/migration-guides/migrate-from-pages/) for an optimal experience and full feature support.
```

**File:** packages/integrations/cloudflare/CHANGELOG.md (L701-716)
```markdown
- [#15435](https://github.com/withastro/astro/pull/15435) [`957b9fe`](https://github.com/withastro/astro/commit/957b9fe2d887a365c55c6e87f0c67c10beb60d1b) Thanks [@rururux](https://github.com/rururux)! - Adds support for configuring the image service as an object with separate `build` and `runtime` options

  It is now possible to set both a build-time and runtime service independently. Currently, `'compile'` is the only available build time option. The supported runtime options are `'passthrough'` (default) and `'cloudflare-binding'`:

  ```js title="astro.config.mjs" ins={6}
  import { defineConfig } from 'astro/config';
  import cloudflare from '@astrojs/cloudflare';

  export default defineConfig({
    adapter: cloudflare({
      imageService: { build: 'compile', runtime: 'cloudflare-binding' },
    }),
  });
  ```

  See the [Cloudflare adapter `imageService` docs](https://docs.astro.build/en/guides/integrations-guide/cloudflare/#imageservice) for more information about configuring your image service.
```

**File:** packages/integrations/cloudflare/CHANGELOG.md (L1027-1043)
```markdown
- [#15345](https://github.com/withastro/astro/pull/15345) [`840fbf9`](https://github.com/withastro/astro/commit/840fbf9e4abc7f847e23da8d67904ffde4d95fff) Thanks [@matthewp](https://github.com/matthewp)! - Removes the `cloudflareModules` adapter option

  The `cloudflareModules` option has been removed because it is no longer necessary. Cloudflare natively supports importing `.sql`, `.wasm`, and other module types.

  #### What should I do?

  Remove the `cloudflareModules` option from your Cloudflare adapter configuration if you were using it:

  ```diff
  import cloudflare from '@astrojs/cloudflare';

  export default defineConfig({
    adapter: cloudflare({
  -   cloudflareModules: true
    })
  });
  ```
```

**File:** packages/integrations/cloudflare/CHANGELOG.md (L2434-2440)
```markdown
  ### process.env

  In the old version of the adapter we used to expose all the environment variables to `process.env`. This is no longer the case, as it was unsafe. If you need to use environment variables, you need to use either `Astro.locals.runtime.env` or `context.locals.runtime.env`. There is no way to access the environment variables directly from `process.env` or in the global scope.

  If you need to access the environment variables in global scope, you should refactor your code to pass the environment variables as arguments to your function or file.

  If you rely on any third library that uses `process.env`, please open an issue and we can investigate what the best way to handle this is.
```

**File:** packages/astro/e2e/fixtures/actions-blog/src/components/BaseHead.astro (L30-42)
```text
<!-- Open Graph / Facebook -->
<meta property="og:type" content="website" />
<meta property="og:url" content={Astro.url} />
<meta property="og:title" content={title} />
<meta property="og:description" content={description} />
<meta property="og:image" content={new URL(image, Astro.url)} />

<!-- Twitter -->
<meta property="twitter:card" content="summary_large_image" />
<meta property="twitter:url" content={Astro.url} />
<meta property="twitter:title" content={title} />
<meta property="twitter:description" content={description} />
<meta property="twitter:image" content={new URL(image, Astro.url)} />
```

**File:** packages/astro/src/runtime/client/dev-toolbar/apps/audit/rules/perf.ts (L6-32)
```typescript
export const perf: AuditRuleWithSelector[] = [
	{
		code: 'perf-use-image-component',
		title: 'Use the Image component',
		message: 'This image could be replaced with the Image component to improve performance.',
		selector: 'img:not([data-image-component])',
		async match(element) {
			// Skip images inside framework components (React, Vue, Svelte, etc.)
			// These are wrapped in astro-island and can't directly use Astro's Image component
			if (element.closest('astro-island')) return false;

			const src = element.getAttribute('src');
			if (!src) return false;

			// Don't match data URIs, they're typically used for specific use-cases that the image component doesn't help with
			if (src.startsWith('data:')) return false;

			// Ignore images that are smaller than 20KB, most of the time the image component won't really help with these, or they're used for specific use-cases (pixel tracking, etc.)
			// Ignore this test for remote images for now, fetching them can be very slow and possibly dangerous
			if (!EXTERNAL_URL_REGEX.test(src)) {
				const imageData = await fetch(src).then((response) => response.blob());
				if (imageData.size < 20480) return false;
			}

			return true;
		},
	},
```
