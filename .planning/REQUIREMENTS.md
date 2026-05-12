# Requirements: Jo Guest House — Website Rebuild

**Defined:** 2026-05-12
**Core Value:** Turn existing TikTok / Meta / Google Ads paid traffic into actual WhatsApp bookings — converting better than the current half-finished WordPress site.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases. Traceability is below.

### Setup

- [ ] **SETUP-01**: Hosting provider selected (Cloudflare Pages free OR Vercel Pro paid) — Vercel Hobby is a ToS violation for commercial use
- [ ] **SETUP-02**: 3 visual design directions generated via `/design-direction` skill; owner picks one before implementation begins
- [ ] **SETUP-03**: Brand tokens (colors, typography, spacing) finalized from chosen direction and locked in `src/styles/global.css` `@theme` block

### Content

- [ ] **CONT-01**: Complete content inventory in `src/content/site.ts` — every string visible on the page typed and reviewed before any layout phase begins
- [ ] **CONT-02**: All Bahasa Indonesia copy reviewed (currently drafted by builder, ideally checked by native speaker)
- [ ] **CONT-03**: English fallback strings written for: hero subtitle, primary CTAs, OTA badge labels, footer rights
- [ ] **CONT-04**: ~40 existing WordPress photos audited; 12–18 best photos selected for the new site (hero + rooms + facilities + location)
- [ ] **CONT-05**: All selected photos optimized via `astro:assets` (AVIF/WebP, responsive `srcset`, lazy except hero LCP)
- [ ] **CONT-06**: Hero LCP photo chosen and confirmed (`<Image priority>` set, no layout shift)
- [ ] **CONT-07**: Open Graph image (1200×630) baked and placed at `public/og-image.jpg` — Jo Guest House + price + "dekat CGK" overlay
- [ ] **CONT-08**: CI grep on build output blocks Lorem Ipsum, placeholder headings ("Add Your Heading Text Here", "A Title to Turn the Visitor Into a Lead"), and stray `$` USD prices from shipping

### Sections (one scrolling page)

- [ ] **SECT-01**: Hero — property name, slogan "Hotel Nyaman Gak Harus Bintang Lima", "±X menit dari Bandara Soekarno-Hatta", Rp 200.000/malam, primary WhatsApp CTA
- [ ] **SECT-02**: Trust bar / USP strip — Free WiFi, Cafe, Water Heater, Room Service, 24×7 Reception, AC (Indonesian-context appropriate icons)
- [ ] **SECT-03**: About — short property description in Bahasa, "your home sweet home" framing
- [ ] **SECT-04**: Rooms — 17 rooms total, Rp 200.000/malam, room photo gallery, amenity list, WhatsApp CTA
- [ ] **SECT-05**: Facilities grid — same items as trust bar but expanded with short descriptions
- [ ] **SECT-06**: Location — embedded Google Maps iframe (lazy), written address, distance/time to CGK airport, distance/time to JORR
- [ ] **SECT-07**: OTA links — Agoda, Traveloka, Booking.com as text links (NOT logos, per OTA brand guidelines); each opens the property's actual listing
- [ ] **SECT-08**: Testimonials / social proof — 3–6 static testimonials with names + Indonesian phrasing (real or pending owner-supplied)
- [ ] **SECT-09**: FAQ — 4–6 common questions (parking, check-in time, airport shuttle, payment methods, cancellation, breakfast)
- [ ] **SECT-10**: Contact / enquiry form — Web3Forms POST + hCaptcha; success state fires tracking event
- [ ] **SECT-11**: Footer — address, WA number, OTA links, copyright

### WhatsApp Primitive (most-blocking)

- [ ] **WA-01**: `WhatsAppButton.astro` component used in 6+ places (hero, rooms, sticky CTA, contact section, FAQ, footer)
- [ ] **WA-02**: Uses `api.whatsapp.com/send` endpoint (NOT `wa.me`) — works on iOS Safari desktop, Firefox, and TikTok/Instagram in-app browsers
- [ ] **WA-03**: Pre-filled message in Bahasa: "Halo, saya tertarik untuk pesan kamar di Jo Guest House. Sumber: [utm_source]"
- [ ] **WA-04**: UTM parameters from page URL injected into the pre-filled message — owner sees ad source directly in WhatsApp
- [ ] **WA-05**: Sticky bottom mobile CTA bar (WhatsApp + Maps + Call) — visible only on mobile, fades in after first scroll
- [ ] **WA-06**: Every WA click fires conversion event through `lib/track.ts` chokepoint (one call → GTM fans out to TikTok / Meta / Google Ads)
- [ ] **WA-07**: WhatsApp number `+62 851 0800 2536` correct format (no `+`, no spaces, no dashes in the URL)

### Tracking & Cutover Safety

- [ ] **TRACK-01**: GTM container `GTM-MK4WJPMF` installed via inline `<script is:inline>` in `<head>` and noscript fallback in `<body>` (skip Partytown for v1)
- [ ] **TRACK-02**: Existing GTM container audited — current trigger events documented before any changes (owner shares GTM access)
- [ ] **TRACK-03**: TikTok Pixel (`D2MR5GRC77U4PA826B90`), Meta Pixel (`770014465395193`), Google Ads conversion (`AW-17438288457`) all fire through GTM, not directly installed
- [ ] **TRACK-04**: Single `src/lib/track.ts` chokepoint — no component touches `window.dataLayer` directly
- [ ] **TRACK-05**: WhatsApp click event verified firing in: Meta Events Manager Test Events, TikTok Events Manager, Google Tag Assistant
- [ ] **TRACK-06**: Form submission success fires conversion event with the same verification
- [ ] **TRACK-07**: UTM tracking parameters preserved end-to-end (ad → page URL → WA pre-fill → tracking event)

### SEO & Schema

- [ ] **SEO-01**: Real `<title>` per page — replaces "My Blog – My WordPress Blog" with Indonesian-targeted title including "guesthouse dekat Bandara Soekarno-Hatta"
- [ ] **SEO-02**: Meta description in Bahasa, ≤160 chars, includes price + airport + booking framing
- [ ] **SEO-03**: Open Graph + Twitter Card meta tags (title, description, image)
- [ ] **SEO-04**: Schema.org `LodgingBusiness` JSON-LD with `name`, `address`, `geo` (real lat/lng from Google Maps pin), `priceRange`, `image`, `telephone`, `sameAs` (OTA URLs), `amenityFeature`
- [ ] **SEO-05**: JSON-LD passes Google Rich Results Test without errors or warnings
- [ ] **SEO-06**: `sitemap.xml` and `robots.txt` generated; sitemap points to `/` only (single-page)
- [ ] **SEO-07**: Old WordPress URLs (`/sample-page/`, `/?p=N` etc.) handled — 301 redirect to `/` at the hosting layer, or `noindex` + sitemap signal
- [ ] **SEO-08**: Canonical URL set to `https://joguesthouse.my.id/`

### Performance & Mobile

- [ ] **PERF-01**: Lighthouse Mobile ≥ 90 Performance, ≥ 95 Accessibility, ≥ 95 SEO, ≥ 95 Best Practices
- [ ] **PERF-02**: LCP < 2.5s on simulated Moto G4 / Slow 4G in Lighthouse
- [ ] **PERF-03**: CLS < 0.05 — no layout shifts after hero loads
- [ ] **PERF-04**: TBT < 200ms; if tracking pushes it higher, fall back to Partytown for GTM
- [ ] **PERF-05**: Total JS budget < 50 KB on first load (excluding GTM container script)
- [ ] **PERF-06**: Mobile-first responsive — designed at 360px, scales up
- [ ] **PERF-07**: All interactive elements ≥ 44×44px tap targets
- [ ] **PERF-08**: Price displayed as hardcoded `Rp 200.000` string — no `Intl.NumberFormat` polyfill cost

### Form Handling

- [ ] **FORM-01**: Web3Forms POST endpoint configured with owner-provided access key in `.env`
- [ ] **FORM-02**: hCaptcha enabled (Web3Forms built-in) — spam guard
- [ ] **FORM-03**: Indonesian phone-number validation (accept `+62…`, `08…`, `628…` formats)
- [ ] **FORM-04**: Success state shows toast / inline confirmation in Bahasa; no page redirect
- [ ] **FORM-05**: Success fires a tracking event through `lib/track.ts`
- [ ] **FORM-06**: Failure state shows user-friendly error in Bahasa with WA fallback CTA

### Deploy & Cutover

- [ ] **DEPLOY-01**: Deployed to chosen host (Cloudflare Pages default) — production URL accessible
- [ ] **DEPLOY-02**: Preview deploys configured for every PR/branch
- [ ] **DEPLOY-03**: `joguesthouse.my.id` DNS pointed at production after staging verification
- [ ] **DEPLOY-04**: TTL on existing DNS records lowered to 5 min at least 48 h before cutover
- [ ] **DEPLOY-05**: SSL certificate issued and verified on new host
- [ ] **DEPLOY-06**: Pixel verification protocol passed on production URL — all 4 pixels firing for WA clicks and form submits
- [ ] **DEPLOY-07**: Old WordPress instance kept live for 7 days as rollback before decommission
- [ ] **DEPLOY-08**: Post-launch monitoring window (T+0 to T+4h) — owner stays on WhatsApp to confirm bookings continue flowing

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Content Expansion

- **CONT2-01**: Real OTA ratings displayed (after confirming current Agoda/Traveloka/Booking ratings ≥ 4.0)
- **CONT2-02**: Photo gallery with lightbox (if room photos justify it)
- **CONT2-03**: Real guest testimonials with photos and dates
- **CONT2-04**: Bahasa copy reviewed by a native speaker
- **CONT2-05**: Airport shuttle info if owner offers it

### SEO Expansion

- **SEO2-01**: Per-room detail pages (`/rooms/[slug]`) if room types diverge later
- **SEO2-02**: Multilingual full pages with `hreflang` if foreign-tourist traffic grows
- **SEO2-03**: Blog or guide content ("Panduan transit Bandara Soekarno-Hatta") for organic SEO

### Booking & Operations

- **BOOK2-01**: On-site booking form with date picker → email/WA owner
- **BOOK2-02**: Real-time availability widget (requires PMS/channel manager)
- **BOOK2-03**: Direct payment (Midtrans) — requires PMS
- **BOOK2-04**: Owner CMS (Decap CMS or similar) for editing copy without git

### Polish

- **POLISH2-01**: A/B test hero copy variants via GTM
- **POLISH2-02**: Heatmaps / session replay (Microsoft Clarity, free)
- **POLISH2-03**: Real-device performance baseline on Moto G4-class Android in Jakarta

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| On-site booking engine with real-time availability | Owner has no PMS / channel manager to integrate with |
| Payment processing (Midtrans, Stripe, GoPay) | Bookings flow through WA + OTAs; no on-site money flow |
| User accounts / login / loyalty program | Single property, single-night transactional flow — no value |
| Blog / content marketing CMS | Not the leverage point for a 17-room property; defer to v2 |
| OTA logo embedding (Agoda/Booking/Traveloka logos) | Their brand guidelines forbid use without permission; legal risk |
| "Lebih murah dari OTA" / price-undercut copy | OTA rate-parity clauses; contract-termination risk |
| Multi-page site (separate /rooms, /about, /contact) | One-page ships faster; 17 identical rooms don't justify multi-page IA |
| Per-room detail pages | All 17 rooms are same type/price; no information density |
| Full bilingual EN site with `/en` route + language toggle | Owner directive: Bahasa primary, English fallback only |
| New domain (`.com` / `.id`) | Owner directive: keep `joguesthouse.my.id` |
| Patching the existing WordPress site | Rebuild faster than fixing template hell |
| React / Vue / Alpine.js | Vanilla Astro + tiny TS modules sufficient; keep JS budget low |
| Cookie consent banner (full EU-style) | Indonesian PDP law allows non-blocking disclosure; no EU traffic expected |
| OTA review scraping / embedding | API complexity + ToS risk; static testimonials instead for v1 |
| Real-time chat widget (Tawk.to, Crisp, Intercom) | WhatsApp IS the chat — bolting another chat on confuses users |
| Newsletter signup | Wrong leverage for a transient-stay budget property |

## Traceability

Which phases cover which requirements. Populated by gsd-roadmapper on 2026-05-12.

| Requirement | Phase | Status |
|-------------|-------|--------|
| SETUP-01 | Phase 1 | Pending |
| SETUP-02 | Phase 1 | Pending |
| SETUP-03 | Phase 1 | Pending |
| CONT-01 | Phase 2 | Pending |
| CONT-02 | Phase 2 | Pending |
| CONT-03 | Phase 2 | Pending |
| CONT-04 | Phase 2 | Pending |
| CONT-05 | Phase 4 | Pending |
| CONT-06 | Phase 4 | Pending |
| CONT-07 | Phase 2 | Pending |
| CONT-08 | Phase 2 | Pending |
| SECT-01 | Phase 5 | Pending |
| SECT-02 | Phase 5 | Pending |
| SECT-03 | Phase 5 | Pending |
| SECT-04 | Phase 5 | Pending |
| SECT-05 | Phase 5 | Pending |
| SECT-06 | Phase 5 | Pending |
| SECT-07 | Phase 5 | Pending |
| SECT-08 | Phase 5 | Pending |
| SECT-09 | Phase 5 | Pending |
| SECT-10 | Phase 5 | Pending |
| SECT-11 | Phase 5 | Pending |
| WA-01 | Phase 4 | Pending |
| WA-02 | Phase 4 | Pending |
| WA-03 | Phase 4 | Pending |
| WA-04 | Phase 4 | Pending |
| WA-05 | Phase 4 | Pending |
| WA-06 | Phase 4 | Pending |
| WA-07 | Phase 4 | Pending |
| TRACK-01 | Phase 3 | Pending |
| TRACK-02 | Phase 3 | Pending |
| TRACK-03 | Phase 3 | Pending |
| TRACK-04 | Phase 3 | Pending |
| TRACK-05 | Phase 6 | Pending |
| TRACK-06 | Phase 6 | Pending |
| TRACK-07 | Phase 4 | Pending |
| SEO-01 | Phase 3 | Pending |
| SEO-02 | Phase 3 | Pending |
| SEO-03 | Phase 3 | Pending |
| SEO-04 | Phase 5 | Pending |
| SEO-05 | Phase 5 | Pending |
| SEO-06 | Phase 3 | Pending |
| SEO-07 | Phase 5 | Pending |
| SEO-08 | Phase 3 | Pending |
| PERF-01 | Phase 5 | Pending |
| PERF-02 | Phase 5 | Pending |
| PERF-03 | Phase 5 | Pending |
| PERF-04 | Phase 5 | Pending |
| PERF-05 | Phase 3 | Pending |
| PERF-06 | Phase 3 | Pending |
| PERF-07 | Phase 3 | Pending |
| PERF-08 | Phase 3 | Pending |
| FORM-01 | Phase 5 | Pending |
| FORM-02 | Phase 5 | Pending |
| FORM-03 | Phase 5 | Pending |
| FORM-04 | Phase 5 | Pending |
| FORM-05 | Phase 5 | Pending |
| FORM-06 | Phase 5 | Pending |
| DEPLOY-01 | Phase 6 | Pending |
| DEPLOY-02 | Phase 6 | Pending |
| DEPLOY-03 | Phase 6 | Pending |
| DEPLOY-04 | Phase 6 | Pending |
| DEPLOY-05 | Phase 6 | Pending |
| DEPLOY-06 | Phase 6 | Pending |
| DEPLOY-07 | Phase 6 | Pending |
| DEPLOY-08 | Phase 6 | Pending |

**Coverage:**
- v1 requirements: 66 total
- Mapped to phases: 66 ✓
- Unmapped: 0 ✓

**Per-phase requirement counts:**
| Phase | Requirements |
|-------|--------------|
| Phase 1: Setup + Key Decisions | 3 (SETUP-01..03) |
| Phase 2: Content Inventory Gate | 6 (CONT-01..04, CONT-07, CONT-08) |
| Phase 3: Foundation + Tracking Scaffold | 13 (TRACK-01..04, SEO-01..03, SEO-06, SEO-08, PERF-05..08) |
| Phase 4: WhatsApp Primitive + Image Pipeline | 10 (WA-01..07, TRACK-07, CONT-05, CONT-06) |
| Phase 5: Sections + SEO Schema + Form | 24 (SECT-01..11, SEO-04, SEO-05, SEO-07, FORM-01..06, PERF-01..04) |
| Phase 6: Tracking Verification + Cutover | 10 (TRACK-05, TRACK-06, DEPLOY-01..08) |
| **Total** | **66** |

---
*Requirements defined: 2026-05-12*
*Last updated: 2026-05-12 after roadmap creation (traceability populated)*
