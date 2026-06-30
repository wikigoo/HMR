---

# COMPREHENSIVE HEALTH & OPTIMIZATION CHECKLIST
## Astro + Cloudflare Tunnel + FlowiseAI + Persian AI Blog + Telegram

---

## 1. CLOUDFLARE TUNNEL & ORIGIN

**1.1**
- **Check**: `cloudflared` daemon is running and managed as a system service
- **How**: `systemctl status cloudflared.service` (Linux/systemd); verify `Active: active (running)` and `Type=notify`
- **Pass criteria**: Service is active, no failed state; systemd unit file at `/etc/systemd/system/cloudflared.service` exists with `Restart=on-failure` and `RestartSec=5s`
- **Severity**: CRITICAL [1](#0-0) 

**1.2**
- **Check**: Tunnel is connected with all 4 HA connections to Cloudflare edge
- **How**: `cloudflared tunnel info <TUNNEL-NAME>` or query the local metrics endpoint: `curl http://localhost:20241/metrics | grep cloudflared_tunnel_server_locations`; expect 4 active connections
- **Pass criteria**: `cloudflared_tunnel_server_locations` gauge shows 4 connections each with value `1`; no `tunnel_register_fail` counter incrementing
- **Severity**: CRITICAL [2](#0-1) 

**1.3**
- **Check**: Ingress rules in `config.yml` are syntactically valid and route correctly
- **How**: `cloudflared tunnel --config /etc/cloudflared/config.yml ingress validate`; then test specific URLs: `cloudflared tunnel --config /etc/cloudflared/config.yml ingress rule https://yourdomain.com/api/flowise`
- **Pass criteria**: `validate` exits 0 with no errors; `rule` command returns the expected backend service for each hostname/path combination; last rule is a catch-all
- **Severity**: CRITICAL [3](#0-2) [4](#0-3) 

**1.4**
- **Check**: Origin exposes NO public-facing ports (all traffic flows exclusively through the tunnel)
- **How**: From an external host: `nmap -Pn -p 80,443,8080,3000 <ORIGIN-PUBLIC-IP>`; from origin: `ss -tlnp` to list listening sockets
- **Pass criteria**: `nmap` shows all ports filtered/closed from the internet; `ss` shows services bound only to `127.0.0.1` or private interfaces, not `0.0.0.0`
- **Severity**: CRITICAL [5](#0-4) 

**1.5**
- **Check**: `cloudflared` version is within Cloudflare's supported window (within 1 year of latest release)
- **How**: `cloudflared --version`; compare against latest release at `https://github.com/cloudflare/cloudflared/releases`; version format is `YYYY.M.P`
- **Pass criteria**: Installed version is dated within 12 months of the most recent release; no deprecation warnings in logs
- **Severity**: HIGH [6](#0-5) 

**1.6**
- **Check**: Auto-update timer is active and functional
- **How**: `systemctl status cloudflared-update.timer`; verify `cloudflared-update.service` exists; check that update script handles exit code `11` (updated) vs `10` (failed) correctly
- **Pass criteria**: Timer is active with `OnCalendar=daily`; update service restarts `cloudflared` on exit code `11`
- **Severity**: MEDIUM [7](#0-6) [8](#0-7) 

**1.7**
- **Check**: Tunnel readiness endpoint responds healthy
- **How**: `curl http://localhost:20241/ready` (or whichever port in range 20241–20245 is bound)
- **Pass criteria**: HTTP 200 response; if not 200, tunnel is not ready to serve traffic
- **Severity**: CRITICAL [9](#0-8) 

**1.8**
- **Check**: Transport protocol is QUIC (preferred) with HTTP/2 fallback configured
- **How**: `curl http://localhost:20241/metrics | grep cloudflared_tunnel_`; check logs for `Initial protocol quic`; run `cloudflared tunnel diag` and inspect `cli-configuration.json` for `protocol` field
- **Pass criteria**: QUIC is the active protocol; HTTP/2 fallback is not permanently stuck as primary
- **Severity**: MEDIUM [10](#0-9) 

**1.9**
- **Check**: Diagnostic bundle can be collected without errors
- **How**: `cloudflared tunnel diag` — produces a `cloudflared-diag-*.zip` containing `tunnelstate.json`, `metrics.txt`, `systeminformation.json`, `network.json`, `cloudflared_logs.txt`, `cli-configuration.json`, `configuration.json`
- **Pass criteria**: Command completes with "Diagnostic completed" (not "completed with one or more errors"); all expected files present in zip
- **Severity**: LOW [11](#0-10) [12](#0-11) 

**1.10**
- **Check**: Tunnel credentials file (`<tunnel-id>.json`) and `cert.pem` are secured with restrictive permissions
- **How**: `ls -la /etc/cloudflared/` — check file permissions on `cert.pem` and `*.json`
- **Pass criteria**: Files are readable only by the `cloudflared` service user (mode `600` or `400`); not world-readable
- **Severity**: HIGH [13](#0-12) 

---

## 2. HOSTING & DEPLOYMENT (Astro + GitHub)

**2.1**
- **Check**: Astro build completes without errors or warnings on the origin
- **How**: Trigger a manual build: `npm run build` (or `pnpm build`); inspect exit code and stdout/stderr for errors
- **Pass criteria**: Exit code 0; no `[ERROR]` or unhandled exception lines; `dist/` directory populated with expected assets
- **Severity**: CRITICAL

**2.2**
- **Check**: GitHub → origin deployment pipeline executes reliably on push to main branch
- **How**: Review GitHub Actions workflow run history; check the deploy step logs; verify the deploy script (SSH, rsync, or webhook) completes successfully
- **Pass criteria**: Last 5 deployments all succeeded; no stuck or timed-out runs; deployment completes and origin serves updated content within expected window
- **Severity**: CRITICAL

**2.3**
- **Check**: All required environment variables and secrets are set in the deployment environment
- **How**: In GitHub Actions: Settings → Secrets and variables → Actions; on origin: `printenv | grep -E 'FLOWISE|TELEGRAM|OPENAI|API'` (as the service user, not root)
- **Pass criteria**: All secrets referenced in workflow YAML and `.env` are present; no `undefined` or empty values at runtime; no secrets hardcoded in repo files (`git grep -i 'api_key\|secret\|token' -- '*.env*' '*.js' '*.ts' '*.astro'`)
- **Severity**: CRITICAL

**2.4**
- **Check**: Astro `output` mode and adapter are correctly configured for the deployment target
- **How**: Inspect `astro.config.mjs`; verify `output: 'server'` or `'hybrid'` if SSR is used; confirm the correct adapter (e.g., `@astrojs/node`) is installed and configured
- **Pass criteria**: Adapter matches the runtime on origin; no mismatch between static/SSR expectations and actual serving behavior
- **Severity**: HIGH

**2.5**
- **Check**: Build cache and `node_modules` are not stale or corrupted between deployments
- **How**: Check if deployment script runs `npm ci` (not `npm install`); verify `.astro/` cache directory is cleared or managed correctly between builds
- **Pass criteria**: `npm ci` used for reproducible installs; no "module not found" errors in build logs
- **Severity**: MEDIUM

**2.6**
- **Check**: Redirect and rewrite rules are correctly defined (e.g., trailing slash, www→apex or vice versa)
- **How**: `curl -I https://yourdomain.com` and `curl -I https://yourdomain.com/some-page/`; check Astro `redirects` config and Cloudflare Page Rules / Transform Rules
- **Pass criteria**: Consistent redirect behavior; no redirect loops (check with `curl -L --max-redirs 5`); canonical URL is enforced
- **Severity**: MEDIUM

---

## 3. DNS & NETWORK (Cloudflare)

**3.1**
- **Check**: DNS CNAME record for the tunnel is correctly pointing to `<tunnel-id>.cfargotunnel.com`
- **How**: `dig CNAME yourdomain.com` and `dig CNAME www.yourdomain.com`; compare against tunnel ID from `cloudflared tunnel list`
- **Pass criteria**: CNAME resolves to `<correct-tunnel-id>.cfargotunnel.com`; record is proxied (orange-cloud, not grey-cloud DNS-only)
- **Severity**: CRITICAL

**3.2**
- **Check**: SSL/TLS mode is set to "Full (strict)" in Cloudflare dashboard
- **How**: Cloudflare Dashboard → SSL/TLS → Overview; verify mode shown
- **Pass criteria**: Mode is "Full (strict)"; NOT "Flexible" (which would allow unencrypted origin traffic) and NOT "Off"
- **Severity**: CRITICAL

**3.3**
- **Check**: HTTPS is enforced — HTTP requests redirect to HTTPS
- **How**: `curl -I http://yourdomain.com`; check for `301` or `308` redirect to `https://`
- **Pass criteria**: All HTTP requests return 3xx redirect to HTTPS; Cloudflare "Always Use HTTPS" is enabled
- **Severity**: HIGH

**3.4**
- **Check**: www and apex domain both resolve and redirect consistently
- **How**: `curl -IL https://www.yourdomain.com` and `curl -IL https://yourdomain.com`; verify one canonically redirects to the other
- **Pass criteria**: One canonical form is used; the other redirects with 301; no split-brain content
- **Severity**: MEDIUM

**3.5**
- **Check**: DNS propagation is complete and consistent across resolvers
- **How**: `dig yourdomain.com @8.8.8.8`, `dig yourdomain.com @1.1.1.1`, `dig yourdomain.com @9.9.9.9`; use `https://dnschecker.org` for global view
- **Pass criteria**: All resolvers return the same Cloudflare proxy IP; no stale records pointing to old IPs
- **Severity**: HIGH

---

## 4. PAGES & FUNCTIONALITY

**4.1**
- **Check**: All defined routes return HTTP 200 (no 404 or 500 errors)
- **How**: `wget --spider -r -nd --no-verbose -H -D yourdomain.com https://yourdomain.com 2>&1 | grep -E 'ERROR|broken'`; or use `linkchecker https://yourdomain.com`
- **Pass criteria**: Zero 4xx or 5xx responses on any internal page; all dynamic routes (e.g., `/blog/[slug]`) resolve correctly
- **Severity**: CRITICAL

**4.2**
- **Check**: All internal links and navigation items resolve correctly
- **How**: `linkchecker --check-extern=no https://yourdomain.com`; manually click through primary navigation
- **Pass criteria**: No broken internal links; navigation items lead to correct pages; no orphaned pages
- **Severity**: HIGH

**4.3**
- **Check**: Static assets (images, fonts, JS, CSS) load without 404 errors
- **How**: Open browser DevTools → Network tab; reload each major page; filter by status 4xx/5xx
- **Pass criteria**: Zero asset 404s; all fonts, images, and scripts load successfully
- **Severity**: HIGH

**4.4**
- **Check**: Dynamic blog routes (`/blog/[slug]`) render correctly for all published posts
- **How**: Fetch sitemap (`https://yourdomain.com/sitemap.xml`); extract all blog URLs; `curl -o /dev/null -s -w "%{http_code}" <url>` for each
- **Pass criteria**: All blog post URLs return 200; no slug collision or missing post pages
- **Severity**: HIGH

**4.5**
- **Check**: Page loading behavior is correct (no flash of unstyled content, no layout shift on load)
- **How**: Open each major page in Chrome with DevTools → Performance tab; record page load; observe CLS score
- **Pass criteria**: No visible FOUC; CLS < 0.1 (Core Web Vitals threshold)
- **Severity**: MEDIUM

---

## 5. CHATBOT (FlowiseAI / Agentflow)

**5.1**
- **Check**: FlowiseAI API endpoint is reachable from the origin and returns healthy status
- **How**: `curl -s http://localhost:<FLOWISE_PORT>/api/v1/chatflows` (from origin); verify HTTP 200 and valid JSON response
- **Pass criteria**: 200 response with chatflow list; no connection refused or timeout
- **Severity**: CRITICAL

**5.2**
- **Check**: Chatbot send/receive cycle works end-to-end from the public site
- **How**: Open the site's chatbot UI; send a test message in Persian; verify a response is returned within acceptable latency
- **Pass criteria**: Response received within 30 seconds; response is in Persian; no error toast or empty response displayed
- **Severity**: CRITICAL

**5.3**
- **Check**: Chatbot API key / bearer token is not exposed in client-side JavaScript bundles
- **How**: `curl https://yourdomain.com/path/to/chatbot-page` | grep -i 'api.key\|bearer\|flowise'; inspect browser DevTools → Sources for any bundled secrets
- **Pass criteria**: No API keys or tokens present in any client-side asset; all authenticated calls proxied server-side
- **Severity**: CRITICAL

**5.4**
- **Check**: Chatbot handles LLM backend timeout or failure gracefully
- **How**: Temporarily stop the FlowiseAI service; send a message via the chatbot UI; observe the user-facing error state
- **Pass criteria**: User sees a clear, localized (Persian) error message; no raw stack trace or JSON error exposed; UI remains functional
- **Severity**: HIGH

**5.5**
- **Check**: Context retention works across a multi-turn conversation
- **How**: Send 3–5 sequential messages referencing earlier context (e.g., "درباره همان موضوع بیشتر توضیح بده"); verify the response demonstrates context awareness
- **Pass criteria**: Responses are contextually coherent; session ID or memory mechanism is functioning
- **Severity**: HIGH

**5.6**
- **Check**: Rate limiting is applied to the chatbot API endpoint
- **How**: Send 20+ rapid requests to the chatbot endpoint using `ab -n 20 -c 5 https://yourdomain.com/api/chat`; check Cloudflare WAF logs for rate-limit triggers
- **Pass criteria**: Requests beyond the threshold receive 429 responses; legitimate users are not permanently blocked
- **Severity**: HIGH

**5.7**
- **Check**: Chatbot input is sanitized against prompt injection and XSS
- **How**: Submit inputs containing `<script>alert(1)</script>`, `'; DROP TABLE`, and common prompt injection strings (e.g., "Ignore previous instructions and..."); observe response and rendered output
- **Pass criteria**: No script execution in browser; injected instructions are not followed by the LLM in a way that leaks system prompts or bypasses guardrails; output is HTML-escaped
- **Severity**: HIGH

---

## 6. AI BLOG PIPELINE (Generate → Publish → Telegram)

**6.1**
- **Check**: Blog generation trigger fires reliably on schedule
- **How**: Inspect the scheduler (cron, GitHub Actions schedule, or custom daemon); check last execution time and exit code in logs
- **Pass criteria**: Trigger fires at the configured interval with no missed runs in the last 7 days; exit code 0 on success
- **Severity**: CRITICAL

**6.2**
- **Check**: Generated post is successfully committed and pushed to the GitHub repo, triggering a deploy
- **How**: Check GitHub repo commit history for automated commits; verify the deploy pipeline runs after each auto-commit; `git log --oneline --author="bot" -10`
- **Pass criteria**: Each generation cycle produces exactly one new commit; deploy pipeline completes successfully after each auto-commit
- **Severity**: CRITICAL

**6.3**
- **Check**: Duplicate post prevention mechanism is in place and working
- **How**: Review the generation script for deduplication logic (slug uniqueness check, title hash, or database of published topics); attempt to trigger generation twice for the same topic
- **Pass criteria**: Second generation attempt for an identical topic is rejected or produces a distinct post; no duplicate slugs exist in the repo
- **Severity**: HIGH

**6.4**
- **Check**: Telegram delivery succeeds for each published post
- **How**: Check Telegram bot send logs; use `curl "https://api.telegram.org/bot<TOKEN>/getUpdates"` to verify recent messages; inspect for `"ok": true` in responses
- **Pass criteria**: Every published post has a corresponding Telegram message delivered within 5 minutes of publish; no silent failures
- **Severity**: CRITICAL

**6.5**
- **Check**: Telegram message respects the 4096-character limit and handles truncation or splitting
- **How**: Identify the longest generated post; measure the Telegram-formatted version character count; check if the send function splits long messages
- **Pass criteria**: No `Bad Request: message is too long` errors in Telegram API logs; long posts are either split into multiple messages or truncated with a "read more" link
- **Severity**: HIGH

**6.6**
- **Check**: RTL rendering and Persian text integrity in Telegram messages
- **How**: Open the Telegram channel/chat on both mobile and desktop; verify Persian text renders right-to-left; check that mixed LTR content (URLs, code) does not break RTL flow
- **Pass criteria**: Persian text is visually RTL; no reversed characters or garbled bidi sequences; links are clickable and correct
- **Severity**: HIGH

**6.7**
- **Check**: Telegram retry logic handles transient API failures
- **How**: Review the send function for retry/backoff implementation; simulate a Telegram API timeout by temporarily blocking outbound traffic to `api.telegram.org`; observe behavior
- **Pass criteria**: At least 3 retry attempts with exponential backoff; failure is logged with alert; post is not silently dropped
- **Severity**: HIGH

**6.8**
- **Check**: A content-review or QA gate exists before auto-publish (human or automated)
- **How**: Review the pipeline code/config for any approval step, content validation function, or moderation queue
- **Pass criteria**: Either a human approval step exists, OR an automated quality gate (minimum word count, language detection confirming Persian, hallucination/toxicity filter) is in place; fully autonomous publish with no gate is flagged as a risk
- **Severity**: HIGH

**6.9**
- **Check**: Link previews in Telegram are correct (Open Graph metadata is fetched correctly)
- **How**: Send a blog post URL to a Telegram chat manually; verify the preview shows the correct title, description, and image
- **Pass criteria**: Preview displays correct Persian title and description; image is present and correctly sized; URL is not broken
- **Severity**: MEDIUM

---

## 7. PERSIAN / RTL CORRECTNESS

**7.1**
- **Check**: `<html>` element has `lang="fa"` and `dir="rtl"` attributes
- **How**: `curl https://yourdomain.com | grep -i '<html'`; or browser DevTools → Elements → inspect `<html>` tag
- **Pass criteria**: `<html lang="fa" dir="rtl">` present on every page; not just the homepage
- **Severity**: HIGH

**7.2**
- **Check**: Page layout is correctly mirrored for RTL (navigation, sidebars, text alignment, icons)
- **How**: Open each major page in Chrome; use DevTools → Rendering → Emulate CSS media feature `direction: rtl`; visually inspect alignment
- **Pass criteria**: Text is right-aligned; navigation flows right-to-left; no LTR-only CSS properties (e.g., `float: left`, `text-align: left`) breaking layout; icons that imply direction (arrows, chevrons) are mirrored
- **Severity**: HIGH

**7.3**
- **Check**: Bidirectional (bidi) text handling is correct for mixed Persian/Latin content (URLs, numbers, code)
- **How**: Find a page with mixed content; inspect in browser; use the Unicode Bidi Algorithm checker at `https://www.unicode.org/cldr/utility/bidi.jsp`
- **Pass criteria**: Persian text flows RTL; embedded Latin/URLs flow LTR within the RTL context; no character reversal or garbled sequences; `unicode-bidi: embed` or `isolate` applied where needed
- **Severity**: HIGH

**7.4**
- **Check**: Persian font loads correctly with appropriate fallback stack
- **How**: Browser DevTools → Network → filter by "Font"; verify the Persian font file (e.g., Vazirmatn, IRANSans) loads with HTTP 200; check CSS `font-family` stack
- **Pass criteria**: Persian font loads successfully; fallback stack includes a system Persian font (e.g., `Tahoma`, `Arial`); no font substitution causing incorrect glyph rendering
- **Severity**: HIGH

**7.5**
- **Check**: Persian numerals and punctuation render correctly
- **How**: Inspect blog posts for numbers; verify whether Eastern Arabic numerals (۰۱۲۳) or Western Arabic (0123) are used consistently per editorial decision; check `،` (Persian comma) and `؟` (Persian question mark) rendering
- **Pass criteria**: Numeral style is consistent with editorial policy; Persian punctuation characters render correctly and are not replaced by Latin equivalents
- **Severity**: MEDIUM

---

## 8. CONTENT QUALITY (AI-Generated Persian)

**8.1**
- **Check**: Generated posts are grammatically correct and fluent in Persian (Farsi)
- **How**: Have a native Persian speaker review 5 randomly selected posts; alternatively, run posts through a Persian grammar checker (e.g., VirastyarAPI or manual review)
- **Pass criteria**: No obvious grammatical errors, unnatural phrasing, or code-switching to Arabic/English mid-sentence; text reads as natural Farsi
- **Severity**: HIGH

**8.2**
- **Check**: Factual accuracy — posts do not contain hallucinated facts, statistics, or citations
- **How**: Select 3 posts on verifiable topics; fact-check key claims against authoritative sources; check for fabricated URLs or citations
- **Pass criteria**: All verifiable claims are accurate; no fabricated statistics or non-existent citations; any uncertain claims are appropriately hedged
- **Severity**: HIGH

**8.3**
- **Check**: Posts are complete — no truncated content, mid-sentence endings, or placeholder text
- **How**: Fetch all blog post pages; check for posts ending abruptly; search for placeholder strings: `grep -r 'TODO\|PLACEHOLDER\|Lorem\|\.\.\.$' dist/`
- **Pass criteria**: All posts have a complete introduction, body, and conclusion; no truncation artifacts; minimum word count threshold is met
- **Severity**: HIGH

**8.4**
- **Check**: Formatting is consistent across all posts (headings, paragraphs, lists, code blocks)
- **How**: Compare 10 posts visually; check Markdown/HTML structure in source; verify heading hierarchy (`h1` → `h2` → `h3`)
- **Pass criteria**: Consistent heading structure; no posts with missing headings or wall-of-text paragraphs; code blocks use `<pre>`/`<code>` tags correctly
- **Severity**: MEDIUM

**8.5**
- **Check**: No empty or near-empty posts are published
- **How**: Query all blog post pages; check word count: `curl https://yourdomain.com/blog/<slug> | python3 -c "import sys,re; print(len(re.findall(r'\w+', sys.stdin.read())))"` for each
- **Pass criteria**: All published posts exceed a minimum word count (e.g., 300 words); no posts with only a title and no body
- **Severity**: HIGH

---

## 9. SEO & DISCOVERABILITY

**9.1**
- **Check**: Every page has a unique, descriptive `<title>` and `<meta name="description">` in Persian
- **How**: `curl https://yourdomain.com/blog/<slug> | grep -E '<title>|meta name="description"'`; run all pages through `https://www.seoptimer.com` or Screaming Frog
- **Pass criteria**: No duplicate titles; no missing descriptions; titles are ≤ 60 characters; descriptions are 120–160 characters; all in Persian
- **Severity**: HIGH

**9.2**
- **Check**: Open Graph and Twitter Card meta tags are present and correct
- **How**: `curl https://yourdomain.com/blog/<slug> | grep -E 'og:|twitter:'`; validate with `https://developers.facebook.com/tools/debug/`
- **Pass criteria**: `og:title`, `og:description`, `og:image`, `og:url`, `og:locale` (set to `fa_IR`) present on all pages; image URL is absolute and returns 200
- **Severity**: HIGH

**9.3**
- **Check**: `sitemap.xml` is present, valid, and includes all published blog posts
- **How**: `curl https://yourdomain.com/sitemap.xml`; validate with `https://www.xml-sitemaps.com/validate-xml-sitemap.html`; count URLs vs. actual post count
- **Pass criteria**: Sitemap returns 200; all published post URLs are listed; `<lastmod>` dates are accurate; sitemap is submitted to Google Search Console
- **Severity**: HIGH

**9.4**
- **Check**: `robots.txt` is correctly configured — allows crawling of public content, disallows admin/API paths
- **How**: `curl https://yourdomain.com/robots.txt`; verify `Sitemap:` directive points to sitemap URL
- **Pass criteria**: `User-agent: *` with `Allow: /`; API and admin paths disallowed; `Sitemap:` directive present
- **Severity**: MEDIUM

**9.5**
- **Check**: Canonical tags are present and correct on all pages
- **How**: `curl https://yourdomain.com/blog/<slug> | grep 'rel="canonical"'`
- **Pass criteria**: Every page has exactly one `<link rel="canonical" href="...">` pointing to the preferred URL; no self-referencing canonicals on paginated pages without `rel="next"`/`rel="prev"`
- **Severity**: MEDIUM

**9.6**
- **Check**: Pages are indexable (not accidentally blocked by `noindex` or Cloudflare settings)
- **How**: `curl -I https://yourdomain.com/blog/<slug> | grep -i 'x-robots-tag'`; check `<meta name="robots">` in HTML; verify Cloudflare is not serving a challenge page to Googlebot
- **Pass criteria**: No `noindex` directive on public pages; `X-Robots-Tag` header absent or set to `index, follow`; Google Search Console shows pages as indexed
- **Severity**: HIGH

**9.7**
- **Check**: Structured data (JSON-LD) is present for blog posts (Article schema)
- **How**: `curl https://yourdomain.com/blog/<slug> | python3 -c "import sys,json,re; [print(json.dumps(json.loads(m), indent=2)) for m in re.findall(r'<script type=\"application/ld\+json\">(.*?)</script>', sys.stdin.read(), re.S)]"`; validate with Google's Rich Results Test
- **Pass criteria**: `Article` or `BlogPosting` schema present with `headline`, `datePublished`, `author`, `inLanguage: "fa"` fields; no validation errors
- **Severity**: MEDIUM

---

## 10. PERFORMANCE & SPEED

**10.1**
- **Check**: Core Web Vitals pass Google's thresholds
- **How**: Run `npx lighthouse https://yourdomain.com --output=json --output-path=./lh.json`; check LCP, FID/INP, CLS values
- **Pass criteria**: LCP ≤ 2.5s; INP ≤ 200ms; CLS ≤ 0.1; Performance score ≥ 80
- **Severity**: HIGH

**10.2**
- **Check**: Cloudflare cache hit ratio is healthy for static assets
- **How**: Cloudflare Dashboard → Analytics → Cache; check "Cache Hit Ratio"; alternatively `curl -I https://yourdomain.com/assets/main.css | grep -i 'cf-cache-status'`
- **Pass criteria**: `CF-Cache-Status: HIT` for static assets (CSS, JS, images, fonts); cache hit ratio > 80% for static content; `MISS` only on first request or after cache purge
- **Severity**: HIGH

**10.3**
- **Check**: Images are optimized (compressed, correct format, responsive `srcset`)
- **How**: Browser DevTools → Network → filter Images; check file sizes; run `npx imagemin-cli` or inspect Astro image optimization config; check for WebP/AVIF usage
- **Pass criteria**: No uncompressed images > 200KB; hero images use WebP or AVIF; `<img>` elements have `width` and `height` attributes to prevent CLS; `loading="lazy"` on below-fold images
- **Severity**: HIGH

**10.4**
- **Check**: Astro partial hydration (islands) is used correctly — no unnecessary client-side JS
- **How**: `npx lighthouse https://yourdomain.com`; check "Unused JavaScript" audit; inspect Astro components for `client:*` directives
- **Pass criteria**: Only interactive components (chatbot, dynamic widgets) use `client:load` or `client:visible`; static content has no hydration directive; total JS payload < 150KB gzipped
- **Severity**: MEDIUM

**10.5**
- **Check**: Persian font loading does not block rendering
- **How**: Lighthouse → "Eliminate render-blocking resources" audit; check `<link rel="preload">` for font files; verify `font-display: swap` in CSS
- **Pass criteria**: Font `<link>` uses `rel="preload"`; CSS uses `font-display: swap`; no font-related render blocking in Lighthouse report
- **Severity**: MEDIUM

**10.6**
- **Check**: Total page payload size is within acceptable limits
- **How**: Browser DevTools → Network → disable cache → reload; check "transferred" total; or `curl -s -o /dev/null -w "%{size_download}" https://yourdomain.com`
- **Pass criteria**: Homepage HTML < 50KB; total page weight (all assets) < 1MB for initial load
- **Severity**: MEDIUM

---

## 11. UI / BRANDING

**11.1**
- **Check**: Logo and favicon are present, correctly sized, and load without errors
- **How**: `curl -I https://yourdomain.com/favicon.ico`; `curl -I https://yourdomain.com/favicon.svg`; visually inspect logo on all pages
- **Pass criteria**: Favicon returns 200; logo renders at correct size on all breakpoints; no broken image icons
- **Severity**: MEDIUM

**11.2**
- **Check**: Responsive design works correctly across breakpoints (mobile 375px, tablet 768px, desktop 1280px)
- **How**: Chrome DevTools → Device Toolbar; test at 375px, 768px, 1280px; check for horizontal scroll, overlapping elements, or broken layouts
- **Pass criteria**: No horizontal overflow at any breakpoint; navigation collapses correctly on mobile; chatbot widget is usable on mobile; Persian text wraps correctly
- **Severity**: HIGH

**11.3**
- **Check**: Color contrast meets WCAG AA accessibility standards
- **How**: Run `npx axe-cli https://yourdomain.com`; or use Chrome DevTools → Accessibility → Color Contrast
- **Pass criteria**: All text/background combinations have contrast ratio ≥ 4.5:1 (normal text) or ≥ 3:1 (large text); no axe-core critical violations
- **Severity**: MEDIUM

**11.4**
- **Check**: All images have descriptive `alt` text in Persian
- **How**: `curl https://yourdomain.com | grep -o '<img[^>]*>'`; check for missing or empty `alt` attributes; run `npx axe-cli https://yourdomain.com --tags wcag2a`
- **Pass criteria**: Every `<img>` has a non-empty `alt` attribute in Persian (or `alt=""` for decorative images); no axe "image-alt" violations
- **Severity**: MEDIUM

**11.5**
- **Check**: Keyboard navigation and focus indicators are functional
- **How**: Tab through each page using only the keyboard; verify focus ring is visible on all interactive elements
- **Pass criteria**: All interactive elements (links, buttons, chatbot input) are reachable via Tab; focus ring is visible and not hidden by `outline: none` without replacement
- **Severity**: LOW

---

## 12. SECURITY

**12.1**
- **Check**: Security response headers are set (CSP, HSTS, X-Frame-Options, X-Content-Type-Options)
- **How**: `curl -I https://yourdomain.com | grep -iE 'strict-transport|content-security|x-frame|x-content-type|referrer-policy'`
- **Pass criteria**: `Strict-Transport-Security: max-age=31536000; includeSubDomains` present; `X-Content-Type-Options: nosniff`; `X-Frame-Options: SAMEORIGIN` or CSP `frame-ancestors`; CSP policy defined (even if report-only initially)
- **Severity**: HIGH

**12.2**
- **Check**: Cloudflare WAF is enabled with appropriate rulesets
- **How**: Cloudflare Dashboard → Security → WAF; verify Cloudflare Managed Ruleset is enabled; check for custom rules blocking common attack patterns
- **Pass criteria**: Managed Ruleset active; Bot Fight Mode or Super Bot Fight Mode enabled; rate limiting rules configured for chatbot and API endpoints
- **Severity**: HIGH

**12.3**
- **Check**: No API keys, tokens, or secrets are present in the public GitHub repository
- **How**: `git log --all --full-history -- '*.env' '*.json' '*.yml'`; use `truffleHog` or `gitleaks`: `gitleaks detect --source . --verbose`
- **Pass criteria**: Zero secrets found in any commit in history; `.env` files are in `.gitignore`; only example/template env files (`.env.example`) are committed
- **Severity**: CRITICAL

**12.4**
- **Check**: `noTLSVerify` is NOT set to `true` in `config.yml` for production ingress rules
- **How**: `grep -i 'noTLSVerify' /etc/cloudflared/config.yml`
- **Pass criteria**: `noTLSVerify` is absent or explicitly `false`; origin TLS certificate is valid and verified
- **Severity**: HIGH [14](#0-13) 

**12.5**
- **Check**: Cloudflare Access JWT validation is configured for any admin or internal endpoints
- **How**: Check `config.yml` for `access.required: true` on sensitive ingress rules; verify `access.teamName` and `access.audTag` are set
- **Pass criteria**: Admin/internal paths have `access.required: true` with correct `teamName` and `audTag`; requests without valid JWT are rejected
- **Severity**: HIGH [15](#0-14) 

**12.6**
- **Check**: Telegram bot token is stored securely and not exposed in logs or client-side code
- **How**: Search codebase: `grep -r 'bot[0-9]\{9\}:' .`; check application logs for token leakage; verify token is loaded from environment variable only
- **Pass criteria**: Token only appears in environment variable configuration; not in source code, logs, or client-side bundles
- **Severity**: CRITICAL

---

## 13. BACKUP & RECOVERY

**13.1**
- **Check**: GitHub repository is the authoritative source and is backed up or mirrored
- **How**: Verify GitHub repository has at least one additional remote or is mirrored (GitHub → Settings → Danger Zone → Transfer/Mirror); check if organization has GitHub backup enabled
- **Pass criteria**: Repository has off-site backup or mirror; last backup is < 24 hours old
- **Severity**: HIGH

**13.2**
- **Check**: FlowiseAI flows, agents, and knowledge bases are backed up
- **How**: Use Flowise export: Settings → Export; verify exported JSON is stored off-origin (e.g., S3, GitHub, external storage); check backup frequency
- **Pass criteria**: Flowise flows exported and stored off-origin; backup is < 7 days old; restore procedure is documented
- **Severity**: HIGH

**13.3**
- **Check**: `cloudflared` tunnel configuration and credentials are backed up
- **How**: Verify `/etc/cloudflared/config.yml` and `<tunnel-id>.json` are backed up off-origin; check if they are stored in a secrets manager or encrypted backup
- **Pass criteria**: Config and credentials backed up securely off-origin; backup is < 7 days old; recovery procedure documented (re-run `cloudflared service install` with backed-up token)
- **Severity**: HIGH

**13.4**
- **Check**: A restore test has been performed within the last 90 days
- **How**: Review runbook or incident log for last restore test date and outcome
- **Pass criteria**: Restore test completed successfully within 90 days; test covered: repo clone + build, Flowise flow import, tunnel re-registration
- **Severity**: MEDIUM

---

## 14. OPTIMIZATION & TECHNICAL DEBT

**14.1**
- **Check**: Astro and all integration packages are up-to-date; no deprecated packages
- **How**: `npm outdated`; `npx npm-check-updates`; check Astro changelog for breaking changes between current and latest version
- **Pass criteria**: No packages more than 2 major versions behind; no packages with known CVEs (`npm audit`); Astro adapter is compatible with current Astro version
- **Severity**: MEDIUM

**14.2**
- **Check**: `cloudflared` auto-update is not disabled without a documented reason
- **How**: `systemctl status cloudflared-update.timer`; check `config.yml` for `no-autoupdate: true`; check service install flags
- **Pass criteria**: Auto-update timer is active OR a documented manual update process with defined frequency exists; `--no-autoupdate` flag is not set without justification
- **Severity**: MEDIUM [16](#0-15) 

**14.3**
- **Check**: Cloudflare Analytics or Web Analytics is enabled for traffic visibility
- **How**: Cloudflare Dashboard → Analytics & Logs → Web Analytics; verify beacon script is present on site pages
- **Pass criteria**: Analytics enabled; data is being collected; no reliance solely on server-side logs which are bypassed by Cloudflare caching
- **Severity**: LOW

**14.4**
- **Check**: Dead or unused configuration in `config.yml` is removed
- **How**: Review `/etc/cloudflared/config.yml` for commented-out rules, unused ingress entries, or legacy flags (e.g., `h2mux` protocol references, deprecated `warp-routing.enabled` boolean)
- **Pass criteria**: No deprecated flags present; no unused ingress rules; config is minimal and documented
- **Severity**: LOW [17](#0-16) 

**14.5**
- **Check**: Cloudflare Cache Rules are configured to maximize cache hit ratio for Astro static output
- **How**: Cloudflare Dashboard → Caching → Cache Rules; verify rules for `/assets/*`, `/_astro/*`, and blog post pages
- **Pass criteria**: Static asset paths cached at edge with appropriate TTL; HTML pages cached with `Cache-Control` headers set by Astro; API/chatbot paths explicitly bypassed from cache
- **Severity**: MEDIUM

**14.6**
- **Check**: Prometheus metrics from `cloudflared` are being scraped and retained
- **How**: Verify a Prometheus scrape job targets `localhost:20241/metrics` (or the bound port); check Grafana or equivalent for `cloudflared_tunnel_*` metrics
- **Pass criteria**: Metrics are scraped at ≤ 60s interval; at least 7 days of retention; dashboards show tunnel connection count, request rate, and error rate
- **Severity**: MEDIUM [18](#0-17) 

---

## 15. MONITORING & ERROR HANDLING

**15.1**
- **Check**: External uptime monitoring is configured for the public site
- **How**: Verify an uptime monitor (e.g., UptimeRobot, Betterstack, Cloudflare Health Checks) is configured for `https://yourdomain.com`; check alert notification channel
- **Pass criteria**: Monitor checks at ≤ 5-minute intervals; alerts sent to email/Slack/Telegram on downtime; last alert test < 30 days ago
- **Severity**: CRITICAL

**15.2**
- **Check**: Tunnel failure triggers an alert
- **How**: Check if `cloudflared_tunnel_register_fail` or `cloudflared_tunnel_rpc_fail` Prometheus metrics have alerting rules; or verify systemd failure notification is configured
- **Pass criteria**: Alert fires within 5 minutes of tunnel going down; alert reaches an on-call channel; `systemd` `OnFailure=` unit or Prometheus alerting rule exists
- **Severity**: CRITICAL [19](#0-18) 

**15.3**
- **Check**: Blog pipeline failures (generation, publish, Telegram send) trigger alerts
- **How**: Review pipeline error handling code; check if failures send a notification (email, Telegram message to admin, or PagerDuty)
- **Pass criteria**: Any pipeline step failure sends an alert within 10 minutes; alert includes which step failed and the error message; no silent failures
- **Severity**: HIGH

**15.4**
- **Check**: Centralized logging is configured and logs are retained
- **How**: `journalctl -u cloudflared --since "1 hour ago"`; verify application logs (Astro, Flowise, pipeline) are written to a persistent location; check log rotation config
- **Pass criteria**: `cloudflared` logs available via journald; application logs retained for ≥ 30 days; log rotation prevents disk exhaustion; `cloudflared tail <TUNNEL-ID>` works for remote log streaming
- **Severity**: HIGH [20](#0-19) 

**15.5**
- **Check**: User-facing error pages are localized in Persian and informative
- **How**: Trigger a 404 (visit a non-existent URL) and a 500 (temporarily break the origin); observe the error page
- **Pass criteria**: Custom 404 and 500 pages exist in Persian; pages include navigation back to homepage; no raw stack traces or English-only Cloudflare default error pages shown to users
- **Severity**: MEDIUM

**15.6**
- **Check**: Chatbot and pipeline error states are logged with sufficient context for debugging
- **How**: Review application logs after a simulated chatbot failure; check that logs include: timestamp, request ID, error type, and stack trace
- **Pass criteria**: Errors are logged at `ERROR` level with full context; logs are queryable; no sensitive data (API keys, user PII) in log output
- **Severity**: MEDIUM

---

## PRIORITIZED SUMMARY — CRITICAL & HIGH ITEMS (First-Pass Order)

**CRITICAL**

| # | Item | Category |
|---|------|----------|
| C1 | `cloudflared` daemon running as systemd service with `Restart=on-failure` | 1.1 |
| C2 | Tunnel connected with 4 HA connections; `/ready` endpoint returns 200 | 1.2, 1.7 |
| C3 | Ingress rules validated (`cloudflared tunnel ingress validate`) | 1.3 |
| C4 | Origin has NO public-facing ports (verified from external host) | 1.4 |
| C5 | GitHub → deploy pipeline succeeds reliably | 2.2 |
| C6 | All required secrets/env vars present; none hardcoded in repo | 2.3 |
| C7 | DNS CNAME points to correct tunnel ID; orange-cloud proxied | 3.1 |
| C8 | SSL/TLS mode is "Full (strict)" | 3.2 |
| C9 | All pages return 200; no 404/500 on any route | 4.1 |
| C10 | FlowiseAI API reachable; chatbot end-to-end works | 5.1, 5.2 |
| C11 | Chatbot API key not exposed in client-side JS | 5.3 |
| C12 | Blog generation trigger fires reliably; auto-commit + deploy succeeds | 6.1, 6.2 |
| C13 | Telegram delivery succeeds for every published post | 6.4 |
| C14 | No secrets in GitHub repo history (`gitleaks detect`) | 12.3 |
| C15 | Telegram bot token not in source code or logs | 12.6 |
| C16 | External uptime monitor configured with alerting | 15.1 |
| C17 | Tunnel failure triggers alert within 5 minutes | 15.2 |

**HIGH**

| # | Item | Category |
|---|------|----------|
| H1 | `cloudflared` version within 1-year support window | 1.5 |
| H2 | Tunnel credentials files have restrictive permissions (600) | 1.10 |
| H3 | Astro build completes without errors | 2.1 |
| H4 | HTTPS enforced; HTTP redirects to HTTPS | 3.3 |
| H5 | DNS propagation consistent across resolvers | 3.5 |
| H6 | All internal links and static assets load without 404 | 4.2, 4.3 |
| H7 | All blog post dynamic routes return 200 | 4.4 |
| H8 | Chatbot handles backend failure gracefully with Persian error message | 5.4 |
| H9 | Chatbot context retention works across multi-turn conversations | 5.5 |
| H10 | Rate limiting applied to chatbot API endpoint | 5.6 |
| H11 | Chatbot input sanitized against XSS and prompt injection | 5.7 |
| H12 | Duplicate post prevention mechanism verified | 6.3 |
| H13 | Telegram message length handled (4096-char limit) | 6.5 |
| H14 | RTL rendering correct in Telegram messages | 6.6 |
| H15 | Telegram retry logic handles transient API failures | 6.7 |
| H16 | Content QA gate exists before auto-publish | 6.8 |
| H17 | `lang="fa" dir="rtl"` on all pages | 7.1 |
| H18 | RTL layout correct on all pages | 7.2 |
| H19 | Bidi text handling correct for mixed content | 7.3 |
| H20 | Persian font loads with correct fallback stack | 7.4 |
| H21 | Generated posts are grammatically correct Persian | 8.1 |
| H22 | Posts contain no hallucinated facts or fabricated citations | 8.2 |
| H23 | No truncated or incomplete posts published | 8.3 |
| H24 | No empty posts published (minimum word count enforced) | 8.5 |
| H25 | Unique Persian meta titles and descriptions on all pages | 9.1 |
| H26 | Open Graph tags present and correct (`og:locale: fa_IR`) | 9.2 |
| H27 | `sitemap.xml` valid and includes all posts | 9.3 |
| H28 | Pages are indexable; no accidental `noindex` | 9.6 |
| H29 | Core Web Vitals pass (LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1) | 10.1 |
| H30 | Cloudflare cache hit ratio > 80% for static assets | 10.2 |
| H31 | Images optimized (WebP/AVIF, compressed, lazy-loaded) | 10.3 |
| H32 | Responsive design verified at 375px, 768px, 1280px | 11.2 |
| H33 | Security headers set (HSTS, CSP, X-Content-Type-Options) | 12.1 |
| H34 | Cloudflare WAF enabled with managed ruleset | 12.2 |
| H35 | `noTLSVerify: false` in production config | 12.4 |
| H36 | Cloudflare Access JWT validation on admin/internal paths | 12.5 |
| H37 | GitHub repo and Flowise flows backed up off-origin | 13.1, 13.2 |
| H38 | Tunnel config and credentials backed up securely | 13.3 |
| H39 | Blog pipeline failures trigger alerts | 15.3 |
| H40 | Centralized logging with ≥ 30-day retention | 15.4 |

### Citations

**File:** cmd/cloudflared/linux_service.go (L44-52)
```go
const (
	serviceConfigDir         = "/etc/cloudflared"
	serviceConfigFile        = "config.yml"
	serviceCredentialFile    = "cert.pem"
	serviceConfigPath        = serviceConfigDir + "/" + serviceConfigFile
	cloudflaredService       = "cloudflared.service"
	cloudflaredUpdateService = "cloudflared-update.service"
	cloudflaredUpdateTimer   = "cloudflared-update.timer"
)
```

**File:** cmd/cloudflared/linux_service.go (L54-71)
```go
var systemdAllTemplates = map[string]ServiceTemplate{
	cloudflaredService: {
		Path: fmt.Sprintf("/etc/systemd/system/%s", cloudflaredService),
		Content: `[Unit]
Description=cloudflared
After=network-online.target
Wants=network-online.target

[Service]
TimeoutStartSec=0
Type=notify
ExecStart={{ .Path }} --no-autoupdate{{ range .ExtraArgs }} {{ . }}{{ end }}
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
`,
```

**File:** cmd/cloudflared/linux_service.go (L73-95)
```go
	cloudflaredUpdateService: {
		Path: fmt.Sprintf("/etc/systemd/system/%s", cloudflaredUpdateService),
		Content: `[Unit]
Description=Update cloudflared
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/bin/bash -c '{{ .Path }} update; code=$?; if [ $code -eq 11 ]; then systemctl restart cloudflared; exit 0; fi; exit $code'
`,
	},
	cloudflaredUpdateTimer: {
		Path: fmt.Sprintf("/etc/systemd/system/%s", cloudflaredUpdateTimer),
		Content: `[Unit]
Description=Update cloudflared

[Timer]
OnCalendar=daily

[Install]
WantedBy=timers.target
`,
	},
```

**File:** cmd/cloudflared/linux_service.go (L189-193)
```go
var noUpdateServiceFlag = &cli.BoolFlag{
	Name:  "no-update-service",
	Usage: "Disable auto-update of the cloudflared linux service, which restarts the server to upgrade for new versions.",
	Value: false,
}
```

**File:** connection/metrics.go (L82-91)
```go
	serverLocations := prometheus.NewGaugeVec(
		prometheus.GaugeOpts{
			Namespace: MetricsNamespace,
			Subsystem: TunnelSubsystem,
			Name:      "server_locations",
			Help:      "Where each tunnel is connected to. 1 means current location, 0 means previous locations.",
		},
		[]string{"connection_id", "edge_location"},
	)
	prometheus.MustRegister(serverLocations)
```

**File:** connection/metrics.go (L93-113)
```go
	rpcFail := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Namespace: MetricsNamespace,
			Subsystem: TunnelSubsystem,
			Name:      "tunnel_rpc_fail",
			Help:      "Count of RPC connection errors by type",
		},
		[]string{"error", "rpcName"},
	)
	prometheus.MustRegister(rpcFail)

	registerFail := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Namespace: MetricsNamespace,
			Subsystem: TunnelSubsystem,
			Name:      "tunnel_register_fail",
			Help:      "Count of tunnel registration errors by type",
		},
		[]string{"error", "rpcName"},
	)
	prometheus.MustRegister(registerFail)
```

**File:** cmd/cloudflared/tunnel/ingress_subcommands.go (L55-63)
```go
func buildValidateIngressCommand() *cli.Command {
	return &cli.Command{
		Name:        "validate",
		Action:      cliutil.ConfiguredActionWithWarnings(validateIngressCommand),
		Usage:       "Validate the ingress configuration ",
		UsageText:   "cloudflared tunnel [--config FILEPATH] ingress validate",
		Description: "Validates the configuration file, ensuring your ingress rules are OK.",
		Flags:       []cli.Flag{ingressDataJSON},
	}
```

**File:** ingress/ingress.go (L373-383)
```go
	// The last rule should catch all hostnames.
	isCatchAllRule := (r.Hostname == "" || r.Hostname == "*") && r.Path == ""
	isLastRule := ruleIndex == totalRules-1
	if isLastRule && !isCatchAllRule {
		return errLastRuleNotCatchAll
	}
	// ONLY the last rule should catch all hostnames.
	if !isLastRule && isCatchAllRule {
		return ruleShouldNotBeCatchAllError{index: ruleIndex, hostname: r.Hostname}
	}
	return nil
```

**File:** README.md (L1-6)
```markdown
# Cloudflare Tunnel client

Contains the command-line client for Cloudflare Tunnel, a tunneling daemon that proxies traffic from the Cloudflare network to your origins.
This daemon sits between Cloudflare network and your origin (e.g. a webserver). Cloudflare attracts client requests and sends them to you
via this daemon, without requiring you to poke holes on your firewall --- your origin can remain as closed as possible.
Extensive documentation can be found in the [Cloudflare Tunnel section](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps) of the Cloudflare Docs.
```

**File:** README.md (L54-58)
```markdown
## Deprecated versions

Cloudflare currently supports versions of cloudflared that are **within one year** of the most recent release. Breaking changes unrelated to feature availability may be introduced that will impact versions released more than one year ago. You can read more about upgrading cloudflared in our [developer documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/#updating-cloudflared).

For example, as of January 2023 Cloudflare will support cloudflared version 2023.1.1 to cloudflared 2022.1.1.
```

**File:** cmd/cloudflared/updater/update.go (L40-53)
```go
// BinaryUpdated implements ExitCoder interface, the app will exit with status code 11
// https://pkg.go.dev/github.com/urfave/cli/v2?tab=doc#ExitCoder
// nolint: errname
type statusSuccess struct {
	newVersion string
}

func (u *statusSuccess) Error() string {
	return fmt.Sprintf("cloudflared has been updated to version %s", u.newVersion)
}

func (u *statusSuccess) ExitCode() int {
	return 11
}
```

**File:** CHANGES.md (L14-15)
```markdown
### Notices
- The use of the `--metrics` is still honoured meaning that if this flag is set the metrics server will try to bind it, however, this version includes a change that makes the metrics server bind to a port with a semi-deterministic approach. If the metrics flag is not present the server will bind to the first available port of the range 20241 to 20245. In case of all ports being unavailable then the fallback is to bind to a random port.
```

**File:** CHANGES.md (L25-27)
```markdown
## 2023.9.0
### Notices
- The `warp-routing` `enabled: boolean` flag is no longer supported in the configuration file. Warp Routing traffic (eg TCP, UDP, ICMP) traffic is proxied to cloudflared if routes to the target tunnel are configured. This change does not affect remotely managed tunnels, but for locally managed tunnels, users that might be relying on this feature flag to block traffic should instead guarantee that tunnel has no Private Routes configured for the tunnel.
```

**File:** CHANGES.md (L33-34)
```markdown
### New Features
- You can now stream your logs from your remote cloudflared to your local terminal with `cloudflared tail <TUNNEL-ID>`. This new feature requires the remote cloudflared to be version 2023.4.1 or higher.
```

**File:** cmd/cloudflared/tunnel/configuration.go (L129-158)
```go
	transportProtocol := c.String(flags.Protocol)
	isPostQuantumEnforced := c.Bool(flags.PostQuantum)
	featureSelector, err := features.NewFeatureSelector(ctx, namedTunnel.Credentials.AccountTag, c.StringSlice(flags.Features), isPostQuantumEnforced, log)
	if err != nil {
		return nil, nil, errors.Wrap(err, "Failed to create feature selector")
	}

	clientConfig, err := client.NewConfig(info.Version(), info.OSArch(), featureSelector)
	if err != nil {
		return nil, nil, err
	}

	log.Info().Msgf("Generated Connector ID: %s", clientConfig.ConnectorID)

	tags, err := NewTagSliceFromCLI(c.StringSlice(flags.Tag))
	if err != nil {
		log.Err(err).Msg("Tag parse failure")
		return nil, nil, errors.Wrap(err, "Tag parse failure")
	}
	tags = append(tags, pogs.Tag{Name: "ID", Value: clientConfig.ConnectorID.String()})

	clientFeatures := featureSelector.Snapshot()
	pqMode := clientFeatures.PostQuantum
	if pqMode == features.PostQuantumStrict {
		// Error if the user tries to force a non-quic transport protocol
		if transportProtocol != connection.AutoSelectFlag && transportProtocol != connection.QUIC.String() {
			return nil, nil, fmt.Errorf("post-quantum is only supported with the quic transport")
		}
		transportProtocol = connection.QUIC.String()
	}
```

**File:** cmd/cloudflared/tunnel/subcommands.go (L1076-1095)
```go
func buildDiagCommand() *cli.Command {
	return &cli.Command{
		Name:        "diag",
		Action:      cliutil.ConfiguredAction(diagCommand),
		Usage:       "Creates a diagnostic report from a local cloudflared instance",
		UsageText:   "cloudflared tunnel [tunnel command options] diag [subcommand options]",
		Description: "cloudflared tunnel diag will create a diagnostic report of a local cloudflared instance. The diagnostic procedure collects: logs, metrics, system information, traceroute to Cloudflare Edge, and runtime information. Since there may be multiple instances of cloudflared running the --metrics option may be provided to target a specific instance.",
		Flags: []cli.Flag{
			metricsFlag,
			diagContainerFlag,
			diagPodFlag,
			noDiagLogsFlag,
			noDiagMetricsFlag,
			noDiagSystemFlag,
			noDiagRuntimeFlag,
			noDiagNetworkFlag,
		},
		CustomHelpTemplate: commandHelpTemplate(),
	}
}
```

**File:** diagnostic/consts.go (L17-36)
```go
	// Endpoints used by the diagnostic HTTP Client.
	cliConfigurationEndpoint    = "/diag/configuration"
	tunnelStateEndpoint         = "/diag/tunnel"
	systemInformationEndpoint   = "/diag/system"
	memoryDumpEndpoint          = "debug/pprof/heap"
	goroutineDumpEndpoint       = "debug/pprof/goroutine"
	metricsEndpoint             = "metrics"
	tunnelConfigurationEndpoint = "/config"
	// Base for filenames of the diagnostic procedure
	systemInformationBaseName = "systeminformation.json"
	metricsBaseName           = "metrics.txt"
	zipName                   = "cloudflared-diag"
	heapPprofBaseName         = "heap.pprof"
	goroutinePprofBaseName    = "goroutine.pprof"
	networkBaseName           = "network.json"
	rawNetworkBaseName        = "raw-network.txt"
	tunnelStateBaseName       = "tunnelstate.json"
	cliConfigurationBaseName  = "cli-configuration.json"
	configurationBaseName     = "configuration.json"
	taskResultBaseName        = "task-result.json"
```

**File:** config/configuration.go (L213-216)
```go
	// Disables TLS verification of the certificate presented by your origin.
	// Will allow any certificate from the origin to be accepted.
	// Note: The connection from your machine to Cloudflare's Edge is still encrypted.
	NoTLSVerify *bool `yaml:"noTLSVerify" json:"noTLSVerify,omitempty"`
```

**File:** config/configuration.go (L236-247)
```go
type AccessConfig struct {
	// Required when set to true will fail every request that does not arrive through an access authenticated endpoint.
	Required bool `yaml:"required" json:"required,omitempty"`

	// TeamName is the organization team name to get the public key certificates for.
	TeamName string `yaml:"teamName" json:"teamName"`

	// AudTag is the AudTag to verify access JWT against.
	AudTag []string `yaml:"audTag" json:"audTag"`

	Environment string `yaml:"environment" json:"environment,omitempty"`
}
```

**File:** proxy/metrics.go (L12-45)
```go
var (
	totalRequests = prometheus.NewCounter(
		prometheus.CounterOpts{
			Namespace: connection.MetricsNamespace,
			Subsystem: connection.TunnelSubsystem,
			Name:      "total_requests",
			Help:      "Amount of requests proxied through all the tunnels",
		},
	)
	concurrentRequests = prometheus.NewGauge(
		prometheus.GaugeOpts{
			Namespace: connection.MetricsNamespace,
			Subsystem: connection.TunnelSubsystem,
			Name:      "concurrent_requests_per_tunnel",
			Help:      "Concurrent requests proxied through each tunnel",
		},
	)
	responseByCode = prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Namespace: connection.MetricsNamespace,
			Subsystem: connection.TunnelSubsystem,
			Name:      "response_by_code",
			Help:      "Count of responses by HTTP status code",
		},
		[]string{"status_code"},
	)
	requestErrors = prometheus.NewCounter(
		prometheus.CounterOpts{
			Namespace: connection.MetricsNamespace,
			Subsystem: connection.TunnelSubsystem,
			Name:      "request_errors",
			Help:      "Count of error proxying to origin",
		},
	)
```
