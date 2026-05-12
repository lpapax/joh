# Feature Research

**Domain:** Indonesian budget guesthouse / one-page marketing+booking landing site (WA-first, ad-traffic-driven, near-airport)
**Researched:** 2026-05-12
**Confidence:** HIGH for table-stakes/anti-features (well-established patterns); MEDIUM for differentiator weighting (single-property context is under-documented; chain analogs over-fit)

## Context Recap

Jo Guest House is a 17-room single-tier (Rp 200k/night) guesthouse in Cengkareng, ~minutes from CGK. Audience is overwhelmingly mobile Indonesian travelers arriving via paid social (TikTok/Meta) and Google Ads. Booking goes through WhatsApp + OTAs (Agoda, Traveloka, Booking.com). Scope is one Bahasa-primary scrolling page in Astro+Tailwind.

The "competitor" reference set (RedDoorz, OYO, Airy property pages) is informative but misleading — those are chain platform listings, not standalone property sites. They over-engineer because they're funnel pages inside a booking engine. We're the opposite: a brochure-with-CTA whose job is to push a tap to WhatsApp.

## Feature Landscape

### Table Stakes (Users Expect These)

Missing = visitor bounces or distrusts the property. Non-negotiable for launch.

| # | Feature | Why Expected | Complexity | Notes |
|---|---------|--------------|------------|-------|
| TS-01 | Hero with property name + 1-line value prop + price + airport distance + primary CTA | Indonesian budget-stay shoppers scan hero in <3s on mobile; everything must be visible above the fold without scroll | S | Slogan already exists ("Hotel Nyaman Gak Harus Bintang Lima"). Hero must contain: property name, slogan, "X menit dari Bandara Soekarno-Hatta", "Mulai Rp 200.000/malam", WA CTA. |
| TS-02 | Sticky bottom WhatsApp CTA bar (mobile) | Indonesian mobile users expect WA to be 1 tap away anywhere on the page; competitors all have this; ~90% mobile audience makes this load-bearing | S | Bottom-bar pattern with WA (primary, full color), Call, and Maps icons. 48dp tap targets. Hide on desktop or convert to floating bubble. |
| TS-03 | Click-to-chat WhatsApp deep link with pre-filled message | Friction removal — users won't type "Halo, mau pesan kamar"; the link must do it for them | S | `https://wa.me/6285108002536?text=` with URL-encoded template (see WA-TEMPLATE section below). |
| TS-04 | Tap-to-call phone link (`tel:`) | Older/less tech-savvy Indonesian segment still calls; mandatory secondary contact | S | Same number as WA, but `tel:+6285108002536`. Don't make it the primary CTA — WA wins on async-friendliness. |
| TS-05 | Rooms section with photos + price + "what's included" | Visitors need to see the room before considering booking; even at single-tier pricing, photo trust drives conversion | S | Single card or carousel for the room type. List inclusions: kasur, AC (if applicable), kamar mandi, sprei bersih, etc. Price displayed once, prominently. |
| TS-06 | Facilities/amenities section with icons | Standard pattern across RedDoorz/OYO/Airy/Traveloka listings; users scan icons to assess value | S | Icon grid: Free WiFi, Cafe, Water Heater, Room Service, 24 Jam Resepsionis, CCTV, Parkir. Use line icons (Lucide/Heroicons), not photos. |
| TS-07 | Location section: embedded Google Map + written address + "X menit dari CGK" | Trust signal + practical info; "menit dari bandara" is the dominant near-airport copy pattern across Indonesian listings | S-M | Lazy-loaded iframe map (or static map image + "Buka di Google Maps" link to avoid Maps JS overhead). Full address text. Distance/time stated explicitly with traffic caveat ("±20 menit normal, ±35 menit jam sibuk"). |
| TS-08 | Photo gallery (rooms, common areas, exterior, cafe) | Visual proof; Indonesian budget-stay shoppers are skeptical and want to see the actual property | M | 12-20 photos max. Lightbox on tap. Grid on desktop, horizontal scroll-snap strip on mobile. Lazy-load below the fold. Avoid stock photos — visible inauthenticity kills trust faster than imperfect real photos. |
| TS-09 | OTA badges with outbound links | Trust through familiar brands; ad-driven visitors who don't trust `.my.id` TLD use OTA as a fallback trust path | S | Agoda, Traveloka, Booking.com logos at recognizable size (≥40px height). Open in new tab. Place AFTER WA CTA, not before, so OTAs aren't the primary path. |
| TS-10 | Operating hours / check-in info | Anxiety reduction: "can I arrive at 2am from a delayed flight?" is a real near-airport question | S | "Check-in: 14:00 / Check-out: 12:00. Resepsionis 24 jam — boleh check-in larut malam (info via WA)." |
| TS-11 | SEO basics: real `<title>`, meta description, OG image | Without these, ad clicks land on a page that looks broken in previews and search; current site fails all three | S | Already in PROJECT.md active list. OG image should be the best hero/room photo with property name overlaid. |
| TS-12 | Schema.org `LodgingBusiness` JSON-LD | Google Hotel pack / local pack visibility; cheap to implement, lifts organic | S | `@type: LodgingBusiness` (or `Hotel`), with address, geo, telephone, priceRange, amenityFeature array, image, sameAs (OTA URLs). |
| TS-13 | Tracking pixels (Meta, TikTok, GTM, Google Ads conversion) functional from day 1 | Owner is currently spending on paid traffic; if pixels don't fire on cutover, attribution dies | S | Hard-coded snippets in Astro layout; fire WA-click as conversion event for Meta/TikTok/Google Ads. This IS the business case. |
| TS-14 | Mobile-first responsive (≥360px viewport tested) | ~90% of Indonesian web traffic is mobile; many users on older Android with cheaper screens | S | Tailwind default mobile-first is fine. Test at 360×640 explicitly. No horizontal scroll. |
| TS-15 | Fast page load (LCP <2.5s on 3G/4G) | Indonesian mobile networks vary; ad-driven traffic abandons slow loads; current WP site is slow | S-M | Astro static output handles this by default. Image optimization (AVIF/WebP), no client JS beyond pixel snippets + GSAP-lite. |
| TS-16 | Price displayed once, prominently, in Indonesian format | Confusion = bounce; current WP shows USD ($125-$425) which is WRONG by 10× | S | Format: `Rp 200.000` with dot as thousands separator (Indonesian convention). Append `/malam`. Avoid `Rp 200rb` on landing hero — looks unprofessional; "ribuan" is fine in body copy. |
| TS-17 | Bahasa Indonesia copy throughout | Primary audience is Indonesian; English-only would alienate ad-driven traffic | S | English fallback ONLY on: OTA badge labels, hero microcopy for international OTA-referred visitors (optional). No language toggle UI — that's an anti-feature (see AF-02). |
| TS-18 | Contact section with address, WA, phone, email (if any), hours | Standard "we are a real business" footer; expected on any business site | S | At the bottom. Include physical address (Ruko Plaza de Lumina, Jl. Outer Ring Road...). |

### Differentiators (Competitive Advantage)

Features that turn a generic budget-hotel landing page into one that converts paid traffic specifically for THIS property.

| # | Feature | Value Proposition | Complexity | Notes |
|---|---------|-------------------|------------|-------|
| DF-01 | "Distance from CGK" as a hero element, not a footnote | Airport proximity IS the value prop; chains bury it 3 sections down because they aren't location-specialized. We surface it in the hero badge. | S | Pattern: hero subhead reads `±20 menit ke Bandara Soekarno-Hatta` with airplane icon. Reinforce in location section with traffic-aware estimate. |
| DF-02 | WhatsApp pre-fill with structured prompts (date, # guests, # nights) | Lowers conversation cold-start friction; owner gets a usable message instead of "hai mas?" | S | Template (URL-encoded): `Halo Jo Guest House, saya mau pesan kamar.%0A%0ATanggal check-in: ___%0ATanggal check-out: ___%0AJumlah tamu: ___%0AAda pertanyaan: ___%0A%0ATerima kasih!` |
| DF-03 | Multiple WA entry points with different pre-fills (book vs ask vs directions) | Different intents get different conversations; book-intent pre-fill = reservation template, "tanya dulu" = general question | S | 3 CTAs total: hero "Pesan via WhatsApp", rooms "Tanyakan Ketersediaan", location "Minta Arah / Petunjuk". All same number, different `?text=`. |
| DF-04 | "Cocok untuk transit" angle / use case framing | The property's geographic position has a specific persona (transit traveler, early/late flight); naming the use case helps self-selection | S | Mini section: "Cocok untuk: transit penerbangan pagi, bisnis di Cengkareng/JORR, keluarga dengan budget terjangkau." 3 line items, no icons needed. |
| DF-05 | Static testimonials block (3-5 real quotes) | Social proof without scraping OTAs (which would violate ToS). Real first-name + city + month. | S | "Mas Andi, Surabaya — April 2026: 'Nyaman, dekat bandara, harga oke.'" Mark as "Ulasan tamu" not "Reviews dari Agoda". Avoid fake-looking 5-star inflation; mix 4 and 5 star wording. |
| DF-06 | "Rated on Agoda/Traveloka" stat callout (if true & current) | Borrows OTA trust without embedding live data | S | "★ 4.x di Agoda · ★ 4.x di Traveloka" as plain text, hyperlinked to listings. Update quarterly. RISK: if rating drops, you have to update — auditable claim. Only use if owner confirms current ratings. |
| DF-07 | Sticky CTA that changes label by scroll position | Hero CTA: "Pesan via WhatsApp". After scrolling past rooms: "Cek Kamar". After contact section: "Tanya Lokasi". Keeps CTA contextual. | M | JS-light intersection observer toggling sticky bar label. Skip if it adds >5KB JS. |
| DF-08 | One-tap "Share di WhatsApp" button | Indonesian visitors share trip plans via WA group chats with family/spouse before booking; making sharing 1-tap captures this | S | `https://wa.me/?text=...` (no number = pick contact). Pre-fill: "Lihat penginapan ini, dekat bandara, Rp 200rb: https://joguesthouse.my.id" |
| DF-09 | "Apa kata tamu" trust strip with real photos (with consent) | Photos of real guests (room selfies / family arriving) beat stock testimonials. Get explicit permission. | M | 3-4 small photos with captions. Defer if no content; can be added in v1.x. |
| DF-10 | FAQ section answering airport-specific anxieties | "Bisa check-in jam 2 pagi?" "Ada antar-jemput bandara?" "Bisa titip koper kalau check-out tapi flight malam?" — these convert hesitant transit guests | S | 5-8 Q&A in accordion. Schema.org `FAQPage` JSON-LD for SEO bonus. |
| DF-11 | Conversion event firing on WA-CTA tap | Owner can measure paid-traffic → WA-tap conversion rate per source (TikTok vs Meta vs Google Ads). This is the entire business case for the rebuild. | S | `data-conversion="wa_click"` attribute, dispatch event to Meta/TikTok/Google Ads conversion APIs via existing pixels. Document each event name. |
| DF-12 | Pre-filled message includes UTM/source token | Owner sees in WA: "from TikTok ad campaign X" — closes the loop manually since no PMS | M | Read `utm_source` from URL, append to WA pre-fill text. E.g., `...Sumber: TikTok-Promo-Lebaran`. Cheap, very high information value. |
| DF-13 | Single price displayed as a feature, not a limitation | "Satu harga, semua kamar — Rp 200.000/malam. Tanpa kejutan." Turn flat pricing into clarity-as-feature. | S | Copy positioning only, no code. Differentiates from OTA pages with dynamic pricing. |
| DF-14 | "Direct booking" implicit advantage (no parity-violating language) | Don't say "lebih murah dari Agoda" (parity violation); DO say "Pesan langsung via WhatsApp, respon cepat, fleksibel." Implicit speed/personal advantage. | S | Copy positioning. See PITFALLS for OTA parity language. |
| DF-15 | Lightweight motion (GSAP scroll reveal) | Modern feel for budget brand; counter-signals the `.my.id` TLD trust gap | S | Subtle fade-up on section enter. Keep <10KB GSAP-core. Respect `prefers-reduced-motion`. |
| DF-16 | OG/Twitter card optimized for ad-share previews | When the URL is shared via WA (huge in Indonesia), the preview card determines whether it gets tapped | S | OG image: hero photo + property name + price + "dekat CGK" overlay. 1200×630. Test in WA link preview. |

### Anti-Features (Commonly Requested, Often Problematic)

| # | Anti-Feature | Why Requested | Why Problematic | Alternative |
|---|--------------|---------------|-----------------|-------------|
| AF-01 | Real-time availability calendar / on-site booking engine | "Looks professional"; chains have it | No PMS to sync against. Stale data = bad UX worse than no calendar. Owner already manages availability via WA conversation. Listed in PROJECT.md Out of Scope. | WA CTA as primary; "Cek ketersediaan via WhatsApp" copy. |
| AF-02 | Language toggle UI (ID ↔ EN) | "International audience"; OTAs have it | Adds complexity, splits content maintenance, and the actual foreign audience comes via OTAs which handle their own language. Listed in PROJECT.md Out of Scope. | Bahasa primary; English fallback on a few key strings (OTA badges, hero subtitle optional). |
| AF-03 | Per-room detail pages | "Each room deserves its own page" | 17 rooms × single price × same type = zero differentiation. Pages would be duplicate content, hurting SEO. | One rooms section with one representative room card + gallery. |
| AF-04 | Blog / content marketing | SEO advice from generic guides | 17-room budget guesthouse can't sustain editorial cadence; blog rot signals abandonment. Listed in PROJECT.md Out of Scope. | Static FAQ section (DF-10) covers long-tail intents without ongoing work. |
| AF-05 | OTA review scraping/embedding | "Show real reviews" | Violates Agoda/Booking ToS; technically fragile (DOM changes); stale on cache. Listed in PROJECT.md Out of Scope. | Static testimonials (DF-05) + linked OTA ratings as text claim (DF-06). |
| AF-06 | Account / login / saved bookings | "Modern sites have accounts" | Property has no PMS, no loyalty program, no repeat-booking system. Adds auth complexity, GDPR/UU PDP exposure, password recovery, login UI on a page that should be a brochure. | None — drop entirely. |
| AF-07 | Loyalty program / discount codes | "Drive repeat business" | Budget guesthouse, single price — no margin for discounts. Repeat business comes from owner relationship via WA. | None — drop entirely. |
| AF-08 | Newsletter signup / email capture | "Build a list" | Owner has no CRM, no email cadence plan, no Mailchimp budget. List would die. Inbox visitor friction for zero return. | WhatsApp follow-up on first booking is the real "list." |
| AF-09 | Multi-step booking form (date, guests, name, email, ...) | "Capture leads properly" | Indonesian users abandon long forms en masse; WA pre-fill (DF-02) captures equivalent info conversationally with no form abandonment. | WA pre-fill template (DF-02) + optional WPForms-style contact form as fallback only. |
| AF-10 | Group/event/wedding packages | "More revenue streams" | 17-room budget property in a ruko isn't event-suitable; misleading for the audience. | None — drop entirely. Owner can handle bespoke group inquiry via WA on the rare occurrence. |
| AF-11 | Gift cards / vouchers | "Holiday revenue boost" | Budget guesthouse, no fulfillment infrastructure, no payment processor (PROJECT.md Out of Scope). | None — drop entirely. |
| AF-12 | Live chat widget (Tawk.to / Crisp / Intercom) | "Convert browsers to chatters" | Adds 50-150KB JS; WhatsApp IS the live chat, owned by the user, async-friendly, runs offline. Native pattern wins. | WA CTAs everywhere (TS-02, TS-03, DF-03). |
| AF-13 | Currency switcher | "Foreign tourists need USD/SGD" | Foreign OTA traffic books on OTA in their currency anyway. Adds maintenance burden for an audience this site isn't acquiring. | Show Rp only. If hero subtitle in English mentioned (TS-17), note "~$13 USD" once. |
| AF-14 | Animated full-screen hero video / parallax | "Premium look" | Heavy assets, slow LCP, distracts from CTA, doesn't match budget brand. Counter-signals price honesty. | Single optimized hero photo (still). Subtle GSAP reveal only (DF-15). |
| AF-15 | "Pesan lebih murah di sini" / explicit OTA-undercutting copy | Easy direct-booking win | Violates rate parity clauses (Agoda, Booking.com, Traveloka contracts). OTA can delist or demote the property. | DF-14: implicit advantage framing ("respon cepat, langsung dari pemilik"). |
| AF-16 | Cookie consent banner / GDPR popup | "Compliance" | UU PDP (Indonesian privacy law) doesn't mandate cookie banners the way GDPR does for first-party tracking. Banner = bounce risk on ad traffic. | Privacy policy page (linked from footer) describing pixel use is sufficient for the audience. Revisit if EU traffic ever becomes significant. |
| AF-17 | Map JS library (Mapbox / Leaflet) interactive map | "Better than iframe" | Adds 100+ KB JS; iframe Google Maps embed (or static image + link) is good enough for "see where we are". | Iframe map with `loading="lazy"`, OR static map image (Maps Static API) + "Buka di Google Maps" link. |
| AF-18 | Carousel of "similar properties" / "you might also like" | OTA pattern bleed-through | This is a single property; nothing to cross-sell. Chain platforms do this; we are not a chain. | None — drop entirely. |

## Feature Dependencies

```
TS-13 (Tracking pixels)
    └──requires──> TS-11 (SEO basics / Astro layout slots)

DF-11 (Conversion event on WA tap)
    └──requires──> TS-13 (Pixels installed) + TS-03 (WA CTA exists)

DF-12 (UTM in WA pre-fill)
    └──requires──> TS-03 (WA pre-fill) + small client JS for URL parsing

TS-02 (Sticky WA bar)
    └──requires──> TS-03 (WA deep link with pre-fill)

DF-07 (Context-aware sticky CTA)
    └──requires──> TS-02 (sticky bar) + IntersectionObserver
    └──enhances──> DF-03 (multiple WA entry points)

DF-10 (FAQ)
    └──enhances──> TS-12 (Schema.org JSON-LD via FAQPage type)

DF-08 (Share via WA)
    └──requires──> DF-16 (Good OG card to make share preview useful)

TS-12 (LodgingBusiness schema)
    └──requires──> TS-07 (Location data) + TS-06 (amenity list as amenityFeature)

DF-05 (Testimonials) ──conflicts──> AF-05 (OTA review scraping)
DF-06 (OTA rating claim) ──conflicts──> AF-15 (parity-violating copy)
```

### Dependency Notes

- **Tracking before anything fancy:** TS-13 + DF-11 must work day 1 — that's the cutover business case. Don't ship without verifying each pixel fires.
- **WA pre-fill is the linchpin:** TS-03 is the foundation under TS-02, DF-02, DF-03, DF-08, DF-11, and DF-12. Spend disproportionate time getting the template right.
- **Schema benefits compound:** TS-12 + DF-10 (FAQ schema) + TS-07 (geo data) together drive local SEO. They're cheap individually but most valuable together.
- **DF-05 vs AF-05:** Static testimonials and OTA scraping serve the same goal (social proof) but the latter is a ToS landmine. Pick the safe one.
- **DF-06 vs AF-15:** Both touch OTA relationship. DF-06 borrows OTA trust without violating parity; AF-15 violates parity directly. The line is narrow but real.

## MVP Definition

### Launch With (v1) — the actual ship list

All P1 features below must be present for cutover. The bar: paid ad traffic landing on this page converts to WA-tap at a measurable rate higher than the current WordPress site (which is ~0).

- [ ] TS-01 Hero (name, slogan, price, airport distance, WA CTA) — value prop in 1 screen
- [ ] TS-02 Sticky mobile bottom CTA bar (WA + Call + Maps)
- [ ] TS-03 WA click-to-chat deep link with pre-filled message
- [ ] TS-04 Tap-to-call
- [ ] TS-05 Rooms section (one card, photos, price, inclusions)
- [ ] TS-06 Facilities icon grid
- [ ] TS-07 Location: embedded map + address + "X menit dari CGK"
- [ ] TS-08 Photo gallery (12-20 photos, lightbox, lazy)
- [ ] TS-09 OTA badges (Agoda, Traveloka, Booking) outbound links
- [ ] TS-10 Check-in info / 24h reception note
- [ ] TS-11 Real SEO basics (title, meta description, OG image)
- [ ] TS-12 LodgingBusiness JSON-LD
- [ ] TS-13 All tracking pixels fire (Meta, TikTok, GTM, Google Ads conversion)
- [ ] TS-14 Mobile-first responsive verified at 360px
- [ ] TS-15 LCP <2.5s on 4G
- [ ] TS-16 Single prominent price in Rp format
- [ ] TS-17 Bahasa primary copy
- [ ] TS-18 Contact section
- [ ] DF-01 Airport proximity as hero element (not footnote)
- [ ] DF-02 Structured WA pre-fill (dates, guests, nights)
- [ ] DF-03 Multiple WA entry points with intent-specific pre-fills
- [ ] DF-11 WA-tap conversion event firing
- [ ] DF-12 UTM/source token appended to WA pre-fill — high ROI, low cost
- [ ] DF-14 Direct-booking soft framing (parity-safe copy)
- [ ] DF-16 OG card optimized for WhatsApp share previews

### Add After Validation (v1.x)

Trigger: site is converting, owner is getting WA bookings, ready for incremental optimization.

- [ ] DF-04 "Cocok untuk transit" use-case framing — add once owner confirms transit is dominant persona (verify via WA pre-fill UTM data)
- [ ] DF-05 Static testimonials block — needs real quotes from owner; add once 5-10 real ones collected
- [ ] DF-06 OTA rating claim — add only after owner confirms current ratings are ≥4.0
- [ ] DF-09 Real guest photos with consent — needs content collection
- [ ] DF-10 FAQ section + FAQPage schema — add once owner provides answers to 5-8 real recurring WA questions
- [ ] DF-13 Single-price-as-feature copy — A/B test against generic copy
- [ ] DF-15 GSAP scroll reveals — polish pass once content stabilized
- [ ] DF-08 Share via WhatsApp button — add once OG card is dialed in

### Future Consideration (v2+)

Defer until evidence demands it.

- [ ] DF-07 Context-aware sticky CTA label — only if heatmap shows CTA fatigue
- [ ] Per-section A/B testing infrastructure — only if traffic volume justifies (current scale may not)
- [ ] English-on-key-strings layer — only if OTA-referred direct traffic becomes significant
- [ ] Multi-page expansion (separate /rooms, /location, /faq) — only if SEO data shows long-tail keyword opportunities the one-pager can't capture
- [ ] Real-time availability — only if owner adopts a PMS (Cloudbeds, Beds24, etc.)

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| TS-01 Hero | HIGH | LOW | P1 |
| TS-02 Sticky mobile CTA | HIGH | LOW | P1 |
| TS-03 WA pre-fill link | HIGH | LOW | P1 |
| TS-13 Pixels firing | HIGH (business) | LOW | P1 |
| DF-11 WA conversion event | HIGH (business) | LOW | P1 |
| DF-01 Airport distance in hero | HIGH | LOW | P1 |
| TS-12 LodgingBusiness schema | MEDIUM | LOW | P1 |
| TS-08 Photo gallery | HIGH | MEDIUM | P1 |
| TS-07 Location + map | HIGH | LOW | P1 |
| DF-12 UTM in WA pre-fill | HIGH (business) | LOW | P1 |
| DF-02 Structured WA pre-fill | HIGH | LOW | P1 |
| DF-16 OG card for WA share | HIGH | LOW | P1 |
| TS-15 Fast LCP | MEDIUM | LOW (Astro default) | P1 |
| DF-10 FAQ section | MEDIUM | LOW | P2 |
| DF-05 Testimonials | MEDIUM | LOW | P2 |
| DF-04 Transit use-case framing | MEDIUM | LOW | P2 |
| DF-06 OTA rating claim | MEDIUM | LOW | P2 |
| DF-15 GSAP reveals | LOW | LOW | P2 |
| DF-09 Guest photos w/ consent | MEDIUM | MEDIUM | P2 |
| DF-08 Share via WA | LOW | LOW | P2 |
| DF-07 Context-aware CTA label | LOW | MEDIUM | P3 |
| DF-13 Single-price-as-feature | LOW | LOW (copy) | P3 |

## Competitor Feature Analysis

| Feature | RedDoorz property page | Traveloka property page | Independent IDN guesthouse sites | Our Approach |
|---------|------------------------|-------------------------|----------------------------------|--------------|
| Hero | Brand-led, RedDoorz logo dominant, property name secondary | Photo carousel + price + location + booking widget | Often weak (placeholder, lorem ipsum) | Property-first, slogan-led, price+distance+WA in one screen |
| Pricing | Dynamic, range, "from Rp X" | Dynamic w/ promo strikethrough | Often missing or wrong (our current site is in USD!) | Single price, displayed once, Rp 200.000/malam, no strikethrough fakery |
| Rooms | Multiple room types w/ price tiers | Multiple room types w/ booking widget per type | Often a single carousel | Single room card (matches reality: all 17 same) |
| Facilities | Icon grid + filterable list | Icon grid + detailed amenity list | Often a bullet list or table | Icon grid, 6-8 most relevant, no filtering |
| Map | Embedded interactive map | Embedded interactive map w/ POI overlays | Often a static screenshot or missing | Lazy iframe (good enough) + "Buka di Google Maps" |
| Reviews | Live OTA reviews displayed | Aggregated review score + carousel of quotes | Often missing or fake-looking | Static testimonials (DF-05) + optional OTA rating callout (DF-06) — no scraping |
| Booking | In-platform booking engine | In-platform booking engine | Often broken contact forms | WhatsApp deep link with structured pre-fill (DF-02) |
| Trust signals | Chain brand, verified reviews, OTA inventory | Verified property badge, traveler choice awards | Often minimal | OTA badges + Schema.org + real photos + transparent address + 24h reception note |
| Mobile CTA | Sticky bottom "Pesan" button | Sticky bottom "Pilih Kamar" | Often missing | Sticky bottom 3-button bar (WA primary, Call, Maps) |
| Language | Bilingual w/ toggle | Bilingual w/ toggle | Often Bahasa only | Bahasa primary, no toggle UI (AF-02) |
| What they over-engineer (skip) | Multi-room dynamic inventory, in-platform messaging, account creation flows | Same + insurance upsells + cross-sells | — | All of the above |
| What they get right (steal) | Sticky mobile CTA, icon grid for amenities, prominent price, photo lazy load | Photo gallery UX, FAQ accordion, "guests are saying" quote carousel | — | Mobile CTA pattern, icon grid, lazy gallery, FAQ accordion |

## Indonesian-Specific Patterns (Required by quality gate)

These are NOT generic landing-page advice — they're specific to the Indonesian budget-stay market and WA-first booking culture:

1. **WhatsApp as primary CTA channel, not "Contact" alternative.** In Western SaaS, WA is a contact option; in Indonesia, WA *is* the booking transaction. The CTA hierarchy reflects this: WA primary (large, sticky, colored), Phone secondary, Form tertiary or omitted.

2. **Rupiah formatting with dot thousands separator.** `Rp 200.000` (not `Rp 200,000` US-style; not `Rp200000` no-separator). The "/malam" suffix is the Indonesian convention; "per night" only when the segment is English-speaking. Avoid `Rp 200rb` in hero/headlines (looks colloquial); fine in body copy.

3. **"X menit dari Bandara" is the airport-proximity idiom.** Indonesian travel content uniformly uses "menit dari [bandara/stasiun/landmark]"; "X km from CGK" is technically accurate but conversion-weaker. Always provide a traffic-aware estimate range.

4. **OTA badge presence as trust laundering.** A `.my.id` TLD (zero-cost free domain) is a trust gap for Indonesian users; Agoda/Traveloka/Booking logos visibly displayed borrows their trust without integration cost. Outbound link, new tab.

5. **WA pre-fill that produces a structured message.** Owner gets a usable booking request, not "halo mas". This works because Indonesian WA culture is comfortable with templated messages; users will edit the template, not abandon it.

6. **24-jam resepsionis as anxiety reduction near airports.** Delayed flights are universal anxiety; explicit "Resepsionis 24 jam — boleh check-in larut malam" addresses it directly. Western "24/7 reception" doesn't carry the same emotional weight as it does for an Indonesian transit guest.

7. **Mobile-first really means mobile-only-tested.** ~90% audience mobile is not a tagline — it's a testing constraint. Desktop is a courtesy; design must survive on 360×640 cheap-Android viewport.

8. **Sticky bottom bar with 3 actions (WA + Call + Maps) is the standard pattern**, not floating bubble. Indonesian users have learned this from Tokopedia/Shopee/Gojek — it's the native mobile commerce vocabulary.

## WhatsApp Pre-Fill Templates (Specification)

Three intent-specific templates, all to `wa.me/6285108002536?text=<urlencoded>`:

**Hero / general "Pesan via WhatsApp"** (DF-02):
```
Halo Jo Guest House, saya mau pesan kamar.

Tanggal check-in: ___
Tanggal check-out: ___
Jumlah tamu: ___
Ada pertanyaan: ___

Terima kasih!
```

**Rooms section "Tanyakan Ketersediaan"** (DF-03):
```
Halo Jo Guest House, mau cek ketersediaan kamar untuk tanggal ___ sampai ___. Untuk ___ tamu. Terima kasih!
```

**Location section "Minta Petunjuk Arah"** (DF-03):
```
Halo Jo Guest House, saya menuju ke lokasi dari ___. Boleh minta petunjuk arah? Terima kasih!
```

Append `%0A%0ASumber: {{utm_source}}-{{utm_campaign}}` to each when UTM params present (DF-12).

## Sources

- [Panduan Lengkap Halaman Booking WhatsApp — Webside.id](https://webside.id/panduan-lengkap-halaman-booking-whatsapp-fitur-contoh-dan-cara-menggunakannya/)
- [Sistem Booking Hotel via WhatsApp — JatisMobile](https://jatismobile.com/id/blog/whatsapp-tips/sistem-booking-hotel-via-whatsapp/)
- [Cara Booking Hotel via WhatsApp — Cozzy.id](https://cozzy.id/news/cara-booking-hotel-via-whatsapp-panduan-lengkap-untuk-memudahkan-pemesanan-akomodasi)
- [Cara Membuat Link WA Me — Dealls](https://dealls.com/pengembangan-karir/membuat-link-wa-me)
- [Hotel Murah Dekat Bandara Soetta Rp 200 Ribuan — Detik](https://travel.detik.com/domestic-destination/d-8115443/10-hotel-murah-dan-nyaman-dekat-bandara-soetta-tarif-mulai-rp-200-ribuan)
- [12 Hotel Dekat Bandara Soekarno-Hatta untuk Transit — Traveloka](https://www.traveloka.com/id-id/explore/tips/10-hotel-dekat-bandara-soekarno-hatta-untuk-tempat-transit/18610)
- [21 Hotel Dekat Bandara Soekarno Hatta — Salsawisata](https://salsawisata.com/hotel-dekat-bandara-soekarno-hatta/)
- [Get to Know More About Airy Rooms, RedDoorz, ZenRooms and OYO — Alinear](https://www.alinear.id/en/read/get-to-know-more-about-airy-rooms-reddoorz-zenrooms-and-oyo)
- [Bottom Navigation Pattern On Mobile Web Pages — Smashing Magazine](https://www.smashingmagazine.com/2019/08/bottom-navigation-pattern-mobile-web-pages/)
- [How Hotels Can Regain Control Over Direct Pricing — HotelChamp](https://www.hotelchamp.com/blog/bloghow-hotels-can-regain-control-over-direct-pricing)
- [Hotel Rate Parity — SiteMinder](https://www.siteminder.com/r/hotel-rate-parity/)
- [Add a WhatsApp button to your website — Infobip](https://www.infobip.com/blog/add-whatsapp-button-to-website)
- [Astro WhatsApp component — dfrios/astro-whatsapp on GitHub](https://github.com/dfrios/astro-whatsapp)
- [Mobile Landing Pages: 8 Examples — Elementor](https://elementor.com/blog/mobile-landing-page/)

---
*Feature research for: Indonesian budget guesthouse / WA-first single-page landing*
*Researched: 2026-05-12*
