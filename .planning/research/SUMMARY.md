# Project Research Summary

**Project:** Jo Guest House - Website Rebuild
**Domain:** Mobile-first, WhatsApp-first single-page hospitality landing site (Indonesia)
**Researched:** 2026-05-12
**Confidence:** HIGH (stack, architecture, pitfalls) / MEDIUM (feature differentiator weighting)

---

## Executive Summary

Jo Guest House needs a fast, conversion-focused single-page Astro + Tailwind static site to replace a broken WordPress install burning paid ad spend (TikTok / Meta / Google Ads) against a page with Lorem Ipsum copy, USD prices, and no tracking events. The rebuild is not a design project - it is a conversion recovery project. Every technical decision should be evaluated against one metric: does a visitor arriving from a TikTok ad tap the WhatsApp CTA? The entire business case collapses if that event does not fire reliably and attribute correctly.

The recommended build is Astro 6 static output with Tailwind v4 (CSS-first via @tailwindcss/vite), deployed to **Cloudflare Pages** (not Vercel Hobby - commercial use violation, see Key Decision below), with all four ad pixels routed through the existing GTM container GTM-MK4WJPMF via a single dataLayer chokepoint in src/lib/track.ts. WhatsApp deep-links are built by a single lib/wa.ts function that appends UTM source tokens from the landing URL, giving the owner manual attribution without a PMS. The architecture is flat: 11 section components, 7 UI primitives, 5 vanilla TS interaction scripts - no framework runtime, total first-party JS budget under 10 KB.

The three risks that most often kill a project of this type are: (1) ad pixels silently mis-installed on cutover, destroying bidding-algorithm learning during the 7-14 day re-training window; (2) owner not delivering content causing Lorem Ipsum to ship on the new site just as it shipped on the old one; and (3) no rollback path if the DNS swap surfaces a bug in the critical 4-hour window. All three are process problems, not code problems. The roadmap must encode content-inventory gating, a mandatory pre-cutover pixel-verification protocol, and a TTL-lowering step 48 hours before DNS flip.

---

## Key Findings

### Recommended Stack

Astro 6.3.1 static output is the clear choice: zero JS by default, best-in-class astro:assets image pipeline (AVIF + WebP + srcset at build time via Sharp), and matches the established Pavelec Studio toolchain. Tailwind v4 installs via @tailwindcss/vite - the old @astrojs/tailwind integration is deprecated and must not be used. All four ad pixels route through the existing GTM container via one inline script tag - no Partytown by default, as it introduces COEP/COOP header complexity and known flakiness with TikTok Pixel. The font is Plus Jakarta Sans self-hosted via Fontsource (two static weights, Latin subset, ~50 KB total) - not Google Fonts CDN, which is intermittently slow on Indonesian mobile networks.

**KEY DECISION - Hosting (contradiction resolved):** STACK.md recommended Vercel Hobby (free). PITFALLS.md identified this as a ToS violation: Vercel Hobby explicitly prohibits commercial use, and a guesthouse site with active paid advertising is commercial. **The new default is Cloudflare Pages** (free, explicitly commercial-OK, unlimited bandwidth, Singapore POP for Indonesian latency). Vercel Pro (USD 20/mo) is the paid alternative if the owner prefers Vercel DX. This decision must be logged in PROJECT.md before Phase 1.

**Core technologies:**
- **Astro 6.3.1** static output - zero-JS landing page; astro:assets handles all 40 photos at build time
- **Tailwind v4 + @tailwindcss/vite** - CSS-first tokens in @theme block; @theme intentionally empty until design direction picks colors
- **Cloudflare Pages** - free, commercial-OK, unlimited bandwidth; replaces Vercel Hobby
- **GTM GTM-MK4WJPMF** - single script tag; all four pixels managed in GTM UI, not in code
- **Plus Jakarta Sans via @fontsource** - self-hosted, Latin subset, Indonesian foundry (Tokotype)
- **Web3Forms** - zero-backend contact form, unlimited free submissions, hCaptcha required
- **wa.me deep-links built by lib/wa.ts** - UTM passthrough for manual attribution, fallback href for no-JS

### Expected Features

The MVP ship list (v1) is well-defined. 18 table-stakes features (TS-01 through TS-18) are non-negotiable for launch; 6 differentiators (DF-01, DF-02, DF-03, DF-11, DF-12, DF-16) are also required in v1 because they carry direct revenue attribution value. Everything else is v1.x post-validation.

**Must have (table stakes - v1):**
- TS-01 Hero with name, slogan, Rp 200.000/malam, airport distance, WA CTA above fold at 360px viewport
- TS-02/03 Sticky mobile bottom CTA bar + WA click-to-chat with pre-filled Bahasa message
- TS-05/06/07/08 Rooms, facilities, location map, and photo gallery (curated to 12-18 photos)
- TS-09 OTA badges (Agoda, Traveloka, Booking.com) as outbound text links - trust signal, not integration
- TS-11/12 Real title, meta description, OG image, LodgingBusiness JSON-LD
- TS-13 All four tracking pixels firing from day one - this is the entire business case for the rebuild
- TS-16/17 Rp 200.000 (dot-thousands, Indonesian format) in Bahasa primary copy

**Should have (v1 differentiators):**
- DF-01 Airport proximity as hero badge (+-20 menit ke Bandara Soekarno-Hatta with traffic caveat)
- DF-02/03 Multiple WA entry points with intent-specific pre-fills (book / availability / directions)
- DF-11 whatsapp_click conversion event firing to GTM on every CTA tap
- DF-12 UTM source token appended to WA pre-fill - closes attribution loop manually
- DF-16 OG image optimized for WhatsApp share previews (1200x630, property name + price overlay)

**Defer to v1.x (after validation):**
- DF-05 Static testimonials - blocked on owner providing real quotes; empty section beats fake testimonials
- DF-06 OTA rating callout - blocked on owner confirming current ratings are >=4.0
- DF-09/10 Guest photos + FAQ section - content-gated; owner must provide before adding
- DF-15 GSAP scroll reveals - polish pass after content is stabilized

**Do not build (anti-features):**
- Real-time availability calendar, language toggle UI, per-room pages, live chat widgets, cookie banner, carousel JS libraries, /en/ route for v1

### Architecture Approach

The site is a single src/pages/index.astro composing 11 section components in scroll order, each reading typed data from src/content/*.ts modules (not Astro Content Collections, which are for blog workloads). Seven UI primitives (WhatsAppButton, StickyWaCta, RoomCard, FacilityItem, OtaBadge, Container, SectionHeading) are shared across sections. Five vanilla TS scripts handle all interactivity - no framework runtime. The tracking architecture funnels every pixel-relevant event through one file (src/lib/track.ts -> window.dataLayer.push) so pixel routing is a one-file change forever.

**Major components and build-order dependencies:**
1. **Foundation** (astro.config.mjs, BaseLayout.astro, global.css, src/content/site.ts) - blocks everything; must ship first
2. **Tracking scaffold** (GtmHead/Body.astro, lib/track.ts) - must exist before any CTA is built
3. **WhatsApp primitive** (WhatsAppButton.astro, lib/wa.ts, lib/utm.ts, scripts/wa-cta.ts) - single most-blocking primitive; pulled into ~6 sections
4. **SEO layer** (SeoHead.astro, JsonLd.astro, lib/schema.ts) - launch gate; not a blocker for section dev
5. **Image pipeline** (migrate WP photos to src/assets/photos/, curate to 12-18, pre-process) - blocks Hero, Rooms, About
6. **Sections** (11 components, mostly parallelizable after steps 1-5)
7. **Cutover** (pixel re-verification, DNS swap, redirects, Search Console)

### Critical Pitfalls

1. **Vercel Hobby = ToS violation for commercial site** - switch to Cloudflare Pages before any DNS work; log the decision in PROJECT.md now. If Vercel Pro chosen instead, owner must own the Vercel account (not builder).

2. **Pixel IDs silently wrong on cutover** - audit the existing GTM container before touching new code (screenshot every tag, trigger, variable); all pixel IDs in one src/content/site.ts constant; mandatory GTM Preview Mode + Meta Pixel Helper + TikTok Helper + Google Tag Assistant verification on staging before DNS flip; never mark tracking done until Events Manager shows test events server-side.

3. **Owner content never arrives - Lorem Ipsum ships again** - create .planning/content-inventory.md before any layout work; block ship with CI grep dist/ check for placeholder strings; honest interim copy beats Lorem Ipsum; owner signs off on every visible string before cutover.

4. **No DNS rollback path** - lower DNS TTL to 300s (5 min) 48h before cutover; document old WordPress IP/nameservers; keep WP running 14 days post-cutover; cut over at 02:00-05:00 WIB Tuesday; define explicit rollback criteria before flip.

5. **GTM + 3 pixels tank TBT on cheap Android** - defer GTM load via requestIdleCallback/2500ms timeout (dataLayer queue ensures events are not missed); verify on real device (Redmi 9 / Samsung A14) with WebPageTest from Jakarta; Partytown only as Lighthouse-driven last resort.

---


## Implications for Roadmap

Based on combined research, the build dependency graph and pitfall phase assignments converge on 6 phases. The ordering is forced by hard dependencies: tracking must precede CTAs; content must precede layout; design direction must precede brand tokens; pixels must be verified before cutover.

### Phase 0: Project Setup + Key Decisions

**Rationale:** Two decisions must be logged before any code is written: (1) hosting target (Cloudflare Pages vs Vercel Pro - Vercel Hobby is off the table), and (2) design direction (owner chooses one of three generated directions before @theme color tokens are defined). The hosting choice affects astro.config.mjs adapter; the design direction blocks brand tokens which blocks layout implementation.
**Delivers:** Updated PROJECT.md Key Decisions table, confirmed Cloudflare Pages account under owner name, one chosen design direction with color palette and typography scale.
**Addresses:** Pitfall 1 (Vercel ToS), Architecture TODO markers in global.css @theme block.
**Gate:** Owner has chosen design direction AND confirmed hosting. No code written until both are resolved.

### Phase 1: Content Inventory Gate

**Rationale:** PITFALLS.md identifies owner content not arriving as the most likely failure mode - the same failure as the current WP site. Preceding layout work with a mandatory content audit prevents the Lorem Ipsum repeat. This phase is process, not code. It also resolves the distance/time claim to CGK (Pitfall 10), the OTA rating figures for DF-06, and exact lat/long coordinates for JSON-LD (Pitfall 8).
**Delivers:** .planning/content-inventory.md with owner-signed-off: hero strings (exact Bahasa), price string (Rp 200.000), verified airport distance range with traffic caveat, each amenity label, FAQ answers if owner provides, 12-18 curated photos selected from 40 WP photos, OTA listing URLs confirmed live.
**Addresses:** Pitfall 7 (Lorem Ipsum repeat), Pitfall 9 (price format), Pitfall 10 (false distance claim).
**Gate:** Owner ticks every string in content inventory. No layout phase until gate passes.
**Research flag:** Process-only phase. Owner is the source of truth. No library research needed.

### Phase 2: Foundation + Tracking Scaffold

**Rationale:** Architecture build-order dependency graph makes this the unblocking work. BaseLayout.astro, global.css with brand tokens (from Phase 0 design direction), and lib/track.ts must exist before any section or CTA can be built. GTM integration goes here - and this is where the pre-cutover GTM container audit happens (screenshot existing tags/triggers before touching anything).
**Delivers:** Working Astro project on Cloudflare Pages staging URL, Tailwind v4 with design-direction-sourced @theme tokens, Plus Jakarta Sans self-hosted, BaseLayout + SeoHead + GtmHead/Body wired up, lib/track.ts with trackWaClick / trackOtaClick / trackEnquirySubmit stubs, src/content/site.ts with full shape defined.
**Stack:** Astro 6.3.1 + Tailwind v4 + @tailwindcss/vite + Cloudflare Pages deploy + @fontsource/plus-jakarta-sans
**Avoids:** Pitfall 2 (pixel IDs wrong) by establishing single site.ts source of truth; Pitfall 16 (Google Fonts CDN) by self-hosting at project init.
**Research flag:** Standard patterns - no additional research needed.

### Phase 3: WhatsApp Primitive + Image Pipeline

**Rationale:** WhatsAppButton.astro + lib/wa.ts + lib/utm.ts + scripts/wa-cta.ts is the single most-blocking primitive in the architecture - used in ~6 sections (Hero, StickyWaCta, Rooms, Enquiry, Footer, Location). It must be built, tested on the full device matrix (Android Chrome, iOS Safari, Instagram in-app browser, macOS Safari desktop), and verified for UTM passthrough before any section that references it can ship. Image pipeline runs in parallel and unblocks Hero, Rooms, and About sections.
**Delivers:** WhatsAppButton.astro with fallback href (server-side, no-JS safe) + client enrichment with UTMs + trackWaClick call; three pre-fill templates (hero/booking, rooms/availability, location/directions); lib/wa.ts with buildWaUrl(messageKey, utms) builder; device-matrix test documented; src/assets/photos/ with curated and pre-processed photos; public/og-image.jpg at 1200x630.
**Avoids:** Pitfall 5 (iOS Safari pre-fill breakage), Pitfall 12 (image overload).
**Implements:** DF-02, DF-03, DF-12.
**Research flag:** Cross-browser WA testing requires real device verification - QA milestone within phase.

### Phase 4: Sections + SEO + Schema

**Rationale:** With tracking scaffold, WA primitive, and image pipeline in place, all 11 sections are mostly parallelizable. This is the bulk layout work. SEO layer (SeoHead, JsonLd, lib/schema.ts) ships as a launch gate. JSON-LD must use exact coordinates (from content inventory), absolute URLs, and valid priceRange format; validate with Google Rich Results Test before signing off.
**Delivers:** All 11 section components, StickyWaCta, OtaBadge click tracking, Faq.astro with FAQPage JSON-LD, Enquiry.astro with Web3Forms + hCaptcha + fallback WA link, LodgingBusiness JSON-LD validated in Rich Results Test, sitemap, robots.txt, canonical, OG tags.
**Implements:** Full architecture component inventory (22 files total).
**Avoids:** Pitfall 6 (OTA logo trademark - use text-only links unless written permission confirmed), Pitfall 8 (invalid JSON-LD), Pitfall 15 (Web3Forms spam without hCaptcha).
**Research flag:** Standard patterns - ARCHITECTURE.md provides complete component-level spec.

### Phase 5: Tracking Verification + Cutover Safety

**Rationale:** This is where the three cross-cutting findings converge into one phase. Architecture src/lib/track.ts single chokepoint + PITFALLS pre-cutover GTM audit + FEATURES DF-11/DF-12 WA-tap-as-conversion requirement all land here. This phase runs on the staging URL before DNS changes. DNS TTL is lowered to 300s in this phase (48h before cutover).
**Delivers:** Completed .planning/cutover-checklist.md with signed-off verifications: GTM Preview Mode screenshotted; Meta Pixel Helper confirms PageView + Contact events with pixel ID 770014465395193; TikTok Pixel Helper confirms events with ID D2MR5GRC77U4PA826B90; Google Tag Assistant confirms Google Ads conversion AW-17438288457 fires; all pixel IDs cross-checked against site.ts constants; CI grep dist/ placeholder check passes; Rich Results Test passes; Lighthouse Mobile LCP <2.5s + TBT <200ms; Web3Forms delivers 3 test submissions to owner inbox; DNS TTL lowered to 300s; old WP origin IP documented; _redirects file built for known WP paths; rollback criteria defined.
**Avoids:** Pitfall 2 (pixel mis-install), Pitfall 3 (no rollback), Pitfall 4 (orphan WP URLs).
**Research flag:** Real-device performance testing (WebPageTest Jakarta, Moto G4 profile) required as quality gate.

### Phase 6: Cutover + Post-Launch

**Rationale:** Cutover is a timed operation with specific sequencing: pause campaigns -> DNS swap -> SSL verification -> pixel re-verification on live domain -> campaign resume -> 24h attribution check -> Search Console + sitemap submission.
**Delivers:** Live site at https://joguesthouse.my.id; all pixels confirmed firing on production domain; campaigns unpaused; sitemap submitted to Search Console; WP kept on standby 14 days; 30-day owner review scheduled; handover guide for content edits (property.ts single-file approach documented).
**Avoids:** Pitfall 3 (DNS chaos), Pitfall 4 (WP orphan URLs via _redirects), Pitfall 14 (owner content drift).
**Research flag:** Standard process - no research phase needed.

### Phase Ordering Rationale

- Phase 0 before all code: hosting and design direction are forced dependencies (wrong adapter = redeploy; wrong color tokens = rework all 11 sections).
- Phase 1 (content) before Phase 4 (layout): Lorem Ipsum repeat is the most likely failure mode; content inventory is the only gate preventing it.
- Phase 3 (WA primitive) before Phase 4 (sections): WhatsAppButton.astro is imported by ~6 sections; building sections first creates integration debt.
- Phase 5 (verification) before Phase 6 (cutover): broken pixels on cutover = 7-14 day bidding-algorithm re-training cost that exceeds the rebuild cost.
- Phases 2 foundation and 3 image pipeline can overlap once foundation scaffold is established.

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 0:** Design direction - run /design-direction skill to generate 3 visual directions for owner choice. Creative process required before brand tokens can be defined.
- **Phase 5:** Real-device performance verification - WebPageTest from Jakarta with Moto G4 profile is a required QA milestone (LCP <2.5s, TBT <200ms, INP <200ms on Slow 4G).

Phases with standard, well-documented patterns (no additional research needed):
- **Phase 2:** Astro 6 + Tailwind v4 + Cloudflare Pages setup - fully documented in STACK.md and Astro official docs.
- **Phase 4:** All 11 section components follow patterns documented in ARCHITECTURE.md at component-level detail.
- **Phase 6:** Cutover sequence fully specified in ARCHITECTURE.md deploy section and PITFALLS.md.

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | All versions verified live via npm view 2026-05-12; patterns verified via Context7 /withastro/docs. One contradiction resolved: Vercel Hobby replaced by Cloudflare Pages. |
| Features | HIGH (table stakes) / MEDIUM (differentiator weighting) | Table stakes verified against Indonesian hospitality patterns and Traveloka/RedDoorz competitor analysis. Differentiator ROI estimates are single-property inferences. |
| Architecture | HIGH | All patterns verified against Astro 6 conventions. 22-component inventory is complete with explicit dependency graph. |
| Pitfalls | HIGH (Vercel ToS, GTM mechanics, DNS) / MEDIUM (UU PDP enforcement, Partytown flakiness) | Vercel ToS violation documented in Vercel own plan page. GTM risks based on specific pixel IDs in existing container. |

**Overall confidence:** HIGH - the only areas of genuine uncertainty are (1) owner willingness to complete the content inventory gate and (2) exact GTM trigger configuration in the existing container (must be audited directly before Phase 2).

### Gaps to Address

- **Exact GTM container configuration unknown:** The existing container GTM-MK4WJPMF must be audited (owner shares GTM access) before Phase 2 starts. Until audited, trackWaClick event name is an assumption - if existing tags listen for a different event name, all conversion attribution breaks.
- **Exact geo-coordinates unverified:** STACK.md placeholder is lat: -6.165, lng: 106.738. Owner or builder must right-click the exact building pin on Google Maps and record 6-decimal coordinates before JSON-LD is written.
- **OTA listing URLs not confirmed:** src/content/otas.ts will contain Agoda/Traveloka/Booking listing URLs. These must come from the owner OTA dashboard. If listings are inactive or delisted, OTA badge strategy changes.
- **Owner email for Web3Forms unconfirmed:** Not recorded in PROJECT.md. Must be collected before Phase 4 form integration.
- **Design direction not chosen:** @theme color tokens have TODO markers. Phase 0 resolution required; if owner delays, Phase 2 starts with a neutral placeholder palette requiring rework.

---

## Sources

### Primary (HIGH confidence)

- .planning/research/STACK.md (2026-05-12) - Astro 6 + Tailwind v4 stack decisions, pixel routing pattern, hosting analysis
- .planning/research/FEATURES.md (2026-05-12) - 52-feature inventory with MVP definition, WA pre-fill templates, anti-feature rationale
- .planning/research/ARCHITECTURE.md (2026-05-12) - complete component inventory, build-order dependency graph, tracking architecture, cutover plan
- .planning/research/PITFALLS.md (2026-05-12) - 17 pitfalls with Vercel ToS finding, GTM audit protocol, DNS rollback pattern
- Context7 /withastro/docs - Astro 6 install, Tailwind v4 via Vite plugin, astro:assets API, i18n options
- npm view (live 2026-05-12) - confirmed package versions for Astro 6.3.1, Tailwind 4.3.0, and adapters
- schema.org/LodgingBusiness - type hierarchy and required properties
- Google Search Central LocalBusiness structured data - required vs recommended fields
- WhatsApp wa.me URL format (faq.whatsapp.com) - number format, pre-fill encoding

### Secondary (MEDIUM confidence)

- Vercel Hobby plan terms (vercel.com/docs/plans/hobby) - commercial use prohibition; resolves contradiction with STACK.md
- Cloudflare Pages commercial-use confirmation - free tier permits commercial sites
- Community reports (launchfa.st, fatbobman.com, DEV) - Partytown + TikTok Pixel flakiness patterns
- Indonesian hospitality pattern sources (Traveloka, Detik Travel, Salsawisata) - airport-proximity copy conventions, WA booking culture
- Webside.id, JatisMobile, Cozzy.id - WhatsApp booking patterns in Indonesian SMB market

### Tertiary (LOW confidence)

- Indonesian RUM mobile percentiles - no public single-property data; Lighthouse synthetic throttling used as proxy
- UU PDP enforcement against small properties - legal text is HIGH confidence; enforcement likelihood against 17-room guesthouse is LOW based on regulator focus patterns

---

*Research completed: 2026-05-12*
*Ready for roadmap: yes*
