# Jo Guest House — Website Rebuild

## What This Is

A new marketing/booking landing site for **Jo Guest House**, a 17-room budget guesthouse in Cengkareng, West Jakarta (near Soekarno-Hatta International Airport, CGK). Replaces the current half-finished WordPress site at `joguesthouse.my.id` with a fast, conversion-focused Astro + Tailwind static site. Primary audience: Indonesian travelers looking for an affordable stay near the airport; secondary: English-speaking visitors who land via OTAs or ads.

## Core Value

**Turn the existing paid traffic (TikTok / Meta / Google Ads) into actual WhatsApp bookings.** Every other consideration — design, SEO, OTA integration — is in service of this. If the new site doesn't convert better than the current one, it has failed.

## Requirements

### Validated

<!-- Shipped and confirmed valuable. -->

(None yet — ship to validate)

### Active

<!-- Current scope. Building toward these. -->

- [ ] Single-page landing site in Bahasa Indonesia (primary) with English fallback on key strings
- [ ] Hero clearly communicates: budget guesthouse + proximity to Soekarno-Hatta airport + Rp 200k/night
- [ ] Primary CTA = WhatsApp (+62 851 0800 2536) — large, sticky on mobile, click-to-chat
- [ ] OTA badges/links visible: Agoda, Traveloka, Booking.com (icons + outbound links to listings)
- [ ] Rooms section: 17 rooms total, Rp 200k/night, photos + amenities
- [ ] Facilities section (Free WiFi, Cafe, Water Heater, Room Service, 24×7 Reception)
- [ ] Location section: embedded map + written address + "X min dari Bandara Soekarno-Hatta"
- [ ] Working contact form (or fallback to WA deep link)
- [ ] Correct SEO basics: real `<title>`, meta description, Open Graph image (replaces "My Blog – My WordPress Blog")
- [ ] Schema.org `LodgingBusiness` / `Hotel` JSON-LD for local SEO
- [ ] Tracking preserved: TikTok Pixel, Meta Pixel, GTM, Google Ads conversion tag (re-installed on new build)
- [ ] Mobile-first responsive (target audience overwhelmingly mobile)
- [ ] Deploy to Vercel, point `joguesthouse.my.id` at it (keep current domain)
- [ ] Choose visual direction via `/design-direction` — generate 3 options, pick one, then implement

### Out of Scope

<!-- Explicit boundaries. Includes reasoning to prevent re-adding. -->

- **On-site booking engine with real-time availability** — bookings flow through WhatsApp and OTAs; owner has no PMS/channel manager to integrate with
- **Payment processing (Midtrans / Stripe)** — same reason; WA + OTAs handle money
- **Multi-page site (separate /rooms, /about, /contact pages)** — v1 is one scrolling page to ship fast; revisit in v2 if SEO data shows demand
- **Per-room detail pages** — all 17 rooms are the same price/type per current info; not enough variance to justify
- **Full bilingual site with language toggle** — Bahasa primary, English limited to a few key strings (CTAs, OTA badges); foreign tourists mostly come from OTAs anyway
- **Blog / content marketing** — not the leverage point for a 17-room budget guesthouse
- **Custom CMS / WordPress migration** — static Astro site, owner edits content via PR / git or asks us; no editorial workflow needed
- **Reviews scraping/embedding from OTAs** — out of scope for v1; static testimonial section instead if needed
- **New domain (`.com` / `.id`)** — owner keeps `joguesthouse.my.id` (decision logged; trust risk to be mitigated by site quality)

## Context

**Current site (`joguesthouse.my.id`) — what we're replacing:**

- WordPress 6.9.4 + Astra theme + Elementor + WPForms (id 336)
- Default title: `<title>My Blog – My WordPress Blog</title>` — never customized
- No meta description, no Open Graph tags
- Navigation = WordPress default ("My Blog", "Sample Page")
- Production content includes Lorem Ipsum and placeholder headings ("A Title to Turn the Visitor Into a Lead", "Add Your Heading Text Here")
- Room prices listed in **USD** ($125 / $255 / $375 / $425 per night) — wrong by ~10×; real price is Rp 200k (~$13 USD)
- 40+ images present, generally usable for content
- WhatsApp contact: `https://wa.me/6285108002536`
- **Active marketing pixels:** TikTok Pixel (`D2MR5GRC77U4PA826B90`), Meta Pixel (`770014465395193`), Google Tag Manager (`GTM-MK4WJPMF`), Google Ads conversion (`AW-17438288457`) — owner is spending on paid acquisition that's hitting a broken page

**Business context:**

- Address: Ruko Plaza de Lumina, Jl. Outer Ring Road No.7, RT.3/RW.7, Duri Kosambi, Kecamatan Cengkareng, Kota Jakarta Barat, DKI Jakarta 11750
- Located in a "ruko" (rumah toko / shop-house) commercial complex, near Jakarta Outer Ring Road (JORR), within striking distance of CGK airport
- 17 rooms, single price tier: Rp 200,000 / night
- Slogan: *"Hotel Nyaman Gak Harus Bintang Lima"* (A comfortable hotel doesn't have to be five-star)
- Primary booking channel: WhatsApp; also listed on Agoda, Traveloka, Booking.com (links/badges to be added)

**Builder context:**

- Pavelec Studio (Michal Pavelec) — Astro + Tailwind is the established stack
- Deploy via Vercel (free tier acceptable for static landing)
- Design exploration via `/design-direction` skill before implementation

## Constraints

- **Tech stack**: Astro + Tailwind v4 + GSAP (light, optional) — matches Pavelec Studio toolchain
- **Timeline**: ASAP — the longer paid traffic hits the current site, the more money burns
- **Budget**: Minimal external services; Vercel free tier, no paid SaaS
- **Domain**: Keep `joguesthouse.my.id` (free `.my.id` zone; site quality must compensate for low-trust TLD)
- **Language**: Bahasa Indonesia primary; English limited to key UI strings
- **Mobile-first**: ~90 % of Indonesian web traffic is mobile — desktop is a courtesy, not primary
- **Tracking parity**: All current ad pixels (TikTok, Meta, GTM, Google Ads conversion) must work on the new site from day one — ad campaigns can't go dark
- **No content backlog**: Owner has not delivered fresh photos, copy, or brand assets ("we don't need it now") — v1 ships using existing WP-site assets + placeholders where needed; refresh in a later iteration

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Rebuild on Astro + Tailwind (not patch WP) | Current WP install is half-finished, slow, and locked into Elementor; a static rebuild is faster end-to-end and matches builder's stack | — Pending |
| Single-page landing for v1 | 17 identical rooms + flat price = no information density that demands sub-pages; ships faster | — Pending |
| WhatsApp-first booking (no on-site engine) | Owner already books via WA; no PMS to integrate with; Indonesian market is WA-native | — Pending |
| Bahasa primary, English thin fallback | Owner's directive; matches actual audience split (locals via ads, foreigners via OTAs) | — Pending |
| Keep `joguesthouse.my.id` domain | Owner's directive; avoid disruption to current ad/SEO history | ⚠️ Revisit (trust risk on `.my.id` TLD) |
| Generate 3 design directions before implementing | Owner is undecided on visual direction; `/design-direction` reduces redesign cost | — Pending |
| Re-implement all tracking pixels (TikTok, Meta, GTM, Google Ads) on new site | Active ad spend must not go dark on cutover | — Pending |
| OTA badges (Agoda, Traveloka, Booking) link out, not embed | Lower complexity, no API integrations, faster ship | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-05-12 after initialization*
