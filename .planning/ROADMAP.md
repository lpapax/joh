# Roadmap: Jo Guest House — Website Rebuild

**Created:** 2026-05-12
**Granularity:** coarse
**Mode:** yolo
**Structure:** Horizontal layers (setup → content → foundation → primitive → sections → cutover)
**Core Value:** Turn existing TikTok / Meta / Google Ads paid traffic into actual WhatsApp bookings — converting better than the current half-finished WordPress site.

This roadmap is built around the **cutover-day constraint**: ad campaigns must continue without going dark on the new site. Phases are sequenced by hard dependency, not by user-facing value increments, because the project ships in one cutover event, not in iterative releases.

## Phases

- [ ] **Phase 1: Setup + Key Decisions** — Lock hosting choice, run /design-direction, finalize brand tokens
- [ ] **Phase 2: Content Inventory Gate** — Every visible string typed, photos curated, owner signs off (hard gate)
- [ ] **Phase 3: Foundation + Tracking Scaffold** — Astro 6 + Tailwind v4 + GTM chokepoint + SEO head + JSON-LD wiring
- [ ] **Phase 4: WhatsApp Primitive + Image Pipeline** — WA component + UTM + tracking chokepoint + curated photos
- [ ] **Phase 5: Sections + SEO Schema + Form** — All 11 sections, LodgingBusiness JSON-LD, Web3Forms with hCaptcha
- [ ] **Phase 6: Tracking Verification + Cutover** — Pre-cutover pixel verification, DNS swap, post-launch monitoring

## Phase Details

### Phase 1: Setup + Key Decisions
**Goal**: Hosting and visual direction are locked before any line of layout code is written, so no rework cascades through 11 sections later.
**Depends on**: Nothing (first phase)
**Requirements**: SETUP-01, SETUP-02, SETUP-03
**Success Criteria** (what must be TRUE):
  1. PROJECT.md Key Decisions table reflects a confirmed hosting choice (Cloudflare Pages free OR Vercel Pro paid) with the owner-named account already created and accessible
  2. Three distinct visual design directions exist (color palette + typography scale + section mood for each) and the owner has explicitly chosen one in writing
  3. Chosen direction's brand tokens (colors, font-family, spacing scale) are locked in `src/styles/global.css` `@theme` block with no remaining TODO markers
  4. Vercel Hobby is documented as rejected with the ToS-violation reason linked in PROJECT.md
**Plans**: TBD
**UI hint**: yes

### Phase 2: Content Inventory Gate
**Goal**: Every visible string and every published photo is real, owner-approved, and Indonesian-correct before any layout code is written — preventing the Lorem Ipsum / USD-price failure mode that broke the current WordPress site.
**Depends on**: Phase 1 (brand tokens needed to mock content tone, though content text itself does not need Phase 1)
**Requirements**: CONT-01, CONT-02, CONT-03, CONT-04, CONT-07, CONT-08
**Success Criteria** (what must be TRUE):
  1. `src/content/site.ts` exists with every visible string (hero, trust bar, about, rooms, facilities, location, OTA labels, testimonials, FAQ, footer) typed in Bahasa with English fallback strings populated for the documented key subset
  2. 12–18 photos selected from the ~40 WP source images, listed by filename and intended section in `.planning/content-inventory.md`, and owner has ticked the sign-off line
  3. A 1200×630 Open Graph image draft exists at `public/og-image.jpg` showing property name + "Rp 200.000" + "dekat CGK" overlay
  4. CI grep script blocks build if `dist/` contains "Lorem Ipsum", "Add Your Heading Text Here", "A Title to Turn the Visitor Into a Lead", or any stray `$` USD price string
  5. Owner has signed the inventory gate line in `.planning/content-inventory.md` — no Phase 3 work begins before this signature
**Plans**: TBD

### Phase 3: Foundation + Tracking Scaffold
**Goal**: A deployable Astro project exists on the chosen host with tracking, SEO head, JSON-LD container, fonts, and i18n scaffolding all wired — every later phase imports from this foundation rather than installing it.
**Depends on**: Phase 1 (hosting + brand tokens), Phase 2 (site.ts shape)
**Requirements**: TRACK-01, TRACK-02, TRACK-03, TRACK-04, SEO-01, SEO-02, SEO-03, SEO-06, SEO-08, PERF-05, PERF-06, PERF-07, PERF-08
**Success Criteria** (what must be TRUE):
  1. Staging URL on the chosen host responds with HTTP 200, serves a blank `BaseLayout` page with brand tokens applied, and shows Plus Jakarta Sans loaded from `@fontsource` (no Google Fonts CDN request)
  2. GTM container `GTM-MK4WJPMF` is installed via inline `<script is:inline>` in `<head>` plus `<noscript>` iframe in `<body>`, and GTM Preview Mode connects successfully to the staging URL
  3. Existing GTM container audit document exists in `.planning/gtm-audit.md` listing every current tag, trigger, and variable with screenshots, before any new GTM changes are made
  4. `src/lib/track.ts` exports `trackWaClick`, `trackOtaClick`, `trackEnquirySubmit` and is the only file in the repo that references `window.dataLayer` (verified by grep)
  5. `<title>`, meta description, canonical URL, and OG/Twitter meta tags render on `view-source:staging-url/` with real Bahasa values — not "My Blog – My WordPress Blog"
  6. Lighthouse Mobile run on the staging foundation scores ≥ 95 SEO and total first-party JS load is under 50 KB (excluding GTM container script)
**Plans**: TBD

### Phase 4: WhatsApp Primitive + Image Pipeline
**Goal**: The single most-reused component (WhatsApp button) works correctly on every browser the audience uses, attribution is preserved end-to-end, and all 12–18 curated photos are pipeline-ready for section consumption.
**Depends on**: Phase 2 (curated photo list + WA pre-fill copy), Phase 3 (track.ts chokepoint)
**Requirements**: WA-01, WA-02, WA-03, WA-04, WA-05, WA-06, WA-07, TRACK-07, CONT-05, CONT-06
**Success Criteria** (what must be TRUE):
  1. WhatsApp button click on iOS Safari desktop opens WhatsApp with the pre-filled Bahasa message "Halo, saya tertarik untuk pesan kamar di Jo Guest House. Sumber: [utm_source]" including the UTM source token from the page URL
  2. Same WhatsApp click works from inside the TikTok in-app browser and the Instagram in-app browser on Android (verified on real device, screenshots saved to `.planning/wa-device-matrix.md`)
  3. WhatsApp number in the deep-link URL is exactly `6285108002536` (no `+`, no spaces, no dashes) and uses the `api.whatsapp.com/send` endpoint, not `wa.me`
  4. Sticky mobile bottom CTA bar (WhatsApp + Maps + Call) is invisible above the fold, fades in after first scroll, and is hidden on viewport widths ≥ 768px
  5. Every WhatsApp click fires exactly one `whatsapp_click` event into the dataLayer via `lib/track.ts` — verified in GTM Preview Mode with the event payload showing the page-source UTM token
  6. All 12–18 curated photos live in `src/assets/photos/`, are imported via `astro:assets` `<Image>` with `priority` set only on the hero image, and the build output contains AVIF + WebP + JPEG variants with responsive `srcset`
**Plans**: TBD
**UI hint**: yes

### Phase 5: Sections + SEO Schema + Form
**Goal**: All 11 section components render real content on the scrolling page, JSON-LD validates clean in Rich Results Test, and the enquiry form delivers test submissions to the owner inbox with hCaptcha enabled.
**Depends on**: Phase 3 (foundation), Phase 4 (WA component + image pipeline)
**Requirements**: SECT-01, SECT-02, SECT-03, SECT-04, SECT-05, SECT-06, SECT-07, SECT-08, SECT-09, SECT-10, SECT-11, SEO-04, SEO-05, SEO-07, FORM-01, FORM-02, FORM-03, FORM-04, FORM-05, FORM-06, PERF-01, PERF-02, PERF-03, PERF-04
**Success Criteria** (what must be TRUE):
  1. Staging URL renders all 11 sections in scroll order (Hero, TrustBar, About, Rooms, Facilities, Location, OtaBadges, Testimonials, FAQ, ContactForm, Footer) with real owner-approved content from Phase 2 — no placeholder copy anywhere
  2. Google Rich Results Test on the staging URL reports the `LodgingBusiness` JSON-LD as valid with zero errors and zero warnings, with `geo` coordinates matching the real Google Maps pin (6-decimal precision) and `sameAs` containing live Agoda / Traveloka / Booking.com listing URLs
  3. The Web3Forms contact form accepts an Indonesian phone number in any of the formats `+62 851...`, `08510800...`, `628510800...`, blocks submission without an hCaptcha token, and on success shows a Bahasa toast plus fires `trackEnquirySubmit` (verified in GTM Preview Mode)
  4. Web3Forms delivers three test submissions to the owner-confirmed email address with all form fields visible
  5. Lighthouse Mobile run on the staging URL scores ≥ 90 Performance, ≥ 95 Accessibility, ≥ 95 SEO, ≥ 95 Best Practices with LCP < 2.5s, CLS < 0.05, TBT < 200ms
  6. Old WordPress URLs (`/sample-page/`, `/?p=N`, `/wp-admin/`) return either a 301 to `/` via host-layer redirect rules or a sitemap-excluded `noindex` response — verified with `curl -I` against staging
**Plans**: TBD
**UI hint**: yes

### Phase 6: Tracking Verification + Cutover
**Goal**: Cutover happens with zero ad-spend dark time — all four pixels are confirmed firing on the production domain before DNS propagation completes, and a rollback path exists for the four-hour post-launch window.
**Depends on**: Phase 5 (verifier must pass before this phase starts)
**Requirements**: TRACK-05, TRACK-06, DEPLOY-01, DEPLOY-02, DEPLOY-03, DEPLOY-04, DEPLOY-05, DEPLOY-06, DEPLOY-07, DEPLOY-08
**Success Criteria** (what must be TRUE):
  1. Meta Events Manager Test Events tab shows `whatsapp_click` and `enquiry_submit` events arriving on the staging URL from a manual test session, attributed to Meta Pixel `770014465395193` — screenshot saved to `.planning/cutover-checklist.md`
  2. TikTok Events Manager and Google Tag Assistant both confirm the same two events firing through TikTok Pixel `D2MR5GRC77U4PA826B90` and Google Ads conversion `AW-17438288457` respectively
  3. DNS TTL on `joguesthouse.my.id` was lowered to 300 seconds at least 48 hours before the DNS swap (verified via `dig joguesthouse.my.id +noall +answer` timestamped log)
  4. After DNS swap, `https://joguesthouse.my.id/` resolves to the new host with a valid SSL certificate, the new content renders, and the same four-pixel verification protocol passes a second time against the production domain (not just staging)
  5. WordPress origin remains live and reachable via its origin IP for 7 days post-cutover, documented in `.planning/cutover-checklist.md` as a rollback target
  6. Owner stays available on WhatsApp for the T+0 to T+4h window after DNS swap and confirms at least one inbound booking message arrived during the window (or, if none, confirms the pixel events fired in Events Manager during that window)
**Plans**: TBD

## Phase Dependencies

```
Phase 1 (Setup)
   ↓
Phase 2 (Content Gate) ──── HARD GATE: owner sign-off required ────┐
   ↓                                                                │
Phase 3 (Foundation) ←─── needs brand tokens from P1, site.ts from P2
   ↓                       │
Phase 4 (WA Primitive) ←───┴── needs track.ts from P3, photos from P2
   ↓
Phase 5 (Sections) ←────── needs WA from P4, foundation from P3
   ↓
   verifier gate (config: workflow.verifier=true) must pass
   ↓
Phase 6 (Cutover) ←─────── needs everything; no DNS work until verifier passes
```

## Hard Gates

| Gate | Between | Trigger | Why |
|------|---------|---------|-----|
| Owner content sign-off | P2 → P3 | Owner ticks the sign-off line in `.planning/content-inventory.md` | Prevents Lorem Ipsum / USD-price repeat — the same failure mode that broke the current WP site |
| Verifier pass | P5 → P6 | All P5 success criteria observably TRUE on staging | `workflow.verifier=true` in config.json; DNS swap with broken pixels = 7–14 day ad-bidding-algorithm re-training cost |

## Progress

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Setup + Key Decisions | 0/TBD | Not started | — |
| 2. Content Inventory Gate | 0/TBD | Not started | — |
| 3. Foundation + Tracking Scaffold | 0/TBD | Not started | — |
| 4. WhatsApp Primitive + Image Pipeline | 0/TBD | Not started | — |
| 5. Sections + SEO Schema + Form | 0/TBD | Not started | — |
| 6. Tracking Verification + Cutover | 0/TBD | Not started | — |

## Coverage

**v1 requirements:** 66 total (REQUIREMENTS.md count; the instructions cited 60 but the actual REQ-ID list totals 66: 3 SETUP + 8 CONT + 11 SECT + 7 WA + 7 TRACK + 8 SEO + 8 PERF + 6 FORM + 8 DEPLOY)
**Mapped:** 66 / 66 ✓
**Orphaned:** 0

See `REQUIREMENTS.md` Traceability table for full REQ-ID → phase mapping.

---
*Roadmap created: 2026-05-12*
*Last updated: 2026-05-12*
