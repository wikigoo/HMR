Here is a comprehensive, technically grounded checklist based on the actual Astro and `@astrojs/cloudflare` codebase.

---

# Website Health & Optimization Checklist
## Astro + Cloudflare Workers + FlowiseAI + Persian Blog

---

## 1. Astro Installation & Configuration

### 1.1 Core Configuration (`astro.config.mjs`)
- [ ] Confirm `@astrojs/cloudflare` adapter is installed and configured as the adapter
- [ ] Verify `output` is set to `'server'` (for SSR with chatbot) or `'static'`/hybrid with selective `export const prerender = true` per page
- [ ] Confirm `site` is set to the production URL (required by `@astrojs/sitemap` and canonical URL generation)
- [ ] Verify `compressHTML` is set appropriately — default in Astro v7 is `'jsx'`; set to `true` for HTML-aware compression or `false` to preserve all whitespace
- [ ] Confirm `build.inlineStylesheets` is set to `'auto'` (default) or `'always'` for optimal performance on Cloudflare's edge
- [ ] Verify `prefetch: true` is enabled for faster page transitions (automatically enabled when using `<ClientRouter />`)
- [ ] Confirm `security.checkOrigin: true` is set (default) to prevent CSRF attacks
- [ ] Verify `trailingSlash` is consistently set (`'always'`, `'never'`, or `'ignore'`) and matches Cloudflare `_redirects` rules [1](#0-0) [2](#0-1) 

### 1.2 Cloudflare Adapter Options
- [ ] Confirm `imageService` is set to `'cloudflare-binding'` (default since adapter v13) or `{ build: 'compile', runtime: 'cloudflare-binding' }` for build-time optimization
- [ ] Verify `sessionKVBindingName` matches the KV binding name in `wrangler.jsonc` (default: `'SESSION'`)
- [ ] Confirm `prerenderEnvironment` is `'workerd'` (default) to ensure prerendered pages use the same runtime as production
- [ ] Check that deprecated options (`mode`, `functionPerRoute`, `routes.strategy`, `runtime`) have been removed — these were removed in adapter v10+ [3](#0-2) [4](#0-3) 

### 1.3 Wrangler Configuration (`wrangler.jsonc`)
- [ ] Verify `main` is set to `"@astrojs/cloudflare/entrypoints/server"` (the unified dev+production entrypoint)
- [ ] Confirm `compatibility_date` is set and not older than the bundled `workerd` binary supports (adapter default: `'2026-04-15'`)
- [ ] Verify all required bindings are declared: KV namespaces, D1 databases, R2 buckets, Durable Objects, environment variables
- [ ] Confirm `assets.binding` is set to `"ASSETS"` for static asset serving
- [ ] Run `wrangler types` to regenerate `src/env.d.ts` after any binding changes [5](#0-4) [6](#0-5) 

### 1.4 TypeScript & Type Safety
- [ ] Run `astro check` (requires `@astrojs/check` and `typescript`) to catch type errors across `.astro` files
- [ ] Verify `src/env.d.ts` has correct `Runtime` type from `@astrojs/cloudflare` for typed `Astro.locals`
- [ ] Confirm `tsconfig.json` uses `moduleResolution: 'bundler'` (Astro default for TypeScript 5.0+)

---

## 2. Cloudflare Deployment & Infrastructure

### 2.1 Deployment Pipeline
- [ ] Confirm GitHub repository is connected to Cloudflare Workers deployment (via Cloudflare dashboard or `wrangler deploy`)
- [ ] Verify the build command is `astro build` and the output directory is `dist`
- [ ] Test `astro preview` locally — it now runs using Cloudflare's `workerd` runtime for production parity
- [ ] Confirm environment variables and secrets are set in Cloudflare dashboard (not committed to the repo)
- [ ] Verify `.dev.vars` is in `.gitignore` for local secret management [7](#0-6) 

### 2.2 DNS & Domain
- [ ] Verify DNS A/AAAA records or CNAME point correctly to Cloudflare Workers
- [ ] Confirm SSL/TLS mode is set to "Full (strict)" in Cloudflare dashboard
- [ ] Check that `www` subdomain redirects to apex (or vice versa) consistently
- [ ] Verify no conflicting DNS records (e.g., duplicate A records)
- [ ] Confirm Cloudflare proxy (orange cloud) is enabled for all relevant records
- [ ] Check DNSSEC is enabled in Cloudflare dashboard

### 2.3 Cloudflare Workers Settings
- [ ] Verify the Worker is deployed to the correct Cloudflare account and zone
- [ ] Confirm the Worker's `compatibility_date` in `wrangler.jsonc` matches what is deployed
- [ ] Check Cloudflare Workers CPU and memory limits are not being exceeded (review analytics)
- [ ] Verify static assets are served via the `ASSETS` binding, not through the Worker itself
- [ ] Confirm Cloudflare Pages migration is complete if previously on Pages (adapter v13+ drops Pages support) [8](#0-7) 

### 2.4 Cloudflare Bindings Health
- [ ] Test KV namespace reads/writes (used for sessions via `SESSION` binding)
- [ ] Verify Cloudflare Images binding (`IMAGES`) is provisioned and accessible
- [ ] If using D1, R2, or Durable Objects: confirm bindings are declared in `wrangler.jsonc` and accessible via `env` from `cloudflare:workers`
- [ ] Confirm `Astro.locals.cfContext` (formerly `Astro.locals.runtime`) is accessible for `ExecutionContext`

---

## 3. FlowiseAI Chatbot & API

### 3.1 FlowiseAI Integration Health
- [ ] Verify the FlowiseAI API endpoint URL is correctly configured (environment variable, not hardcoded)
- [ ] Test the chatbot API endpoint returns a valid response (HTTP 200) with a sample query
- [ ] Confirm the FlowiseAI Agentflow is active and not in a stopped/error state
- [ ] Verify API authentication tokens/keys are stored as Cloudflare Worker secrets (not in source code)
- [ ] Test chatbot response latency — Cloudflare Workers have a 30-second CPU time limit per request; ensure streaming is used for long responses

### 3.2 Chatbot UI Integration
- [ ] Confirm the chatbot widget/component loads correctly on all target pages
- [ ] Verify the chatbot iframe or embedded component does not block page rendering (use `client:idle` or `client:visible` directive)
- [ ] Test chatbot on mobile viewports for usability
- [ ] Confirm RTL layout does not break the chatbot UI (check `dir="rtl"` propagation)
- [ ] Verify CORS headers are correctly set on the FlowiseAI API to allow requests from the site's domain
- [ ] Test chatbot error states (API down, timeout) — confirm graceful fallback UI is shown

### 3.3 Security for API
- [ ] Confirm the FlowiseAI API is not publicly accessible without authentication
- [ ] Verify rate limiting is applied to the chatbot API endpoint (Cloudflare Rate Limiting rules or middleware)
- [ ] Check that `security.checkOrigin: true` is set in `astro.config.mjs` to prevent CSRF on form-based chatbot submissions

---

## 4. Blog Content & AI Publishing Pipeline

### 4.1 Content Collections Configuration
- [ ] Verify `src/content.config.ts` defines the blog collection using `defineCollection` with a `glob` loader
- [ ] Confirm the collection schema (Zod) validates required frontmatter fields: `title`, `description`, `pubDate`, `author`, `lang`
- [ ] Verify Persian content files use UTF-8 encoding and are stored in the correct collection directory
- [ ] Run `astro sync` to regenerate content collection types after schema changes
- [ ] Confirm `getCollection('blog')` returns all expected entries in production [9](#0-8) 

### 4.2 AI Content Generation Pipeline
- [ ] Verify the AI content generation script/workflow runs on schedule and produces valid Markdown/MDX
- [ ] Confirm generated frontmatter includes all required fields defined in the collection schema
- [ ] Test that generated content passes `astro check` without type errors
- [ ] Verify the pipeline commits and pushes to the GitHub repository, triggering a Cloudflare deployment
- [ ] Confirm there are no duplicate slugs in generated content (Astro's `glob` loader warns on duplicates)
- [ ] Verify Persian text is correctly encoded and renders without character corruption

### 4.3 Blog Post Quality
- [ ] Confirm each post has a unique, descriptive `title` and `description` for SEO
- [ ] Verify `pubDate` is a valid ISO 8601 date string
- [ ] Check that images referenced in posts exist and are optimized via `astro:assets`
- [ ] Confirm reading time, tags, and categories are populated if used in the UI
- [ ] Verify no placeholder or template text appears in published posts

### 4.4 Telegram Bot Integration
- [ ] Verify the Telegram bot token is stored as a Cloudflare Worker secret
- [ ] Test that new blog posts trigger a Telegram message within the expected time window
- [ ] Confirm the Telegram message format is social-network-friendly: includes title, excerpt, link, and relevant hashtags in Persian
- [ ] Verify the Telegram bot handles API rate limits (30 messages/second per bot)
- [ ] Test Telegram message rendering for RTL Persian text — confirm it displays correctly in Telegram clients
- [ ] Confirm the Telegram webhook or polling mechanism is active and not failing silently

---

## 5. RTL & Persian Language Support

### 5.1 HTML & CSS RTL Configuration
- [ ] Verify `<html lang="fa" dir="rtl">` is set on all Persian pages
- [ ] Confirm global CSS includes `direction: rtl` and `text-align: right` (or `start`) for Persian content
- [ ] Verify font stack includes a Persian-compatible font (e.g., Vazirmatn, IRANSans) loaded via Astro's `fonts` API or a CDN
- [ ] Check that UI components (navigation, cards, buttons) render correctly in RTL layout
- [ ] Verify that third-party components (chatbot widget, code blocks) do not break RTL layout

### 5.2 Astro i18n Configuration
- [ ] If using Astro's built-in i18n routing, confirm `i18n.locales` includes `'fa'` and `i18n.defaultLocale` is set correctly
- [ ] Verify `i18n.routing.prefixDefaultLocale` is configured as intended (prefix or no-prefix for default locale)
- [ ] Confirm `i18n.fallback` is set if some content is not available in Persian
- [ ] Verify `@astrojs/sitemap` is configured with `i18n.locales` to generate correct `hreflang` alternate links [10](#0-9) [11](#0-10) 

### 5.3 Number & Date Formatting
- [ ] Verify Persian numerals (۰–۹) are used where appropriate, or confirm Latin numerals are intentional
- [ ] Confirm dates are formatted in the Persian calendar (Jalali) if required by the target audience
- [ ] Verify `Intl.DateTimeFormat` or a Jalali library is used consistently for date display

---

## 6. SEO & Metadata

### 6.1 Head Metadata
- [ ] Confirm every page has a unique `<title>` and `<meta name="description">` in Persian
- [ ] Verify canonical URLs are set correctly using `new URL(Astro.url.pathname, Astro.site)`
- [ ] Confirm Open Graph tags (`og:title`, `og:description`, `og:image`, `og:url`) are present on all pages
- [ ] Verify Twitter Card meta tags are present (`twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`)
- [ ] Confirm `<meta name="generator" content={Astro.generator} />` is present (Astro default)
- [ ] Verify `<link rel="icon">` points to a valid favicon (SVG preferred) [12](#0-11) 

### 6.2 Sitemap
- [ ] Confirm `@astrojs/sitemap` is installed and configured in `astro.config.mjs`
- [ ] Verify `site` is set in `astro.config.mjs` (required for sitemap generation)
- [ ] Confirm `sitemap-index.xml` and `sitemap-0.xml` are accessible at the production URL
- [ ] Verify the sitemap is submitted to Google Search Console
- [ ] Check `lastmod` dates in the sitemap reflect actual content update dates
- [ ] Confirm `robots.txt` references the sitemap URL [13](#0-12) 

### 6.3 Structured Data
- [ ] Add JSON-LD `Article` schema to blog post pages with `headline`, `datePublished`, `author`, `inLanguage: "fa"`
- [ ] Add `WebSite` schema to the homepage with `potentialAction` for sitelinks search

---

## 7. Performance & Core Web Vitals

### 7.1 Image Optimization
- [ ] Verify all images use Astro's `<Image />` or `<Picture />` component from `astro:assets` (not raw `<img>` tags)
- [ ] Confirm `imageService` is set to `'cloudflare-binding'` in the adapter config for runtime optimization
- [ ] Verify `image.domains` or `image.remotePatterns` is configured for any remote image sources
- [ ] Confirm `width` and `height` are specified on all `<Image />` components to prevent CLS
- [ ] Verify `alt` text is present on all images (Astro enforces this at build time)
- [ ] Consider enabling `image.responsiveStyles: true` and `layout` attribute for responsive images [14](#0-13) 

### 7.2 JavaScript & CSS Optimization
- [ ] Verify `build.inlineStylesheets: 'auto'` (default) is inlining small stylesheets to reduce request count
- [ ] Confirm client-side JavaScript is minimal — use `client:idle` or `client:visible` for non-critical islands
- [ ] Verify `prefetch: true` is enabled for faster navigation between pages
- [ ] Consider enabling `<ClientRouter />` from `astro:transitions` for SPA-like navigation without full page reloads
- [ ] Audit unused JavaScript with Cloudflare Workers analytics or browser DevTools [15](#0-14) 

### 7.3 Caching Strategy
- [ ] Configure `cache` and `routeRules` in `astro.config.mjs` (stable in Astro v7) for SSR response caching
- [ ] Set appropriate `maxAge` and `swr` (stale-while-revalidate) for blog listing and post pages
- [ ] Verify Cloudflare's edge cache is being utilized for static assets (check `CF-Cache-Status: HIT` headers)
- [ ] Confirm the Worker's `cache` binding is enabled in `wrangler.jsonc` if using Cloudflare's Cache API [16](#0-15) 

### 7.4 Core Web Vitals Targets
- [ ] Run Lighthouse or PageSpeed Insights on the homepage and a blog post page
- [ ] Verify LCP (Largest Contentful Paint) < 2.5s — optimize hero images and fonts
- [ ] Verify CLS (Cumulative Layout Shift) < 0.1 — ensure image dimensions are specified
- [ ] Verify INP (Interaction to Next Paint) < 200ms — minimize main-thread blocking JS
- [ ] Use Cloudflare Web Analytics or a third-party tool (Plausible, Fathom) for ongoing monitoring

---

## 8. Security

### 8.1 Content Security Policy
- [ ] Enable `security.csp` in `astro.config.mjs` (available since Astro v6, stable)
- [ ] Configure `security.csp.directives` to include `default-src`, `img-src`, `connect-src` for the FlowiseAI API domain
- [ ] Add FlowiseAI API domain to `connect-src` directive
- [ ] Note: `security.csp` is incompatible with `<ClientRouter />` (View Transitions) — choose one or use native browser View Transitions API [17](#0-16) [18](#0-17) 

### 8.2 HTTP Security Headers
- [ ] Add security headers via Cloudflare `_headers` file or Astro middleware:
  - `X-Content-Type-Options: nosniff`
  - `X-Frame-Options: DENY` (or `SAMEORIGIN` if embedding is needed)
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Permissions-Policy` to restrict unnecessary browser APIs
- [ ] Verify `Strict-Transport-Security` (HSTS) is set (Cloudflare can enforce this)
- [ ] Confirm `security.allowedDomains` is configured if the site is behind a reverse proxy to prevent header injection [19](#0-18) 

### 8.3 Authentication & Access Control
- [ ] Verify all API keys and secrets are stored as Cloudflare Worker secrets, not in environment variables committed to the repo
- [ ] Confirm the FlowiseAI admin panel is not publicly accessible
- [ ] Verify Cloudflare WAF (Web Application Firewall) rules are active
- [ ] Enable Cloudflare Bot Fight Mode to reduce malicious traffic

---

## 9. Branding, UI & UX

### 9.1 Visual Identity
- [ ] Verify the logo renders correctly at all sizes and on all pages (SVG preferred for scalability)
- [ ] Confirm favicon is present in multiple sizes (`favicon.ico`, `favicon.svg`, Apple touch icon)
- [ ] Verify brand colors, typography, and spacing are consistent across all pages
- [ ] Confirm the Persian font renders correctly across major browsers (Chrome, Firefox, Safari)
- [ ] Use Astro's `fonts` API with `fontProviders.google()` or `fontProviders.fontsource()` for optimized font loading with preloading and fallback generation [20](#0-19) 

### 9.2 Responsive Design
- [ ] Test all pages on mobile (320px–480px), tablet (768px), and desktop (1280px+) viewports
- [ ] Verify the chatbot widget is usable on mobile without obscuring content
- [ ] Confirm navigation is accessible on mobile (hamburger menu or equivalent)
- [ ] Verify RTL layout is correct on all breakpoints

### 9.3 Accessibility
- [ ] Run an automated accessibility audit (axe, Lighthouse) on all key pages
- [ ] Verify all interactive elements have visible focus indicators
- [ ] Confirm color contrast ratios meet WCAG 2.1 AA (4.5:1 for normal text)
- [ ] Verify `lang="fa"` and `dir="rtl"` are set for screen reader compatibility
- [ ] Confirm all form inputs (chatbot, search) have associated `<label>` elements

### 9.4 Page Completeness Audit
- [ ] Verify a custom `404.astro` page exists and is styled consistently with the site
- [ ] Confirm a `500.astro` error page exists for server errors
- [ ] Verify the homepage, blog listing, individual blog post, and about/contact pages are all functional
- [ ] Confirm all internal links resolve correctly (no broken links)
- [ ] Verify external links open in a new tab with `rel="noopener noreferrer"`

---

## 10. GitHub Repository & CI/CD

### 10.1 Repository Health
- [ ] Verify `.gitignore` excludes `dist/`, `node_modules/`, `.dev.vars`, `.wrangler/`, `.astro/`
- [ ] Confirm `package.json` has correct `build`, `dev`, and `preview` scripts
- [ ] Verify `astro.config.mjs` is committed and up to date
- [ ] Confirm `wrangler.jsonc` is committed (if using custom bindings)
- [ ] Verify dependency versions are pinned or use lockfile (`package-lock.json` or `pnpm-lock.yaml`)

### 10.2 Automated Deployment
- [ ] Confirm GitHub Actions or Cloudflare's Git integration triggers a build on every push to `main`
- [ ] Verify the build succeeds without errors or warnings in CI
- [ ] Confirm deployment previews are generated for pull requests (if configured)
- [ ] Verify `astro check` runs in CI to catch TypeScript errors before deployment
- [ ] Confirm the AI content generation workflow has error handling and alerts on failure

### 10.3 Backup & Recovery
- [ ] Verify the GitHub repository is the source of truth for all site content and configuration
- [ ] Confirm Cloudflare KV data (sessions) is not the only copy of critical data
- [ ] Verify a rollback procedure exists (e.g., revert commit and redeploy)
- [ ] Confirm Cloudflare Workers deployment history is retained for rollback

---

## 11. Technical Debt & Upgrade Recommendations

### 11.1 Deprecated APIs to Remove
- [ ] Remove any usage of `Astro.locals.runtime` — replace with `env` from `cloudflare:workers`, `Astro.request.cf`, and `Astro.locals.cfContext`
- [ ] Remove deprecated `markdown.remarkPlugins`, `markdown.rehypePlugins`, `markdown.remarkRehype` if not using `@astrojs/markdown-remark`
- [ ] Remove `output: 'hybrid'` if present — it was removed in Astro v5; use `output: 'static'` instead
- [ ] Remove `experimental.assets`, `experimental.i18n`, `experimental.viewTransitions` flags if present — all are now stable
- [ ] Remove `experimental.csp` — replaced by `security.csp` (stable in v6)
- [ ] Remove `experimental.cache` and `experimental.routeRules` — replaced by top-level `cache` and `routeRules` (stable in v7) [7](#0-6) [16](#0-15) 

### 11.2 Recommended Astro Features to Adopt
- [ ] **Route Caching** (`cache` + `routeRules`): Cache blog post pages at the edge with `maxAge` and `swr` for significant performance gains on Cloudflare
- [ ] **Fonts API** (`fonts` config): Replace manual Google Fonts `<link>` tags with Astro's built-in font optimization (preloading, fallback generation, self-hosting)
- [ ] **SVG Optimization** (`experimental.svgOptimizer`): Enable SVGO-based SVG optimization for logos and icons
- [ ] **Content Security Policy** (`security.csp`): Enable for XSS protection, especially important given the AI chatbot integration
- [ ] **Responsive Images** (`image.responsiveStyles: true`): Automatically generate `srcset` and `sizes` for blog post images
- [ ] **`astro check` in CI**: Add to the GitHub Actions workflow to catch type errors before deployment [21](#0-20) [22](#0-21) 

### 11.3 Cloudflare-Specific Optimizations
- [ ] Enable Cloudflare's **Automatic Platform Optimization (APO)** if applicable
- [ ] Configure **Cloudflare Cache Rules** for static assets with long TTLs (`Cache-Control: public, max-age=31536000, immutable` for hashed assets)
- [ ] Enable **Cloudflare Zaraz** for privacy-friendly analytics instead of client-side tracking scripts
- [ ] Review **Cloudflare Workers Analytics** for CPU time, error rates, and request volume
- [ ] Consider **Cloudflare D1** for structured blog metadata storage if KV becomes a bottleneck
- [ ] Enable **Cloudflare R2** for storing AI-generated media assets if volume grows

---

## 12. Monitoring & Ongoing Health

- [ ] Set up Cloudflare Workers error alerting (via Cloudflare dashboard notifications)
- [ ] Configure uptime monitoring for the homepage, chatbot API endpoint, and a sample blog post URL
- [ ] Monitor Cloudflare Workers CPU time — stay well below the 30-second limit per request
- [ ] Set up alerts for failed AI content generation pipeline runs
- [ ] Monitor Telegram bot delivery success rate
- [ ] Review Cloudflare Web Analytics weekly for traffic anomalies, top pages, and error rates
- [ ] Schedule a quarterly review of Astro and `@astrojs/cloudflare` changelogs for breaking changes and new features [23](#0-22)

### Citations

**File:** packages/astro/src/core/config/schemas/base.ts (L61-119)
```typescript
export const ASTRO_CONFIG_DEFAULTS = {
	root: '.',
	srcDir: './src',
	publicDir: './public',
	outDir: './dist',
	cacheDir: './node_modules/.astro',
	base: '/',
	trailingSlash: 'ignore',
	build: {
		format: 'directory',
		client: './client/',
		server: './server/',
		assets: '_astro',
		serverEntry: 'entry.mjs',
		redirects: true,
		inlineStylesheets: 'auto',
		concurrency: 1,
	},
	image: {
		endpoint: { entrypoint: undefined, route: '/_image' },
		service: { entrypoint: 'astro/assets/services/sharp', config: {} },
		dangerouslyProcessSVG: false,
		responsiveStyles: false,
	},
	devToolbar: {
		enabled: true,
	},
	compressHTML: 'jsx',
	server: {
		host: false,
		port: 4321,
		open: false,
		allowedHosts: [],
	},
	integrations: [],
	markdown: markdownConfigDefaults,
	vite: {},
	legacy: {
		collectionsBackwardsCompat: false,
	},
	redirects: {},
	security: {
		checkOrigin: true,
		allowedDomains: [],
		csp: false,
		actionBodySizeLimit: 1024 * 1024,
		serverIslandBodySizeLimit: 1024 * 1024,
	},
	env: {
		schema: {},
		validateSecrets: false,
	},
	prerenderConflictBehavior: 'warn',
	fetchFile: 'fetch',
	experimental: {
		clientPrerender: false,
		contentIntellisense: false,
		chromeDevtoolsWorkspace: false,
	},
```

**File:** packages/astro/src/core/config/schemas/base.ts (L494-541)
```typescript
	),
	security: z
		.object({
			checkOrigin: z.boolean().default(ASTRO_CONFIG_DEFAULTS.security.checkOrigin),
			allowedDomains: z
				.array(
					z.object({
						hostname: z.string().optional(),
						protocol: z.string().optional(),
						port: z.string().optional(),
					}),
				)
				.optional()
				.default(ASTRO_CONFIG_DEFAULTS.security.allowedDomains),
			actionBodySizeLimit: z
				.number()
				.optional()
				.default(ASTRO_CONFIG_DEFAULTS.security.actionBodySizeLimit),
			serverIslandBodySizeLimit: z
				.number()
				.optional()
				.default(ASTRO_CONFIG_DEFAULTS.security.serverIslandBodySizeLimit),
			csp: z
				.union([
					z.boolean().optional().default(ASTRO_CONFIG_DEFAULTS.security.csp),
					z.object({
						algorithm: cspAlgorithmSchema,
						directives: z.array(allowedDirectivesSchema).optional(),
						styleDirective: z
							.object({
								resources: z.array(z.string()).optional(),
								hashes: z.array(cspHashSchema).optional(),
							})
							.optional(),
						scriptDirective: z
							.object({
								resources: z.array(z.string()).optional(),
								hashes: z.array(cspHashSchema).optional(),
								strictDynamic: z.boolean().optional(),
							})
							.optional(),
					}),
				])
				.optional()
				.default(ASTRO_CONFIG_DEFAULTS.security.csp),
		})
		.optional()
		.default(ASTRO_CONFIG_DEFAULTS.security),
```

**File:** packages/integrations/cloudflare/test/fixtures/vite-plugin/astro.config.mjs (L1-53)
```javascript
// @ts-check
import cloudflare from '@astrojs/cloudflare';
import { defineConfig, envField, fontProviders } from 'astro/config';
import mdx from '@astrojs/mdx';
import { fileURLToPath } from 'node:url';
import react from '@astrojs/react';
import vue from "@astrojs/vue"
import svelte from "@astrojs/svelte"

export default defineConfig({
	adapter: cloudflare({
		imageService: 'cloudflare-binding',
		sessionKVBindingName: "SESSION"
	}),
	build: {
		inlineStylesheets: 'never'
	},
	vite: {
		resolve: {
			alias: {
				'@images': fileURLToPath(new URL('./images', import.meta.url)),
			},
		},
	},
	i18n: {
		defaultLocale: "en",
		locales: ["en", "fr"],
		routing: {
			prefixDefaultLocale: false,
			redirectToDefaultLocale: false
		},
		fallback: {
			"fr": "en"
		}
	},
	integrations: [mdx(), react(), vue(), svelte()],
	env: {
		schema: {
			FOO: envField.string({ context: 'server', access: 'public' }),
			BAR: envField.string({ context: 'client', access: 'public' }),
			SECRET: envField.string({ context: 'server', access: 'secret' }),
		}
	},
	prefetch: true,
	fonts: [{
		provider: fontProviders.google(),
		name: "Roboto",
		cssVariable: "--font-roboto"
	}],
	experimental: {
		chromeDevtoolsWorkspace: true
	},
});
```

**File:** packages/integrations/cloudflare/src/index.ts (L77-119)
```typescript
export interface Options
	extends Pick<
		PluginConfig,
		'auxiliaryWorkers' | 'configPath' | 'inspectorPort' | 'persistState' | 'remoteBindings'
	> {
	/** Options for handling images. */
	imageService?: ImageServiceConfig;

	/**
	 * By default, Astro will be configured to use Cloudflare KV to store session data. The KV namespace
	 * will be automatically provisioned when you deploy.
	 *
	 * By default, the binding is named `SESSION`, but you can override this by providing a different name here.
	 * If you define the binding manually in your wrangler config, Astro will use your configuration instead.
	 *
	 * See https://developers.cloudflare.com/workers/wrangler/configuration/#automatic-provisioning for more details.
	 */
	sessionKVBindingName?: string;

	/**
	 * When `imageService` is set to `cloudflare-binding`, the Cloudflare Images binding will be used
	 * to transform images. The binding will be automatically configured for you.
	 *
	 * By default, the binding is named `IMAGES`, but you can override this by providing a different name here.
	 * If you define the binding manually in your wrangler config, Astro will use your configuration instead.
	 *
	 * See https://developers.cloudflare.com/images/transform-images/bindings/ for more details.
	 */
	imagesBindingName?: string;

	/**
	 * Controls which runtime is used for prerendering static pages at build time.
	 *
	 * - `'workerd'` (default): Uses Cloudflare's workerd runtime.
	 * - `'node'`: Uses Astro's default node prerender environment.
	 */
	prerenderEnvironment?: 'workerd' | 'node';

	experimental?: Pick<
		NonNullable<PluginConfig['experimental']>,
		'headersAndRedirectsDevModeSupport'
	>;
}
```

**File:** packages/integrations/cloudflare/src/index.ts (L280-303)
```typescript
								enforce: 'pre',
								resolveId: {
									filter: {
										id: /^cloudflare:/,
									},
									handler(id) {
										return { id, external: true };
									},
								},
							},
							{
								name: '@astrojs/cloudflare:environment',
								configEnvironment(environmentName, _options) {
									// Skip dependency pre-bundling during type generation (see `isTypeGenPhase` above).
									if (isTypeGenPhase) {
										return { optimizeDeps: { noDiscovery: true, include: [] } };
									}
									const isServerEnvironment = ['astro', 'ssr', 'prerender'].includes(
										environmentName,
									);
									if (isServerEnvironment && !_options.optimizeDeps?.noDiscovery) {
										return {
											optimizeDeps: {
												include: [
```

**File:** packages/integrations/cloudflare/src/wrangler.ts (L1-79)
```typescript
import type { PluginConfig, WorkerConfig } from '@cloudflare/vite-plugin';

export const DEFAULT_SESSION_KV_BINDING_NAME = 'SESSION';
export const DEFAULT_IMAGES_BINDING_NAME = 'IMAGES';
export const DEFAULT_ASSETS_BINDING_NAME = 'ASSETS';

// Default compatibility date used when the user doesn't set one in their wrangler config.
// The @cloudflare/vite-plugin falls back to today's date, but that can exceed the maximum
// date supported by the bundled workerd binary (which has a ~7 day buffer from its build date),
// causing ERR_RUNTIME_FAILURE. A hard-coded date avoids this issue.
// This should be updated when upgrading wrangler/workerd dependencies.
const DEFAULT_COMPATIBILITY_DATE = '2026-04-15';

interface CloudflareConfigOptions {
	sessionKVBindingName?: string | undefined;
	needsSessionKVBinding?: boolean;
	imagesBindingName?: string | false | undefined;
	needsWorkerCache?: boolean;
}

type KVNamespace = NonNullable<WorkerConfig['kv_namespaces']>[number];

/**
 * Returns a config customizer that sets up the Astro Cloudflare defaults.
 * Sets the main entrypoint and adds bindings for auto-provisioning.
 */
export function cloudflareConfigCustomizer(
	options?: CloudflareConfigOptions,
): (config: Partial<WorkerConfig>) => Partial<WorkerConfig> {
	const sessionKVBindingName = options?.sessionKVBindingName ?? DEFAULT_SESSION_KV_BINDING_NAME;
	const needsSessionKVBinding = options?.needsSessionKVBinding ?? true;
	const imagesBindingName =
		options?.imagesBindingName === false
			? undefined
			: (options?.imagesBindingName ?? DEFAULT_IMAGES_BINDING_NAME);
	const needsWorkerCache = options?.needsWorkerCache ?? false;

	const customizer = (config: Partial<WorkerConfig>): Partial<WorkerConfig> => {
		const getNonInheritableBindings = (
			nonInheritableConfig: WorkerConfig['previews'],
		): WorkerConfig['previews'] => {
			const hasSessionBinding = nonInheritableConfig?.kv_namespaces?.some(
				(kv: KVNamespace) => kv.binding === sessionKVBindingName,
			);
			const hasImagesBinding = nonInheritableConfig?.images?.binding !== undefined;

			return {
				kv_namespaces:
					!needsSessionKVBinding || hasSessionBinding
						? undefined
						: [{ binding: sessionKVBindingName }],
				images:
					hasImagesBinding || !imagesBindingName
						? undefined
						: {
								binding: imagesBindingName,
							},
			};
		};

		const hasAssetsBinding = config.assets?.binding !== undefined;

		return {
			...getNonInheritableBindings(config),
			compatibility_date: config.compatibility_date ?? DEFAULT_COMPATIBILITY_DATE,
			main: config.main ?? '@astrojs/cloudflare/entrypoints/server',
			assets: hasAssetsBinding
				? undefined
				: {
						binding: DEFAULT_ASSETS_BINDING_NAME,
					},
			// Enable the Worker caching layer when a Cloudflare cache provider is configured
			cache: needsWorkerCache && !config.cache?.enabled ? { enabled: true } : undefined,
			previews: getNonInheritableBindings(config.previews),
		};
	};

	return customizer satisfies PluginConfig['config'];
}
```

**File:** packages/integrations/cloudflare/test/fixtures/custom-entryfile/wrangler.jsonc (L1-40)
```text
{
  "compatibility_date": "2026-01-28",
  "main": "./src/worker.ts",
  "name": "astro-cloudflare-custom-entryfile",
  "assets": {
    "directory": "./dist",
    "binding": "ASSETS"
  },
  "queues": {
    "producers": [
      {
        "queue": "MY-QUEUE-NAME",
        "binding": "MY_QUEUE"
      }
    ],
    "consumers": [
      {
        "queue": "MY-QUEUE-NAME",
        "max_batch_size": 10,
        "max_batch_timeout": 5
      }
    ]
  },
  "durable_objects": {
    "bindings": [
      {
        "name": "MY_DURABLE_OBJECT",
        "class_name": "MyDurableObject"
      }
    ]
  },
  "migrations": [
    {
      "tag": "v1",
      "new_sqlite_classes": [
        "MyDurableObject"
      ]
    }
  ]
}
```

**File:** packages/integrations/cloudflare/CHANGELOG.md (L604-618)
```markdown
- [#14306](https://github.com/withastro/astro/pull/14306) [`141c4a2`](https://github.com/withastro/astro/commit/141c4a26419fe5bb4341953ea5a0a861d9b398c0) Thanks [@ematipico](https://github.com/ematipico)! - Development server now runs in workerd

  `astro dev` now runs your Cloudflare application using Cloudflare's workerd runtime instead of Node.js. This means your development environment is now a near-exact replica of your production environment—the same JavaScript engine, the same APIs, the same behavior. You'll catch issues during development that would have only appeared in production, and features like Durable Objects, Workers Analytics Engine, and R2 bindings work exactly as they do on Cloudflare's platform.

  #### New runtime

  Previously, `Astro.locals.runtime` provided access to Cloudflare-specific APIs. These APIs have now moved to align with Cloudflare's native patterns.

  #### What should I do?

  Update occurrences of `Astro.locals.runtime`:
  - `Astro.locals.runtime.env` → Import `env` from `cloudflare:workers`
  - `Astro.locals.runtime.cf` → Access via `Astro.request.cf`
  - `Astro.locals.runtime.caches` → Use the global `caches` object
  - `Astro.locals.runtime` (for `ExecutionContext`) → Use `Astro.locals.cfContext`
```

**File:** packages/integrations/cloudflare/CHANGELOG.md (L695-697)
```markdown
- [#15480](https://github.com/withastro/astro/pull/15480) [`e118214`](https://github.com/withastro/astro/commit/e118214eeaaa27384528c882af8b68e8daa61e2c) Thanks [@alexanderniebuhr](https://github.com/alexanderniebuhr)! - Drops official support for Cloudflare Pages in favor of Cloudflare Workers

  The Astro Cloudflare adapter now only supports deployment to Cloudflare Workers by default in order to comply with Cloudflare's recommendations for new projects. If you are currently deploying to Cloudflare Pages, consider [migrating to Workers by following the Cloudflare guide](https://developers.cloudflare.com/workers/static-assets/migration-guides/migrate-from-pages/) for an optimal experience and full feature support.
```

**File:** packages/integrations/cloudflare/CHANGELOG.md (L726-752)
```markdown
- [#15037](https://github.com/withastro/astro/pull/15037) [`8641805`](https://github.com/withastro/astro/commit/8641805289d2cd852458b023c83da54bf67cd579) Thanks [@matthewp](https://github.com/matthewp)! - The Wrangler configuration file is now optional. If you don't have custom Cloudflare bindings (KV, D1, Durable Objects, etc.), Astro will automatically generate a default configuration for you.

  ##### What should I do?

  If your `wrangler.jsonc` only contains basic configuration like this:

  ```jsonc
  {
    "main": "@astrojs/cloudflare/entrypoints/server",
    "compatibility_date": "2026-01-28",
    "assets": {
      "directory": "./dist",
      "binding": "ASSETS",
    },
  }
  ```

  You can safely delete the file. Astro will handle this configuration automatically.

  You only need a wrangler config file if you're using:
  - KV namespaces
  - D1 databases
  - Durable Objects
  - R2 buckets
  - Environment variables
  - Custom compatibility flags
  - Other Cloudflare-specific features
```

**File:** packages/astro/e2e/fixtures/content-collections/src/content.config.ts (L1-11)
```typescript
import { defineCollection } from "astro:content";
import { glob } from "astro/loaders";


const posts = defineCollection({
	loader: glob({ pattern: '**/*.{md,mdx}', base: './src/content/posts' }),
});

export const collections = { posts };


```

**File:** packages/astro/src/types/public/config.ts (L719-762)
```typescript
		serverIslandBodySizeLimit?: number;

		/**
		 * @docs
		 * @name security.csp
		 * @kind h4
		 * @type {boolean | object}
		 * @default `false`
		 * @version 6.0.0
		 * @description
		 *
		 * Enables support for [Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP) to help minimize certain types of security threats by controlling which resources a document is allowed to load. This provides additional protection against [cross-site scripting (XSS)](https://developer.mozilla.org/en-US/docs/Glossary/Cross-site_scripting) attacks.
		 *
		 * Enabling this feature adds additional security to Astro's handling of processed and bundled scripts and styles by default, and allows you to further configure these, and additional, content types.
		 *
		 * This feature comes with some limitations:
		 * - External scripts and external styles are not supported out of the box, but you can [provide your own hashes](https://docs.astro.build/en/reference/configuration-reference/#securitycspscriptdirectivehashes).
		 * - [Astro's view transitions](https://docs.astro.build/en/guides/view-transitions/) using the `<ClientRouter />` are not supported, but you can [consider migrating to the browser native View Transition API](https://events-3bg.pages.dev/jotter/astro-view-transitions/) instead if you are not using Astro's enhancements to the native View Transitions and Navigation APIs.
		 * - Shiki isn't currently supported. By design, Shiki functions use inline styles that cannot work with Astro CSP implementation. Consider [using `<Prism />`](https://docs.astro.build/en/guides/syntax-highlighting/#prism-) when your project requires both CSP and syntax highlighting.
		 * - `unsafe-inline` directives are incompatible with Astro's CSP implementation. By default, Astro will emit hashes for all its bundled scripts (e.g. client islands) and all modern browsers will automatically reject `unsafe-inline` when it occurs in a directive with a hash or nonce.
		 *
		 * :::note
		 * Due to the nature of the Vite dev server, this feature isn't supported while working in `dev` mode. Instead, you can test this in your Astro project using `build` and `preview`.
		 * :::
		 *
		 * When enabled, Astro will add a `<meta>` element inside the `<head>` element of each page.
		 * This element will have the `http-equiv="content-security-policy"` attribute, and the `content` attribute will provide values for the `script-src` and `style-src` [directives](https://docs.astro.build/en/reference/configuration-reference/#securitycspdirectives) based on the script and styles used in the page.
		 *
		 * ```html
		 *
		 * <head>
		 *   <meta
		 *     http-equiv="content-security-policy"
		 *     content="
		 *     script-src 'self' 'sha256-somehash';
		 *     style-src 'self' 'sha256-somehash';
		 *     "
		 *   >
		 * </head>
		 * ```
		 *
		 * You can further customize the `<meta>` element by enabling this feature with a configuration object that includes additional options.
		 */
		csp?:
```

**File:** packages/astro/src/types/public/config.ts (L1709-1753)
```typescript
	prefetch?:
		| boolean
		| {
				/**
				 * @docs
				 * @name prefetch.prefetchAll
				 * @type {boolean}
				 * @description
				 * Enable prefetching for all links, including those without the `data-astro-prefetch` attribute.
				 * This value defaults to `true` when using the `<ClientRouter />` router. Otherwise, the default value is `false`.
				 *
				 * ```js
				 * prefetch: {
				 * 	prefetchAll: true
				 * }
				 * ```
				 *
				 * When set to `true`, you can disable prefetching individually by setting `data-astro-prefetch="false"` on any individual links.
				 *
				 * ```html
				 * <a href="/about" data-astro-prefetch="false">About</a>
				 *```
				 */
				prefetchAll?: boolean;

				/**
				 * @docs
				 * @name prefetch.defaultStrategy
				 * @type {'tap' | 'hover' | 'viewport' | 'load'}
				 * @default `'hover'`
				 * @description
				 * The default prefetch strategy to use when the `data-astro-prefetch` attribute is set on a link with no value.
				 *
				 * - `'tap'`: Prefetch just before you click on the link.
				 * - `'hover'`: Prefetch when you hover over or focus on the link. (default)
				 * - `'viewport'`: Prefetch as the links enter the viewport.
				 * - `'load'`: Prefetch all links on the page after the page is loaded.
				 *
				 * You can override this default value and select a different strategy for any individual link by setting a value on the attribute.
				 *
				 * ```html
				 * <a href="/about" data-astro-prefetch="viewport">About</a>
				 * ```
				 */
				defaultStrategy?: 'tap' | 'hover' | 'viewport' | 'load';
```

**File:** packages/astro/src/types/public/config.ts (L2332-2396)
```typescript
	i18n?: {
		/**
		 * @docs
		 * @name i18n.locales
		 * @type {Locales}
		 * @version 3.5.0
		 * @description
		 *
		 * A list of all locales supported by the website. This is a required field.
		 *
		 * Languages can be listed either as individual codes (e.g. `['en', 'es', 'pt-br']`) or mapped to a shared `path` of codes (e.g.  `{ path: "english", codes: ["en", "en-US"]}`). These codes will be used to determine the URL structure of your deployed site.
		 *
		 * No particular language code format or syntax is enforced, but your project folders containing your content files must match exactly the `locales` items in the list. In the case of multiple `codes` pointing to a custom URL path prefix, store your content files in a folder with the same name as the `path` configured.
		 */
		locales: [TLocales] extends [never] ? Locales : TLocales;

		/**
		 * @docs
		 * @name i18n.defaultLocale
		 * @type {string}
		 * @version 3.5.0
		 * @description
		 *
		 * The default locale of your website/application, that is one of the specified `locales`. This is a required field.
		 *
		 * No particular language format or syntax is enforced, but we suggest using lower-case and hyphens as needed (e.g. "es", "pt-br") for greatest compatibility.
		 */
		defaultLocale: [TLocales] extends [never] ? string : NormalizeLocales<NoInfer<TLocales>>;

		/**
		 * @docs
		 * @name i18n.fallback
		 * @type {Record<string, string>}
		 * @version 3.5.0
		 * @description
		 *
		 * The fallback strategy when navigating to pages that do not exist (e.g. a translated page has not been created).
		 *
		 * Use this object to declare a fallback `locale` route for each language you support. If no fallback is specified, then unavailable pages will return a 404.
		 *
		 * ##### Example
		 *
		 * The following example configures your content fallback strategy to redirect unavailable pages in `/pt-br/` to their `es` version, and unavailable pages in `/fr/` to their `en` version. Unavailable `/es/` pages will return a 404.
		 *
		 * ```js
		 * export default defineConfig({
		 * 	i18n: {
		 * 		defaultLocale: "en",
		 * 		locales: ["en", "fr", "pt-br", "es"],
		 * 		fallback: {
		 * 			pt: "es",
		 * 		  fr: "en"
		 * 		}
		 * 	}
		 * })
		 * ```
		 */
		fallback?: [TLocales] extends [never]
			? Record<string, string>
			: {
					[Locale in NormalizeLocales<NoInfer<TLocales>>]?: Exclude<
						NormalizeLocales<NoInfer<TLocales>>,
						Locale
					>;
				};
```

**File:** packages/astro/src/types/public/config.ts (L2650-2684)
```typescript
	};

	/**
	 * @docs
	 * @kind heading
	 * @name fonts
	 * @type {Array<FontFamily>}
	 * @default `[]`
	 * @version 6.0.0
	 * @description
	 * Configures fonts and allows you to specify some customization options on a per-font basis.
	 *
	 * See our guide for more information on [using custom fonts in Astro](https://docs.astro.build/en/guides/fonts/).
	 */

	/**
	 * @docs
	 * @name font.provider
	 * @type {FontProvider}
	 * @version 6.0.0
	 * @description
	 * The source of your font files. You can use a [built-in provider](https://docs.astro.build/en/reference/font-provider-reference/#built-in-providers) or write your own [custom provider](https://docs.astro.build/en/reference/font-provider-reference/#building-a-font-provider):
	 *
	 * ```js
	 * import { defineConfig, fontProviders } from "astro/config";
	 *
	 * export default defineConfig({
	 *   fonts: [{
	 *     provider: fontProviders.google(),
	 *     name: "Roboto",
	 *     cssVariable: "--font-roboto"
	 *   }]
	 * });
	 * ```
	 */
```

**File:** packages/integrations/sitemap/src/index.ts (L21-55)
```typescript
export type SitemapOptions =
	| {
			filenameBase?: string;
			filter?(page: string): boolean;
			customSitemaps?: string[];
			customPages?: string[];

			i18n?: {
				defaultLocale: string;
				locales: Record<string, string>;
			};
			// number of entries per sitemap file
			entryLimit?: number;
			// sitemap specific
			changefreq?: ChangeFreq;
			lastmod?: Date;
			priority?: number;

			// called for each sitemap item just before to save them on disk, sync or async
			serialize?(item: SitemapItem): SitemapItem | Promise<SitemapItem | undefined> | undefined;

			xslURL?: string;
			chunks?: Record<
				string,
				(item: SitemapItem) => SitemapItem | Promise<SitemapItem | undefined> | undefined
			>;
			// namespace configuration
			namespaces?: {
				news?: boolean;
				xhtml?: boolean;
				image?: boolean;
				video?: boolean;
			};
	  }
	| undefined;
```

**File:** packages/integrations/sitemap/src/index.ts (L98-106)
```typescript
			'astro:build:done': async ({ dir, pages, logger }) => {
				try {
					if (!config.site) {
						logger.warn(
							'The Sitemap integration requires the `site` astro.config option. Skipping.',
						);
						return;
					}

```

**File:** packages/astro/e2e/fixtures/actions-blog/src/components/BaseHead.astro (L1-42)
```text
---
// Import the global.css file here so that it is included on
// all pages through the use of the <BaseHead /> component.
import '../styles/global.css';

interface Props {
	title: string;
	description: string;
	image?: string;
}

const canonicalURL = new URL(Astro.url.pathname, Astro.site);

const { title, description, image = '/blog-placeholder-1.jpg' } = Astro.props;
---

<!-- Global Metadata -->
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
<meta name="generator" content={Astro.generator} />

<!-- Canonical URL -->
<link rel="canonical" href={canonicalURL} />

<!-- Primary Meta Tags -->
<title>{title}</title>
<meta name="description" content={description} />

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

**File:** packages/integrations/cloudflare/src/utils/image-config.ts (L38-115)
```typescript
// The default Astro dev image endpoint uses node:fs which is unavailable in workerd.
// Use the generic endpoint instead, which loads images via fetch through the dev server.
const GENERIC_ENDPOINT = { entrypoint: 'astro/assets/endpoint/generic' };

// Passthrough endpoint that serves original images via the ASSETS binding.
const CLOUDFLARE_PASSTHROUGH_ENDPOINT = {
	entrypoint: '@astrojs/cloudflare/image-passthrough-endpoint',
};

// Workerd-compatible image service stub: baseService (no sharp) + passthrough transform.
// Used by both `compile` and `cloudflare-binding` for URL generation in workerd.
const WORKERD_IMAGE_SERVICE = { entrypoint: '@astrojs/cloudflare/image-service-workerd' };

export function setImageConfig(
	service: ImageServiceConfig | undefined,
	config: AstroConfig['image'],
	command: HookParameters<'astro:config:setup'>['command'],
	logger: AstroIntegrationLogger,
) {
	const { buildService, runtimeService } = normalizeImageServiceConfig(service);

	switch (buildService) {
		case 'passthrough':
			return {
				...config,
				service: passthroughImageService(),
				endpoint: command === 'dev' ? GENERIC_ENDPOINT : CLOUDFLARE_PASSTHROUGH_ENDPOINT,
			};

		case 'cloudflare':
			// The external Cloudflare image service generates `/cdn-cgi/image/...` URLs,
			// which only work on Cloudflare's production edge network. In dev mode,
			// fall back to passthrough so images render normally without transformation.
			if (command === 'dev') {
				return {
					...config,
					service: passthroughImageService(),
					endpoint: GENERIC_ENDPOINT,
				};
			}
			return {
				...config,
				service: { entrypoint: '@astrojs/cloudflare/image-service' },
			};

		case 'cloudflare-binding':
			return {
				...config,
				service: WORKERD_IMAGE_SERVICE,
				endpoint: {
					entrypoint: '@astrojs/cloudflare/image-transform-endpoint',
				},
			};

		case 'compile':
			return {
				...config,
				service: WORKERD_IMAGE_SERVICE,
				// Dev: IMAGES binding (via Cloudflare Vite plugin) for real transforms.
				// Build: endpoint depends on runtime - `cloudflare-binding` uses IMAGES, `passthrough` uses generic.
				endpoint:
					command === 'dev' || runtimeService === 'cloudflare-binding'
						? { entrypoint: '@astrojs/cloudflare/image-transform-endpoint' }
						: CLOUDFLARE_PASSTHROUGH_ENDPOINT,
			};

		case 'custom':
			return { ...config };

		default:
			if (config.service.entrypoint === 'astro/assets/services/sharp') {
				logger.warn(
					`The current configuration does not support image optimization. To allow your project to build with the original, unoptimized images, the image service has been automatically switched to the 'passthrough' option. See https://docs.astro.build/en/reference/configuration-reference/#imageservice`,
				);
				return { ...config, service: passthroughImageService() };
			}
			return { ...config };
	}
```

**File:** packages/astro/CHANGELOG.md (L269-299)
```markdown
- [#17116](https://github.com/withastro/astro/pull/17116) [`f95e58e`](https://github.com/withastro/astro/commit/f95e58eaa6a6d7ac02f84193b485471f0cd14de6) Thanks [@ascorbic](https://github.com/ascorbic)! - Stabilizes route caching, removing the `experimental.cache` and `experimental.routeRules` flags and replacing them with the top-level `cache` and `routeRules` configuration options.

  Route caching, introduced experimentally in v6.0.0, is now stable. It gives you a platform-agnostic way to cache responses from [on-demand rendered](https://docs.astro.build/en/guides/on-demand-rendering/) pages and endpoints, based on standard HTTP caching semantics.

  Update your config to move `cache` and `routeRules` out of the `experimental` block:

  ```diff
  // astro.config.mjs
  import { defineConfig, memoryCache } from 'astro/config';

  export default defineConfig({
  -  experimental: {
  -    cache: {
  -      provider: memoryCache(),
  -    },
  -    routeRules: {
  -      '/blog/[...path]': { maxAge: 300, swr: 60 },
  -    },
  -  },
  +  cache: {
  +    provider: memoryCache(),
  +  },
  +  routeRules: {
  +    '/blog/[...path]': { maxAge: 300, swr: 60 },
  +  },
  });
  ```

  Set caching directives in your routes with `Astro.cache` (in `.astro` pages) or `context.cache` (in API routes and middleware), and Astro translates them into the appropriate headers or runtime behavior depending on your configured cache provider. You can also define cache rules for routes declaratively in your config using `routeRules`, without modifying route code.

  See the [route caching guide](https://docs.astro.build/en/guides/caching/) for more information.
```

**File:** packages/astro/CHANGELOG.md (L1388-1409)
```markdown
- [#16333](https://github.com/withastro/astro/pull/16333) [`0f7c3c8`](https://github.com/withastro/astro/commit/0f7c3c8e3dfe0fff58bd6e68c40f6f8fb280144e) Thanks [@florian-lefebvre](https://github.com/florian-lefebvre)! - Adds an experimental flag `svgOptimizer` that enables automatic optimization of your SVG components using the provided optimizer. This supersedes the `svgo` experimental flag, which is now removed.

  When enabled, your imported SVG files used as components will be optimized for smaller file sizes and better performance while maintaining visual quality. This can significantly reduce the size of your SVG assets by removing unnecessary metadata, comments, and redundant code.

  Astro ships with a [SVGO](https://svgo.dev/) based optimizer, but any can be used.

  To enable this feature, add the experimental flag in your Astro config and remove `svgo` if it was enabled:

  ```diff
  // astro.config.mjs
  -import { defineConfig } from "astro/config";
  +import { defineConfig, svgoOptimizer } from "astro/config";

  export default defineConfig({
  +  experimental: {
  +    svgOptimizer: svgoOptimizer()
  -    svgo: true
  +  }
  });
  ```

  For more information on enabling and using this feature in your project, see the [experimental SVG optimization docs](https://docs.astro.build/en/reference/experimental-flags/svg-optimization/).
```

**File:** packages/astro/CHANGELOG.md (L2398-2416)
```markdown
- [#15291](https://github.com/withastro/astro/pull/15291) [`89b6cdd`](https://github.com/withastro/astro/commit/89b6cdd4075f5c9362291c386bb1e7c100b467a5) Thanks [@florian-lefebvre](https://github.com/florian-lefebvre)! - Adds a new Fonts API to provide first-party support for adding custom fonts in Astro.

  This feature allows you to use fonts from both your file system and several built-in supported providers (e.g. Google, Fontsource, Bunny) through a unified API. Keep your site performant thanks to sensible defaults and automatic optimizations including preloading and fallback font generation.

  To enable this feature, configure `fonts` with one or more fonts:

  ```js title="astro.config.mjs"
  import { defineConfig, fontProviders } from 'astro/config';

  export default defineConfig({
    fonts: [
      {
        provider: fontProviders.fontsource(),
        name: 'Roboto',
        cssVariable: '--font-roboto',
      },
    ],
  });
  ```
```

**File:** packages/astro/src/core/csp/config.ts (L43-65)
```typescript
const ALLOWED_DIRECTIVES = [
	'base-uri',
	'child-src',
	'connect-src',
	'default-src',
	'fenced-frame-src',
	'font-src',
	'form-action',
	'frame-ancestors',
	'frame-src',
	'img-src',
	'manifest-src',
	'media-src',
	'object-src',
	'referrer',
	'report-to',
	'report-uri',
	'require-trusted-types-for',
	'sandbox',
	'trusted-types',
	'upgrade-insecure-requests',
	'worker-src',
] as const;
```
