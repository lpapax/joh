<!-- GSD:project-start source:PROJECT.md -->
## Project

**Jo Guest House — Website Rebuild**

A new marketing/booking landing site for **Jo Guest House**, a 17-room budget guesthouse in Cengkareng, West Jakarta (near Soekarno-Hatta International Airport, CGK). Replaces the current half-finished WordPress site at `joguesthouse.my.id` with a fast, conversion-focused Astro + Tailwind static site. Primary audience: Indonesian travelers looking for an affordable stay near the airport; secondary: English-speaking visitors who land via OTAs or ads.

**Core Value:** **Turn the existing paid traffic (TikTok / Meta / Google Ads) into actual WhatsApp bookings.** Every other consideration — design, SEO, OTA integration — is in service of this. If the new site doesn't convert better than the current one, it has failed.

### Constraints

- **Tech stack**: Astro + Tailwind v4 + GSAP (light, optional) — matches Pavelec Studio toolchain
- **Timeline**: ASAP — the longer paid traffic hits the current site, the more money burns
- **Budget**: Minimal external services; Vercel free tier, no paid SaaS
- **Domain**: Keep `joguesthouse.my.id` (free `.my.id` zone; site quality must compensate for low-trust TLD)
- **Language**: Bahasa Indonesia primary; English limited to key UI strings
- **Mobile-first**: ~90 % of Indonesian web traffic is mobile — desktop is a courtesy, not primary
- **Tracking parity**: All current ad pixels (TikTok, Meta, GTM, Google Ads conversion) must work on the new site from day one — ad campaigns can't go dark
- **No content backlog**: Owner has not delivered fresh photos, copy, or brand assets ("we don't need it now") — v1 ships using existing WP-site assets + placeholders where needed; refresh in a later iteration
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Executive Summary
## Recommended Stack
### Core Technologies
| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| **Astro** | `^6.3.1` | Static-site framework with islands architecture | Zero-JS by default → ideal for a landing page where the JS budget should be ~0 KB before pixels. Mature `astro:assets` pipeline gives best-in-class responsive images. Matches the established Pavelec Studio toolchain. Confidence: **HIGH** |
| **Tailwind CSS** | `^4.3.0` | Utility-first CSS | v4 is CSS-first (no `tailwind.config.js`), 5× faster builds, smaller runtime, native CSS variables. Installed via the official `@tailwindcss/vite` plugin (Astro 5.2+ uses this; the old `@astrojs/tailwind` integration is deprecated). Confidence: **HIGH** |
| **@tailwindcss/vite** | `^4.3.0` | Tailwind Vite plugin | Replaces the deprecated `@astrojs/tailwind` integration. Installed automatically by `npx astro add tailwind`. Confidence: **HIGH** |
| **@astrojs/vercel** | `^10.0.6` | Vercel adapter | Needed if any server feature is used; for a pure static site you can also skip the adapter and rely on Vercel's static build (`output: 'static'`). Recommend installing the adapter so Vercel Analytics + Image Optimization work natively. Confidence: **HIGH** |
| **Sharp** | `^0.34.5` | Image processor (built into `astro:assets`) | Bundled transitively — no manual install needed. Generates AVIF/WebP at build time. Confidence: **HIGH** |
### Supporting Libraries
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| **@astrojs/sitemap** | `^3.7.2` | Generates `sitemap-index.xml` | Always — Google Search Console wants it. One line in `astro.config.mjs`. |
| **astro-seo** | `^0.8.x` (or roll your own `<SEO>` component) | OG tags, Twitter cards, canonical | Recommend a **hand-rolled `<SEO.astro>` component** (~30 lines) over the library — gives precise control of OG image, JSON-LD slot, and Bahasa-friendly meta. The library hasn't shipped in a while. |
| **@vercel/analytics** | `^2.0.1` | Privacy-friendly traffic analytics | Free tier: 2,500 events/month — fine for a single-page guesthouse site getting paid clicks. Drop-in via `<Analytics />` script. |
| **astro-icon** | `^1.1.5` | Compile-time SVG icons | For OTA logos (Agoda/Traveloka/Booking) and amenity icons. Inlines SVGs so no extra network requests. Pairs with `iconify-json` collections. |
| **GSAP** | `^3.13.x` | Optional light animation | Only for hero reveal / scroll micro-interactions. Use the free CDN build, no ScrollSmoother (paid). Keep total budget < 30 KB. |
| **@astrojs/partytown** | `^2.1.7` | Move 3rd-party scripts to web worker | **Conditional install** — only if Lighthouse TBT > 200ms after pixel install. Default: skip it. See "Tracking Strategy" below. |
### Development Tools
| Tool | Purpose | Notes |
|------|---------|-------|
| **@astrojs/check** | `^0.9.x` + `typescript@^5.x` | Type-check `.astro` files | Run in CI via `astro check`. Strict mode in `tsconfig.json`. |
| **Prettier + `prettier-plugin-astro`** | Formatter | Pre-commit hook keeps diffs clean. |
| **Vercel CLI** | `vercel` | Local preview parity with prod | `vercel dev` only when debugging adapter-specific behavior; otherwise `astro dev` is faster. |
| **Lighthouse CI / WebPageTest** | Performance regression | Run on every PR before going live. Target Mobile LCP < 2.5s on Slow 4G (Indonesian mobile reality). |
## Installation
# 1. Scaffold
# 2. Tailwind v4 (creates src/styles/global.css with @import "tailwindcss")
# 3. Vercel adapter (static output by default; needed for @vercel/analytics)
# 4. Sitemap
# 5. Icons + Vercel analytics + (optional) GSAP
# 6. Partytown — DO NOT INSTALL UNLESS NEEDED (see Tracking Strategy)
# npx astro add partytown
## Detailed Decisions (per question)
### 1. Astro version + integrations
- **Astro `^6.3.1`** (latest stable, verified via `npm view astro version`, 2026-05-12). Confidence: **HIGH**
- **Integrations to install:** `@astrojs/sitemap`, `@astrojs/vercel`, `astro-icon`. That's it.
- **Skip `@astrojs/robotstxt`** — write a 4-line `public/robots.txt` by hand.
- **Skip `@astrojs/partytown` by default** — see Tracking Strategy.
### 2. Tailwind v4 setup (CSS-first)
### 3. Fonts (Bahasa + English, mobile)
- Designed by Indonesian foundry Tokotype, commissioned for the Jakarta city brand. The name alone has soft local credibility for a Jakarta guesthouse. Confidence (cultural fit): **HIGH**
- Bahasa Indonesia uses the basic Latin alphabet with **no diacritics** in standard orthography (unlike Vietnamese or Czech). The default `latin` subset is sufficient — no need for `latin-ext`. This is a real mobile bandwidth win.
- Two weights (400 body, 700 display). Skip variable font for v1 — a single static weight is ~25 KB woff2 vs ~80 KB variable. With only two weights the static files are smaller in total.
- **Self-host via `@fontsource/plus-jakarta-sans`** instead of Google Fonts CDN. Reasons: (a) no third-party request, better LCP; (b) `fonts.googleapis.com` is occasionally flaky on Indonesian mobile networks; (c) no GDPR concern (irrelevant here, but bonus).
- Add `font-display: swap` and `<link rel="preload" as="font" type="font/woff2" crossorigin>` for the 400 weight only.
### 4. Image strategy
- Move the photos from the WordPress media library into `src/assets/photos/` so Astro processes them (build-time AVIF + WebP + responsive `srcset`).
- For the hero photo: `<Image src={...} alt="..." width={1600} height={900} loading="eager" fetchpriority="high" />` — force eager + high priority for LCP. All other photos: default lazy.
- For gallery / room cards: `<Picture src={...} formats={['avif', 'webp']} widths={[400, 800, 1200]} sizes="(max-width: 768px) 100vw, 33vw" />`.
- Set `image.layout: 'constrained'` globally so every image gets responsive `srcset`/`sizes` automatically.
- **Migration of WP photos:** download via WP's media-library export or scrape `/wp-content/uploads/` directly with `wget --recursive --accept jpg,jpeg,png,webp`. Run a one-time pre-processing step (`sharp` CLI or `squoosh`) to crop to consistent aspect ratios (16:9 for hero, 4:3 for rooms) before committing to `src/assets/`. This is a one-day chore.
### 5. Marketing pixel installation — RECOMMENDED PATTERN
### 6. WhatsApp click-to-chat
- Number format: **no `+`, no spaces, no dashes, no leading zeros** — `6285108002536` (62 = Indonesia country code).
- Pre-filled message in Bahasa: `Halo, saya tertarik dengan Jo Guest House. Apakah masih ada kamar tersedia?`
- One Astro component `<WhatsAppButton />` accepts a `messageKey` prop so each section can prefill context (e.g., the rooms section sends `"...tertarik dengan kamar Rp 200.000..."`).
- **Sticky mobile CTA pattern:** fixed bottom bar on mobile (`md:hidden`), 56px tall, full-width green button. On desktop, a floating right-bottom 64px circular FAB. Both fire a `dataLayer.push({event: 'whatsapp_click', section: '...'})` on tap so GTM can attribute to source.
- Use `rel="noopener"` and `target="_blank"` so the chat opens in WA app (mobile) or web.whatsapp.com (desktop). No `noreferrer` — we want the referrer for WA's own analytics.
### 7. Schema.org JSON-LD
- `Hotel` implies full-service / star-rated. A 17-room budget guesthouse without a star rating doesn't fit Google's mental model.
- `BedAndBreakfast` implies breakfast included and host-hosted, usually smaller. Jo Guest House has a separate "cafe" but isn't a B&B.
- `LodgingBusiness` is the parent type, accepted by Google for rich results when `Hotel`/`BedAndBreakfast` don't fit. Indonesian SERPs render it identically.
### 8. Map embed
| Option | Pros | Cons | Verdict |
|--------|------|------|---------|
| **Google Maps iframe** (recommended) | Free, trust signal (Indonesians know the UI), interactive (pan/zoom), one tag | ~700 KB transferred after lazy-trigger | **Pick this** — it's below the fold and lazy-loaded |
| Mapbox GL | Custom styling | Requires API key + JS bundle (~200 KB) + paid tier | Overkill for a single pin |
| Static Maps API image | Fastest (single PNG, ~50 KB) | Requires API key + billing account, no interactivity, no "open in Maps" button | Backup plan only |
| Leaflet + OSM tiles | Free, no API key | OSM tiles for Jakarta are accurate but lower trust signal than Google | Skip — Indonesian users expect to see Google Maps |
### 9. Hosting / deploy
- Static Astro output ships as plain HTML/CSS/images to Vercel's edge CDN. No Functions invocations consumed.
- Free tier limits that apply: 100 GB bandwidth/month (way above 17-room landing site needs), unlimited static deploys, custom domain free.
- Configure DNS: point `joguesthouse.my.id` to Vercel via CNAME on the apex (use ALIAS/ANAME if registrar supports it, or A records to Vercel's IPs).
- **ISR not needed** — content is hand-edited and rebuilt on push. Set up Vercel's GitHub integration so every push to `main` redeploys.
- Set the `Cache-Control` headers in `vercel.json` for `/_astro/*` assets to `public, max-age=31536000, immutable` (Astro fingerprints filenames, so this is safe).
### 10. Analytics
- Vercel Web Analytics: 2,500 events/month free, one `<Analytics />` script tag, zero config when adapter is installed, privacy-friendly (no cookies). Sufficient for a 17-room guesthouse — even with paid traffic, expect 1–3k sessions/month at v1.
- **Skip Plausible** ($9/mo) — overkill for budget constraint.
- **Skip Umami** — self-hosting overhead (Postgres + Vercel KV or Railway) violates "no paid SaaS, minimal external services" constraint.
- GTM already captures detailed marketing funnel data (pixel conversions). Vercel Analytics covers organic traffic / direct visitors.
### 11. Form handling
| Option | Free tier | Setup | Verdict |
|--------|-----------|-------|---------|
| **Web3Forms** | Unlimited submissions, hCaptcha included | One POST to `https://api.web3forms.com/submit` with an access key in a hidden field | **Pick this** |
| Formspree | 50 submissions/month, then paid | Same pattern, plus reCAPTCHA | Free tier too tight |
| Resend API | 100 emails/day free | Requires a server function (Astro Action + Vercel Function) — adds complexity | Save for when we need transactional email |
| Pure WA deep-link only | N/A | Easiest | Loses the small % of users who prefer a form |
### 12. i18n
- Site is a single page with ~30 strings total.
- Audience: 90% Bahasa speakers via ads, 10% English via OTAs (and English-fluent Indonesians).
- Building two routes (`/` + `/en/`) doubles SEO surface for marginal gain, fragments paid-ad landing-page URLs, and complicates the WhatsApp deep-link UTM tagging.
- Owner directive: "Bahasa primary, English limited to key UI strings."
## Alternatives Considered
| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Astro 6 static | Next.js 15 / Remix | If a real on-site booking engine is added later (server actions, DB, payment). For a static landing page it's massive over-tooling. |
| Tailwind v4 via Vite plugin | Tailwind v3 + `@astrojs/tailwind` integration | Don't — v3's integration is deprecated and v4 builds are 5–10× faster. Only stick with v3 if a critical plugin hasn't migrated (none relevant here). |
| `astro:assets` `<Image />` | `unpic-img`, `astro-imagetools` | Only if you need on-demand CDN transforms (e.g., 1000s of user-uploaded photos). 40 hand-curated photos = native pipeline wins. |
| Inline GTM | Partytown-wrapped GTM | Only as Lighthouse-driven fallback if TBT > 200ms post-launch. |
| Google Maps iframe | Static map image / Mapbox | Static image if owner removes interactivity requirement; Mapbox only if custom styling becomes a brand requirement. |
| Web3Forms | Astro Actions + Resend | When you want full server control (custom validation, auto-reply with HTML email, lead-routing logic). Not needed for v1. |
| Vercel Analytics | Plausible self-hosted | If event count exceeds 2,500/month (will only happen if paid spend ramps significantly). Cost: $9/mo cloud or ~$5/mo Railway self-host. |
| Plus Jakarta Sans (self-hosted) | Inter + system fallback | If brand later wants a more international feel. Plus Jakarta has the local-brand advantage. |
| No i18n routing | Astro built-in i18n with `/en/` prefix | If analytics later shows >25% English-speaking organic traffic, then split into two routes for separate SEO. |
## What NOT to Use
| Avoid | Why | Use Instead |
|-------|-----|-------------|
| **WordPress (patching existing site)** | Half-finished, locked into Elementor + Astra, slow, owner has no editorial workflow. Patching wastes more time than rebuilding. | Astro static rebuild. |
| **`@astrojs/tailwind` integration** | Deprecated. Tailwind v4 uses the Vite plugin directly. | `@tailwindcss/vite` (installed by `astro add tailwind`). |
| **`tailwind.config.js`** | Tailwind v4 is CSS-first. Config in `@theme {}` block in CSS. | `@theme` directive in `global.css`. |
| **Google Fonts CDN `<link>` tag** | Third-party request, indeterminate cache behavior on Indonesian mobile networks, no preload control. | Self-host via `@fontsource/plus-jakarta-sans`. |
| **Direct `<img src=...>` for photos** | No optimization, no AVIF/WebP, no responsive `srcset`, blocking LCP. | `<Image />` from `astro:assets`. |
| **Per-pixel manual install of TikTok + Meta + Google Ads** | Three more script tags, three more places to break, owner can't edit. | One GTM container; manage tags inside GTM UI. |
| **Partytown by default** | Adds COEP/COOP header complexity, service worker, known flakiness with TikTok Pixel; only ~single-digit TBT benefit for 4 pixels on a 1-page site. | Inline GTM. Add Partytown only if Lighthouse demands it. |
| **WPForms / any WordPress dependency** | The whole point is leaving WP. | Web3Forms (free, no backend). |
| **A real booking engine (Cloudbeds / Mews / SiteMinder)** | Out of scope per PROJECT.md. Owner books via WA + OTAs. | `wa.me` deep-link + OTA outbound badges. |
| **JavaScript-heavy carousel libraries (Swiper, Embla with autoplay JS)** | Each adds 20–60 KB and JS execution. For 40 photos use CSS scroll-snap. | Native CSS `scroll-snap-type: x mandatory;` with `<Image />`s. GSAP only for hero entrance. |
| **Multi-page architecture (separate /rooms, /about)** | Explicitly out of scope. One page ships faster. | Anchor links to sections (`#kamar`, `#fasilitas`, `#lokasi`). |
| **Language toggle UI / `/en/` route** | Audience split doesn't justify it; complicates ad-landing-URL strategy. | Bahasa primary, three English strings inline. |
| **Cookie banner / CMP** | Indonesia has no GDPR-equivalent that requires CMPs for this scale. Adds friction. | Skip. (Revisit if EU traffic > 5%.) |
| **Vercel Edge Middleware / ISR / server actions for v1** | Static output is sufficient. Don't pay complexity tax for nothing. | `output: 'static'`. |
## Stack Patterns by Variant
- Move to Plausible Cloud ($9/mo for 10k visitors) or self-host Umami on Railway.
- Reason: Vercel Analytics caps hard; Plausible's UI is cleaner for marketing reports the owner will read.
- Add `@astrojs/partytown` with `forward: ['dataLayer.push', 'fbq', 'ttq', 'gtag']`.
- Change GTM script to `type="text/partytown"`.
- Add COEP/COOP headers in `vercel.json`.
- Re-run Lighthouse. If TikTok Pixel breaks, revert (this is a known issue, MEDIUM confidence).
- Move from JSON-LD `LodgingBusiness` to `Hotel` + `HotelRoom` + `Offer` nested structure.
- Add a `src/data/rooms.json` file. Map over it for both the rooms section and the JSON-LD.
- Still no CMS — git is the source of truth.
- Add `i18n: { locales: ['id', 'en'], defaultLocale: 'id', routing: { prefixDefaultLocale: false } }`.
- Mirror `src/pages/index.astro` to `src/pages/en/index.astro`.
- Add `hreflang` tags.
## Version Compatibility
| Package A | Compatible With | Notes |
|-----------|-----------------|-------|
| `astro@^6.x` | `@tailwindcss/vite@^4.x` | Native via `astro add tailwind` in Astro ≥5.2. Confirmed in `/withastro/docs`. |
| `astro@^6.x` | `@astrojs/vercel@^10.x` | Major version bumped for Astro 5/6. The `@astrojs/vercel/serverless` and `@astrojs/vercel/static` sub-paths were unified into a single export — use plain `import vercel from '@astrojs/vercel'`. |
| `astro@^6.x` | `@astrojs/sitemap@^3.7.x` | Standard. |
| `astro@^6.x` | `@astrojs/partytown@^2.1.x` | Compatible but optional. |
| `tailwindcss@^4.x` | Browsers | v4 drops IE11/legacy Safari support; targets evergreen browsers + Safari ≥15.4. Fine for Indonesian mobile (Chrome Android dominant). |
| `sharp@^0.34.x` | Node.js ≥18.17 | Vercel build images use Node 20 by default. |
| `@fontsource/plus-jakarta-sans` | Any bundler | Pure CSS imports, no version conflicts. |
## Sources
- Context7 `/withastro/docs` — Astro 6 install, Tailwind v4 integration via Vite plugin, `<Image />` / `<Picture />` `astro:assets` API, responsive `layout` config, i18n `prefixDefaultLocale`. **HIGH** confidence.
- Context7 `/qwikdev/partytown` — verified Partytown forwarding patterns for `dataLayer.push` and `fbq`; standard service-forward configs from `@qwik.dev/partytown/services`. **HIGH** confidence on the pattern, **MEDIUM** confidence on its applicability here (we recommend skipping by default).
- `npm view` (live, 2026-05-12) — current versions: `astro@6.3.1`, `tailwindcss@4.3.0`, `@tailwindcss/vite@4.3.0`, `@astrojs/vercel@10.0.6`, `@astrojs/sitemap@3.7.2`, `@astrojs/partytown@2.1.7`, `sharp@0.34.5`, `@vercel/analytics@2.0.1`, `astro-icon@1.1.5`, `@astrojs/check@0.9.9`. **HIGH** confidence.
- [https://schema.org/LodgingBusiness](https://schema.org/LodgingBusiness), [https://schema.org/BedAndBreakfast](https://schema.org/BedAndBreakfast) — type hierarchy and properties. **HIGH** confidence.
- [Google Search Central — LocalBusiness structured data](https://developers.google.com/search/docs/appearance/structured-data/local-business) — required vs recommended properties. **HIGH** confidence.
- [BusinessChat — wa.me URL format](https://help.businesschat.io/en/articles/6517838-how-to-build-a-whatsapp-click-to-chat-url-wa-me) — phone number formatting (no `+`/spaces) and URL-encoded text. **HIGH** confidence.
- [launchfa.st — Using Partytown with GTM in Astro](https://www.launchfa.st/blog/astro-gtm-partytown), [DEV — Partytown + GTM in Astro](https://dev.to/reeshee/using-partytown-with-google-tag-manager-in-astro-a-step-by-step-guide-4b48), [fatbobman.com — forwarding GTM events from Partytown](https://fatbobman.com/en/snippet/how-to-forward-custom-tag-events-to-gtm-running-inside-partytown/) — community-reported friction with multi-pixel Partytown setups. **MEDIUM** confidence (multiple sources agree on the pain points, but no official Astro statement).
- [Web3Forms Astro docs](https://docs.web3forms.com/how-to-guides/static-site-generators/astro), [Web3Forms vs Formspree comparison](https://web3forms.com/alternatives/formspree-alternative) — free-tier and integration details. **HIGH** confidence on free tier; **MEDIUM** on bias (vendor source).
- [Google Fonts — Plus Jakarta Sans](https://fonts.google.com/specimen/Plus+Jakarta+Sans), [tokotype/PlusJakartaSans on GitHub](https://github.com/tokotype/PlusJakartaSans) — font origin, weight options, Latin subset coverage. **HIGH** confidence.
- [Vercel Analytics pricing / Plausible / Umami comparisons](https://www.pkgpulse.com/blog/vercel-analytics-vs-plausible-vs-umami-privacy-first-2026), [Swetrix Umami vs Vercel](https://swetrix.com/comparison/umami/vs-vercel-web-analytics) — free tier limits (2,500 events/mo Vercel; Plausible $9/mo cloud). **MEDIUM** confidence (third-party comparison sites, but cross-corroborated).
- [Google Maps Embed API overview](https://developers.google.com/maps/documentation/embed/get-started) — `loading="lazy"` support, sizing recommendations. **HIGH** confidence.
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
