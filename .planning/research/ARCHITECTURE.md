# Architecture Research — Jo Guest House

**Domain:** Static one-page Astro + Tailwind landing site (WhatsApp-first conversion, marketing pixels, OTA outbound, Vercel deploy)
**Researched:** 2026-05-12
**Confidence:** HIGH (overall — patterns verified against STACK.md and Astro 6 conventions)

---

## Standard Architecture

### System Overview

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           BROWSER (Mobile-first)                          │
│                                                                            │
│  ┌──────────────┐  ┌────────────────┐  ┌──────────────────────────────┐  │
│  │  Static HTML │  │  Inline <script>│  │  Third-party (lazy)          │  │
│  │  + CSS + img │  │  - GTM bootstrap│  │  - GTM container → TT/Meta/  │  │
│  │  (Astro out) │  │  - track.ts util│  │    Ads via dataLayer.push    │  │
│  │              │  │  - wa-cta.ts    │  │  - Google Maps iframe        │  │
│  │              │  │  - mobile-nav.ts│  │  - Web3Forms POST endpoint   │  │
│  │              │  │  - lightbox.ts  │  │  - wa.me deep-link           │  │
│  └──────┬───────┘  └────────┬───────┘  └────────────┬─────────────────┘  │
│         │                   │                       │                      │
│         └───────────────────┴───────────────────────┘                      │
│                              │                                             │
└──────────────────────────────┼─────────────────────────────────────────────┘
                               │ (HTTPS)
┌──────────────────────────────▼─────────────────────────────────────────────┐
│                    VERCEL EDGE CDN (static hosting)                         │
│   /                index.html (single page)                                  │
│   /_astro/*        fingerprinted JS/CSS/images (immutable cache)             │
│   /og-image.jpg    OpenGraph card                                            │
│   /robots.txt      crawl directives                                          │
│   /sitemap-index.xml from @astrojs/sitemap                                   │
└─────────────────────────────────────────────────────────────────────────────┘
                               ▲
                               │ (git push → auto-deploy)
┌──────────────────────────────┴─────────────────────────────────────────────┐
│                          GITHUB REPO  →  Vercel build                       │
│                                                                              │
│  src/pages/index.astro                                                       │
│      └─ uses → src/layouts/BaseLayout.astro                                  │
│             └─ slots → <SeoHead/> + <GtmHead/> + page content + <GtmBody/>   │
│      └─ composes sections from src/components/sections/*                     │
│             └─ pulls strings from src/content/site.ts (typed)                │
│             └─ pulls photos from src/assets/photos/* (astro:assets)          │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

| Layer | Responsibility | Implementation |
|-------|----------------|----------------|
| **Page** | One entry route, scroll-driven section composition | `src/pages/index.astro` only |
| **Layout** | `<html>`, `<head>`, GTM, SEO, fonts, body wrapper | `src/layouts/BaseLayout.astro` |
| **Section components** | One DOM region of the landing page; fully self-contained markup + per-section copy slot | `src/components/sections/*.astro` |
| **UI primitives** | Reusable atoms used by sections (button, image card, icon) | `src/components/ui/*.astro` |
| **Client islands** | Tiny vanilla TS modules hydrated for one specific interaction | `src/scripts/*.ts` referenced via `<script>` blocks inside the Astro component that owns the behaviour |
| **Content** | All user-facing strings, room data, OTA URLs, contact info — typed | `src/content/site.ts`, `src/content/rooms.ts`, `src/content/otas.ts` |
| **Lib** | i18n helper, tracking helper, WA URL builder, schema-builder | `src/lib/*.ts` |
| **Assets** | Source photos, OG image source, favicon source | `src/assets/photos/*`, `public/og-image.jpg` |

---

## Recommended Project Structure (v1, concrete)

```
jo-guesthouse/
├── astro.config.mjs              # site, output:'static', vercel adapter, sitemap, icon
├── tsconfig.json                 # strict
├── package.json
├── vercel.json                   # cache headers for /_astro/*, /redirects if needed
├── .env.example                  # PUBLIC_WEB3FORMS_KEY=...
├── public/
│   ├── favicon.svg
│   ├── og-image.jpg              # 1200×630, pre-baked (NOT astro:assets — must be at fixed URL for OG crawlers)
│   ├── robots.txt                # 4 lines, hand-written
│   └── apple-touch-icon.png
├── src/
│   ├── pages/
│   │   └── index.astro           # THE page. Composes sections in order. ~120 lines max.
│   ├── layouts/
│   │   └── BaseLayout.astro      # <html>, <head>, GTM bootstrap, fonts, <body> + <slot/>
│   ├── components/
│   │   ├── seo/
│   │   │   ├── SeoHead.astro     # <title>, <meta>, og:*, twitter:*, canonical, hreflang(future)
│   │   │   └── JsonLd.astro      # LodgingBusiness schema (props: data — built via lib/schema.ts)
│   │   ├── tracking/
│   │   │   ├── GtmHead.astro     # <script is:inline> GTM bootstrap
│   │   │   └── GtmBody.astro     # <noscript><iframe> fallback
│   │   ├── sections/             # one component per page section, in render order
│   │   │   ├── Header.astro      # logo + mobile burger; thin, sticky on scroll
│   │   │   ├── Hero.astro        # H1, sub-tagline, hero photo (Image priority), primary CTA
│   │   │   ├── TrustBar.astro    # "17 kamar • Rp 200rb/malam • 10 min ke CGK" — 3 KPI strip
│   │   │   ├── About.astro       # 2-3 paragraph "Tentang Jo Guest House"
│   │   │   ├── Rooms.astro       # CSS scroll-snap gallery of RoomCard
│   │   │   ├── Facilities.astro  # FacilityGrid of FacilityItem (icon + label)
│   │   │   ├── Location.astro    # address text + lazy Google Maps iframe + "X min dari CGK"
│   │   │   ├── OtaBadges.astro   # Agoda / Traveloka / Booking outbound links + logos
│   │   │   ├── Enquiry.astro     # Web3Forms POST + "Lebih cepat via WhatsApp →" link above
│   │   │   ├── Faq.astro         # 4-6 Q&A (Bahasa) — also fed into FAQPage JSON-LD
│   │   │   └── Footer.astro      # address, phone, OTA links repeat, copyright
│   │   ├── ui/                   # primitives — used by sections
│   │   │   ├── WhatsAppButton.astro   # the ONE component every WA CTA uses
│   │   │   ├── StickyWaCta.astro      # mobile-only fixed bottom bar; uses WhatsAppButton
│   │   │   ├── RoomCard.astro         # one photo + caption (rooms are identical, but card pattern)
│   │   │   ├── FacilityItem.astro     # icon + Bahasa label
│   │   │   ├── OtaBadge.astro         # logo + outbound link, tracked
│   │   │   ├── Container.astro        # max-w + px wrapper used by every section
│   │   │   └── SectionHeading.astro   # eyebrow + h2 pair used by every section
│   │   └── icons/
│   │       └── (astro-icon resolves from @iconify-json/* — no files needed)
│   ├── content/                  # all strings + structured data (typed, NOT Content Collections)
│   │   ├── site.ts               # brand, contact, slogan, navigation labels, hero copy, etc.
│   │   ├── rooms.ts              # array of {id, name, photo, amenities[]} — 1 entry for v1, ready to scale
│   │   ├── facilities.ts         # array of {iconName, labelId, labelEn}
│   │   ├── otas.ts               # array of {name, url, logoIcon, label}
│   │   ├── faq.ts                # array of {q: {id,en}, a: {id,en}}
│   │   └── strings.ts            # the ~30 short UI strings, keyed (the t() dictionary)
│   ├── lib/                      # pure TS, no Astro
│   │   ├── i18n.ts               # Lang type + t() helper + strings re-export
│   │   ├── wa.ts                 # buildWaUrl(messageKey, utms) → string
│   │   ├── track.ts              # window.dataLayer.push wrappers: trackWaClick(), trackOtaClick(), trackEnquirySubmit()
│   │   ├── schema.ts             # buildLodgingSchema() → object; buildFaqSchema()
│   │   └── utm.ts                # parseUtmsFromLocation() → record (client-only)
│   ├── scripts/                  # client-side islands, vanilla TS, one concern each
│   │   ├── wa-cta.ts             # delegated click handler for [data-wa-cta]; calls trackWaClick() + buildWaUrl(); window.open
│   │   ├── mobile-nav.ts         # burger toggle, scroll-lock
│   │   ├── sticky-cta.ts         # show sticky bar after hero leaves viewport (IntersectionObserver)
│   │   ├── gallery.ts            # optional lightbox for room photos (kept tiny; can defer to v1.1)
│   │   └── enquiry.ts            # form fetch + toast + trackEnquirySubmit()
│   ├── styles/
│   │   └── global.css            # @import "tailwindcss"; @theme tokens; @fontsource imports
│   └── assets/
│       └── photos/               # 40 source photos, processed by astro:assets
│           ├── hero.jpg          # 1920×1080 source → outputs AVIF/WebP responsive
│           ├── rooms/
│           │   ├── 01.jpg
│           │   └── ...
│           ├── facilities/
│           └── building/
└── .planning/
    └── ...
```

### Structure Rationale (non-obvious choices)

- **`src/content/*.ts` (typed TS modules) — NOT Astro Content Collections, NOT MDX, NOT inline.** Content Collections shine for blog/article workloads with frontmatter + many files; this site has ~30 strings + 1 room entry + 3 OTAs + 6 FAQ items, all of which fit in five typed `.ts` files. Inline strings in `.astro` would scatter copy across 11 components, making owner-or-builder edits (and the eventual EN fallback) a grep-and-pray exercise. Single source of truth in `src/content/` keeps copy edits to one folder. The "as const" object also gives the i18n `t()` helper compile-time key checking.
- **Sections live in `components/sections/`, primitives in `components/ui/`.** Sections are render-once (each appears in one place on the page). Primitives (`WhatsAppButton`, `RoomCard`, `SectionHeading`, `Container`) are used 3-15 times each. Splitting them prevents "is `Hero.astro` a section or a primitive?" debates and makes the import graph readable.
- **`src/scripts/*.ts` separate from `.astro` files.** Each interaction (WA click, mobile nav, sticky CTA, form, lightbox) is one tiny vanilla TS module. They are imported via `<script>` inside the component that owns the behavior. Keeps `.astro` files small and lets `astro check` type-check the TS independently.
- **`public/og-image.jpg` lives in `public/`, NOT `src/assets/`.** OG crawlers (Facebook, WhatsApp, Twitter) hit a fixed URL and don't follow Astro's fingerprinted asset names. Photos used in-page go through `astro:assets` for optimization; the OG image is hand-prepared at 1200×630 once and served at a stable URL.
- **`src/lib/track.ts` is the only place that touches `window.dataLayer`.** No component calls `dataLayer.push` directly. This means changing pixel routing later (e.g., swapping GTM for direct gtag) edits one file.
- **No `.env` for `GTM_ID` etc.** GTM container ID, Meta pixel ID, WhatsApp number are all NOT secrets — they're in `src/content/site.ts` so the owner can read/edit them in one place. Only `PUBLIC_WEB3FORMS_KEY` lives in env (it's a per-deploy access key, not a secret per se, but treating it as build-env is the right shape).

---

## Component Inventory

### Sections (11) — render order on `index.astro`

| # | Component | Responsibility | Reads from |
|---|-----------|----------------|------------|
| 1 | `Header.astro` | Logo + nav anchor links + mobile burger | `site.ts` (nav labels) |
| 2 | `Hero.astro` | H1, slogan, hero photo (LCP), primary `<WhatsAppButton>` | `site.ts`, `assets/photos/hero.jpg` |
| 3 | `TrustBar.astro` | 3 KPI strip ("17 kamar", "Rp 200rb/malam", "10 min ke CGK") | `site.ts` |
| 4 | `About.astro` | "Tentang" paragraph + 1 photo | `site.ts`, `assets/photos/building/*` |
| 5 | `Rooms.astro` | CSS scroll-snap row of `<RoomCard>` × N | `rooms.ts` |
| 6 | `Facilities.astro` | 5-cell grid of `<FacilityItem>` (WiFi, Cafe, Water Heater, Room Service, 24/7) | `facilities.ts` |
| 7 | `Location.astro` | Address text + "X min dari CGK" + lazy Google Maps iframe | `site.ts` |
| 8 | `OtaBadges.astro` | Agoda + Traveloka + Booking outbound logos | `otas.ts` |
| 9 | `Enquiry.astro` | Form + "Lebih cepat via WA →" link above | `site.ts`, env `PUBLIC_WEB3FORMS_KEY` |
| 10 | `Faq.astro` | 4-6 Q&A — also generates `FAQPage` JSON-LD | `faq.ts` |
| 11 | `Footer.astro` | Address, phone, OTA links, copyright | `site.ts`, `otas.ts` |

### UI primitives (7)

| Component | Responsibility | Used by |
|-----------|----------------|---------|
| `WhatsAppButton.astro` | Renders `<a data-wa-cta data-message-key="..." data-section="...">` with WA styling. Server-renders an href fallback (`wa.me/...` with default message) for no-JS; client script enriches with UTMs + tracking. | Hero, StickyWaCta, Rooms, Enquiry, Footer |
| `StickyWaCta.astro` | Fixed bottom bar on `md:hidden`; wraps `WhatsAppButton`. Hidden until hero leaves viewport (via `sticky-cta.ts`). | `index.astro` (renders once at page level) |
| `RoomCard.astro` | One photo (via `<Picture>`) + caption + amenity icons | Rooms |
| `FacilityItem.astro` | astro-icon SVG + label | Facilities |
| `OtaBadge.astro` | astro-icon logo + outbound `<a target="_blank" data-ota-badge>` (tracked) | OtaBadges, Footer |
| `Container.astro` | `max-w-6xl mx-auto px-4 md:px-6` wrapper | Every section |
| `SectionHeading.astro` | Eyebrow text + `<h2>` pair | Every section except Hero, TrustBar, Header, Footer |

### Infrastructure components (4)

| Component | Responsibility | Slotted in |
|-----------|----------------|------------|
| `BaseLayout.astro` | `<html lang="id">`, `<head>`, body wrapper, accepts `<slot/>` for page content | `index.astro` |
| `SeoHead.astro` | `<title>`, meta description, canonical, OG, Twitter card | `BaseLayout` (inside `<head>`) |
| `JsonLd.astro` | `<script type="application/ld+json" is:inline set:html={JSON.stringify(data)}>` — props: `data` | `BaseLayout` (for LodgingBusiness) + `Faq.astro` (for FAQPage) |
| `GtmHead.astro` / `GtmBody.astro` | GTM bootstrap (head) + noscript iframe (first thing in body) | `BaseLayout` |

**Total component count for v1: ~22 files.** This is the right size — every file has one clear job, no file is over ~120 lines.

---

## Architectural Patterns

### Pattern 1: Single source of truth content modules

**What:** All strings, room data, OTA URLs, contact info live in `src/content/*.ts` as typed `as const` objects. Components import what they need. No copy in `.astro` markup except section-level connective tissue.

**When to use:** Always, for this project. Site has finite copy and one editor (the builder).

**Trade-offs:**
- (+) Owner-or-builder edits all copy in one folder; future EN translation = one PR
- (+) TypeScript catches typos in lookup keys at build time
- (-) Slightly more indirection than inline strings; first edit feels alien

**Example:**
```typescript
// src/content/site.ts
export const site = {
  brand: 'Jo Guest House',
  slogan: { id: 'Hotel Nyaman Gak Harus Bintang Lima', en: 'Comfortable, no five-star needed' },
  whatsapp: { number: '6285108002536', defaultMessageId: 'enquiry' },
  contact: { phoneDisplay: '+62 851 0800 2536', email: '...' },
  address: {
    street: 'Ruko Plaza de Lumina, Jl. Outer Ring Road No.7',
    locality: 'Cengkareng', region: 'DKI Jakarta', postalCode: '11750',
  },
  geo: { lat: -6.165, lng: 106.738 },
  airportMinutes: 10,
  pricePerNightIdr: 200_000,
  roomCount: 17,
} as const;
```

```astro
---
// src/components/sections/Hero.astro
import { site } from '../../content/site';
import { t } from '../../lib/i18n';
---
<h1>{site.brand}</h1>
<p>{site.slogan.id}</p>
<WhatsAppButton messageKey="hero" section="hero" />
```

### Pattern 2: i18n via t() dictionary, no routing

**What:** A tiny `t(key, lang='id')` helper reads from a typed `strings` object. v1 renders in Bahasa only; three or four elements opt into EN by reading `.en`. No `/en/` route, no language switcher.

**When to use:** Single-page sites with <50 strings and asymmetric language priority. Upgrade to Astro's built-in `i18n` config + `/en/` route once analytics shows >25% EN organic traffic (per STACK.md decision point).

**Trade-offs:**
- (+) Zero extra build output, zero SEO fragmentation
- (+) Trivial to extend: add `.en` value to the object, render it
- (-) Not suitable if EN demand grows substantially; migration path is documented

**Example:**
```typescript
// src/lib/i18n.ts
import { strings } from '../content/strings';

export type Lang = 'id' | 'en';

export const t = <K extends keyof typeof strings>(
  key: K,
  lang: Lang = 'id'
): string => strings[key][lang] ?? strings[key].id;
```

```typescript
// src/content/strings.ts
export const strings = {
  bookNow:      { id: 'Pesan Sekarang',  en: 'Book Now' },
  viewRooms:    { id: 'Lihat Kamar',     en: 'View Rooms' },
  rooms:        { id: 'Kamar',           en: 'Rooms' },
  facilities:   { id: 'Fasilitas',       en: 'Facilities' },
  location:     { id: 'Lokasi',          en: 'Location' },
  contactUs:    { id: 'Hubungi Kami',    en: 'Contact Us' },
  fasterViaWa:  { id: 'Lebih cepat via WhatsApp',
                  en: 'Faster via WhatsApp' },
  // ... ~25 more
} as const;
```

### Pattern 3: Centralised tracking via dataLayer (no per-pixel SDK)

**What:** All pixel-relevant user actions push to `window.dataLayer`. GTM (single script tag) routes events to TikTok, Meta, Google Ads, and any pixel added later. Components never touch `fbq`, `ttq`, or `gtag` directly.

**When to use:** Always — STACK.md decision. The owner manages pixels in GTM UI without code changes.

**Trade-offs:**
- (+) Owner self-serves pixel changes; no redeploys for marketing tweaks
- (+) One script tag, not four
- (-) Owner must learn GTM UI (already does — current site uses GTM)

**Example:**
```typescript
// src/lib/track.ts
type DataLayerEvent = Record<string, unknown> & { event: string };

declare global {
  interface Window { dataLayer: DataLayerEvent[] }
}

const push = (e: DataLayerEvent): void => {
  window.dataLayer = window.dataLayer || [];
  window.dataLayer.push(e);
};

export const trackWaClick = (section: string, messageKey: string): void =>
  push({ event: 'whatsapp_click', section, messageKey });

export const trackOtaClick = (ota: string): void =>
  push({ event: 'ota_click', ota });

export const trackEnquirySubmit = (): void =>
  push({ event: 'enquiry_submit' });
```

```typescript
// src/scripts/wa-cta.ts
import { trackWaClick } from '../lib/track';
import { buildWaUrl } from '../lib/wa';
import { parseUtmsFromLocation } from '../lib/utm';

document.addEventListener('click', (e) => {
  const target = (e.target as HTMLElement).closest<HTMLAnchorElement>('[data-wa-cta]');
  if (!target) return;
  e.preventDefault();
  const section = target.dataset.section ?? 'unknown';
  const messageKey = target.dataset.messageKey ?? 'default';
  const utms = parseUtmsFromLocation();
  trackWaClick(section, messageKey);
  window.open(buildWaUrl(messageKey, utms), '_blank', 'noopener');
});
```

### Pattern 4: WhatsApp URL builder with UTM passthrough

**What:** Every WhatsApp CTA renders an `<a>` with a default `href` (server-side, for no-JS users) and is enriched on click with current page UTMs baked into the pre-filled message. One builder function in `src/lib/wa.ts`.

**When to use:** Always — STACK.md requirement that paid-traffic attribution survive the WA jump.

**Trade-offs:**
- (+) Even with JS off, the link works (graceful degradation)
- (+) UTM survives WA jump → owner sees "this WhatsApp lead came from TikTok campaign X"
- (-) The pre-filled message gets longer; trim to <120 chars to avoid WA truncation

**Example:**
```typescript
// src/lib/wa.ts
import { site } from '../content/site';

const messages: Record<string, string> = {
  default:  'Halo, saya tertarik dengan Jo Guest House. Apakah masih ada kamar tersedia?',
  hero:     'Halo, saya melihat website Jo Guest House. Mau tanya soal kamar Rp 200rb.',
  rooms:    'Halo, saya tertarik booking kamar Rp 200rb di Jo Guest House.',
  enquiry:  'Halo, saya ingin enquiry kamar di Jo Guest House.',
  footer:   'Halo, saya tertarik dengan Jo Guest House.',
};

export const buildWaUrl = (
  messageKey: string,
  utms: Record<string, string> = {}
): string => {
  const base = messages[messageKey] ?? messages.default;
  const utmSuffix = Object.entries(utms)
    .filter(([k]) => k.startsWith('utm_'))
    .map(([k, v]) => `${k}=${v}`)
    .join(' | ');
  const text = utmSuffix ? `${base}\n\n(${utmSuffix})` : base;
  return `https://wa.me/${site.whatsapp.number}?text=${encodeURIComponent(text)}`;
};
```

### Pattern 5: Islands-by-script-tag (not framework islands)

**What:** For five small interactions (WA click, mobile nav, sticky CTA, form submit, optional lightbox), use plain `<script>` blocks at the component level, each importing one vanilla TS file from `src/scripts/`. No React/Vue/Alpine. Astro's `client:*` directives are not needed because there are no framework components.

**When to use:** When all interactivity is delegated DOM event handling (clicks, IntersectionObserver, form submit) totaling <2 KB of JS. Upgrade to a framework island only if a section needs reactive state (e.g., a real availability widget).

**Trade-offs:**
- (+) Near-zero JS budget — likely <5 KB total minified
- (+) No hydration cost, no framework runtime
- (-) Manual DOM querying instead of declarative components — fine at this scale, painful at 10×

**Example:**
```astro
---
// src/components/sections/Header.astro — owns mobile nav behavior
---
<nav>
  <button data-mobile-nav-toggle aria-label="Menu">☰</button>
  <ul data-mobile-nav-menu hidden>
    <li><a href="#kamar">Kamar</a></li>
    ...
  </ul>
</nav>

<script>
  import '../../scripts/mobile-nav';
</script>
```

```typescript
// src/scripts/mobile-nav.ts
const toggle = document.querySelector<HTMLButtonElement>('[data-mobile-nav-toggle]');
const menu = document.querySelector<HTMLElement>('[data-mobile-nav-menu]');
toggle?.addEventListener('click', () => {
  const isOpen = menu?.hasAttribute('hidden') === false;
  if (isOpen) menu?.setAttribute('hidden', '');
  else menu?.removeAttribute('hidden');
});
```

### Pattern 6: SEO head — one component, slot for JSON-LD

**What:** `<SeoHead>` is a single Astro component that renders all `<meta>` tags, OG, Twitter card, and canonical, taking props with sensible defaults from `site.ts`. `<JsonLd>` is a sibling component that takes a `data` prop and renders `<script type="application/ld+json">`. `BaseLayout` slots `<SeoHead>` and `<JsonLd data={lodgingSchema}>` into `<head>`; the FAQ section emits a second `<JsonLd>` into a head slot.

**When to use:** Always — STACK.md requires real title, meta description, OG image, and `LodgingBusiness` schema.

**Trade-offs:**
- (+) All SEO surface in two components — easy to audit
- (+) JSON-LD is a separate component, so the FAQ section can emit its own without duplicating logic
- (-) Requires Astro's head-injection slot pattern (use `<slot name="head"/>` in BaseLayout)

**Example:**
```astro
---
// src/layouts/BaseLayout.astro
import SeoHead from '../components/seo/SeoHead.astro';
import JsonLd from '../components/seo/JsonLd.astro';
import GtmHead from '../components/tracking/GtmHead.astro';
import GtmBody from '../components/tracking/GtmBody.astro';
import { buildLodgingSchema } from '../lib/schema';
import '../styles/global.css';

interface Props { title?: string; description?: string; }
const { title, description } = Astro.props;
---
<!DOCTYPE html>
<html lang="id">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <GtmHead />
    <SeoHead title={title} description={description} />
    <JsonLd data={buildLodgingSchema()} />
    <slot name="head" />
  </head>
  <body>
    <GtmBody />
    <slot />
  </body>
</html>
```

### Pattern 7: Image pipeline — `src/assets/` for photos, `public/` for OG only

**What:** All 40 photos live in `src/assets/photos/` and are rendered via `<Image>` / `<Picture>` from `astro:assets`. Astro generates AVIF + WebP + responsive `srcset` at build time. The OG image is pre-baked at 1200×630 and placed in `public/og-image.jpg` so it sits at a stable URL for crawlers.

**When to use:** Always. Decision is documented in STACK.md and enforced by directory choice.

**Trade-offs:**
- (+) Best-in-class LCP for Indonesian mobile (AVIF support is 100% on Chrome Android)
- (+) Build-time generation — no runtime image service costs
- (-) Build is slower (~30s for 40 photos via Sharp); acceptable

**Example:**
```astro
---
// src/components/sections/Hero.astro
import { Image } from 'astro:assets';
import heroPhoto from '../../assets/photos/hero.jpg';
---
<Image
  src={heroPhoto}
  alt="Jo Guest House — kamar nyaman dekat Bandara Soekarno-Hatta"
  width={1600}
  height={900}
  loading="eager"
  fetchpriority="high"
  sizes="(max-width: 768px) 100vw, 1600px"
/>
```

Naming convention: kebab-case, descriptive, no spaces. `hero.jpg`, `rooms/standard-01.jpg`, `facilities/cafe-interior.jpg`, `building/exterior-front.jpg`.

---

## Data Flow

### Page render (build time)

```
git push → Vercel build
    ↓
astro build
    ↓
src/pages/index.astro
    ↓
imports BaseLayout + sections + content/*.ts + assets/photos/*
    ↓
Astro compiles to static HTML, processes images via Sharp (AVIF/WebP/srcset)
    ↓
emits: dist/index.html, dist/_astro/*.{js,css,avif,webp}, dist/sitemap-index.xml
    ↓
Vercel serves from edge CDN
```

### WhatsApp click (the critical path)

```
User taps any WA button (Hero / StickyMobile / Rooms / Enquiry / Footer)
    │
    ▼
Browser fires click on <a data-wa-cta data-message-key="..." data-section="...">
    │
    ▼
Delegated handler in src/scripts/wa-cta.ts
    │ (1) e.preventDefault()
    │ (2) parseUtmsFromLocation() → reads ?utm_source=tiktok&utm_campaign=... from URL
    │ (3) trackWaClick(section, messageKey) → window.dataLayer.push({event: 'whatsapp_click', ...})
    │     └──► GTM picks up 'whatsapp_click' event
    │           ├──► fbq('track', 'Contact')         (Meta pixel)
    │           ├──► ttq.track('Contact')            (TikTok pixel)
    │           └──► gtag('event', 'conversion', {send_to: 'AW-17438288457/...'})  (Google Ads)
    │ (4) buildWaUrl(messageKey, utms) → https://wa.me/6285108002536?text=...
    │ (5) window.open(url, '_blank', 'noopener')
    ▼
WhatsApp opens (app on mobile, web on desktop) with pre-filled message
```

### Form submission

```
User submits #enquiry form
    ↓
src/scripts/enquiry.ts intercepts submit
    ↓
fetch('https://api.web3forms.com/submit', { method: 'POST', body: FormData })
    ↓
Web3Forms emails owner → returns { success: true }
    ↓
on success:
    ├─► trackEnquirySubmit() → dataLayer.push({event: 'enquiry_submit'})
    │     └──► GTM fires Meta/TikTok 'Lead' + Google Ads conversion
    └─► render toast "Pesan terkirim — kami akan balas via WhatsApp"
```

### OTA badge click

```
User taps Agoda / Traveloka / Booking badge
    ↓
Delegated handler in wa-cta.ts (same file, separate listener) or a tiny ota.ts
    ↓
trackOtaClick(otaName) → dataLayer.push({event: 'ota_click', ota: 'agoda'})
    ↓
Default <a target="_blank"> behavior continues (no preventDefault)
    ↓
OTA opens in new tab
```

### Content edits (owner / builder workflow)

```
Owner emails builder: "change slogan to X"
    ↓
Builder edits src/content/site.ts
    ↓
git commit + push
    ↓
Vercel auto-deploys (~30s build)
    ↓
Live
```

For copy-only changes, only the content file is touched — no component edits, no JSON-LD edits (the schema builder reads from `site.ts`).

---

## Tracking Architecture (the critical part)

```
                    src/lib/track.ts
                    (single dataLayer wrapper)
                          │
       ┌──────────────────┼──────────────────┬────────────────────┐
       │                  │                  │                    │
trackWaClick()    trackOtaClick()    trackEnquirySubmit()    (future events)
       │                  │                  │                    │
       └──────────────────┴──────────────────┴────────────────────┘
                          │
                  window.dataLayer.push({event, ...payload})
                          │
                          ▼
                  ┌──────────────────────┐
                  │  GTM container       │
                  │  GTM-MK4WJPMF        │
                  │  (loaded once by     │
                  │   <GtmHead/>)        │
                  └──────────┬───────────┘
                             │ (fans out via GTM rules)
                  ┌──────────┼──────────┬─────────────────┬───────────────┐
                  ▼          ▼          ▼                 ▼               ▼
            Meta Pixel  TikTok Pixel  Google Ads     Google Analytics  (future tags)
            770014465.. D2MR5GRC..    AW-17438288.. (if added)
```

**Physical locations:**

| Code | Lives in | Loaded |
|------|----------|--------|
| GTM bootstrap snippet | `src/components/tracking/GtmHead.astro` | Inside `<head>` of every page, `is:inline`, no async needed because GTM is itself async |
| GTM noscript fallback | `src/components/tracking/GtmBody.astro` | First child of `<body>` |
| `window.dataLayer.push` calls | `src/lib/track.ts` (only place) | Imported by `src/scripts/*.ts` |
| Pixel IDs (Meta, TikTok, Google Ads) | **In GTM dashboard, NOT in code** | (Already configured — STACK.md confirms) |
| WA click handler | `src/scripts/wa-cta.ts` | Loaded by `<script>` in `WhatsAppButton.astro` (deduped by Astro since the script import is identical) |
| Form submit handler | `src/scripts/enquiry.ts` | Loaded by `<script>` in `Enquiry.astro` |
| OTA click handler | `src/scripts/wa-cta.ts` (same file, separate listener) or `src/scripts/ota.ts` | Loaded by `<script>` in `OtaBadge.astro` |

**Critical rule:** the only file that imports/uses `window.dataLayer` is `src/lib/track.ts`. Every other file calls the exported helpers. This makes the pixel-routing decision (GTM today, direct gtag tomorrow, server-side GA4 next year) a one-file change.

---

## SEO Architecture

| Surface | Implementation | Notes |
|---------|----------------|-------|
| `<title>` | `<SeoHead>` component, prop with default from `site.ts` | "Jo Guest House — Hotel Murah Dekat Bandara Soekarno-Hatta" |
| Meta description | Same | 150-160 chars, Bahasa, mentions Rp 200k + CGK + WhatsApp |
| Canonical | `<SeoHead>` reads `Astro.url.pathname` and emits `<link rel="canonical">` | Always `https://joguesthouse.my.id/` for v1 |
| OG image | `public/og-image.jpg` (1200×630, pre-baked) | Stable URL for FB/WA crawlers |
| `LodgingBusiness` JSON-LD | `<JsonLd>` in `BaseLayout`, data from `lib/schema.ts → buildLodgingSchema()` reading `site.ts` | One blob |
| `FAQPage` JSON-LD | `<JsonLd>` slotted into head from `Faq.astro` | Auto-generated from `faq.ts` |
| `sitemap-index.xml` | `@astrojs/sitemap` integration | One URL for v1 |
| `robots.txt` | `public/robots.txt`, hand-written, 4 lines | `User-agent: *  Allow: /  Sitemap: https://joguesthouse.my.id/sitemap-index.xml` |
| `hreflang` | Not for v1 (no `/en/` route) | Added when EN route arrives |

---

## Routing / URL Surface

For v1:

| URL | Source | Purpose |
|-----|--------|---------|
| `/` | `src/pages/index.astro` | The page |
| `/sitemap-index.xml` | `@astrojs/sitemap` | Search Console |
| `/sitemap-0.xml` | Same | Indexed by sitemap-index |
| `/robots.txt` | `public/robots.txt` | Crawler directives |
| `/og-image.jpg` | `public/og-image.jpg` | OG/Twitter/WA crawlers |
| `/favicon.svg`, `/apple-touch-icon.png` | `public/` | Browser chrome |
| `/_astro/*` | Astro build output | Fingerprinted assets (immutable cache) |

Recommendation: **only `/` for v1.** No `/en/` route. No `/rooms` page. No `/contact` page. Section anchors (`#kamar`, `#fasilitas`, `#lokasi`, `#hubungi`) carry navigation. This keeps the SEO surface focused (one URL = all authority concentrated) and matches the STACK.md "no i18n routing" decision.

---

## Deploy + DNS Architecture

```
GitHub repo (main branch)
    │
    │ (Vercel GitHub integration: every push to main → production deploy)
    ▼
Vercel project: jo-guesthouse
    ├── Production environment       → joguesthouse.my.id  (after cutover)
    │                                → jo-guesthouse.vercel.app (always)
    ├── Preview environments         → <branch>-jo-guesthouse-<hash>.vercel.app
    │                                  (every non-main branch + every PR)
    └── Env vars
        └── PUBLIC_WEB3FORMS_KEY     (build-time, Production + Preview)

Domain: joguesthouse.my.id (PANDI .my.id registrar, currently pointing at WP host)
    │
    │ Cutover: change DNS at registrar → CNAME (or A records) to Vercel
    ▼
Vercel verifies domain, issues Let's Encrypt cert automatically
```

### Cutover plan (preserving ad pixel + Google Ads conversion attribution)

The single most important constraint per PROJECT.md: **active ad spend on TikTok, Meta, and Google Ads must not see broken landing pages during the swap.**

**Pre-cutover checklist (all done on `jo-guesthouse.vercel.app` while WP is still live):**

1. New site is deployed to Vercel default subdomain
2. GTM container `GTM-MK4WJPMF` is installed and firing `whatsapp_click`, `enquiry_submit`, `ota_click` events (verified in GTM Preview Mode)
3. All four pixels (Meta, TikTok, Google Ads, GA4-if-present) confirmed firing in GTM Preview, cross-verified in:
   - Meta Events Manager → "Test Events" tab
   - TikTok Events Manager → "Test Event Code"
   - Google Ads → "Conversions" → "Recent conversions"
4. Web3Forms key set in Vercel env, form submits a test message to owner inbox
5. JSON-LD validates in Google Rich Results Test
6. Lighthouse mobile run: LCP < 2.5s on Slow 4G, TBT < 200ms (else Partytown fallback per STACK.md)

**Cutover steps (in this order):**

1. **T-0:** Pause active TikTok / Meta / Google Ads campaigns (avoids spend during DNS propagation chaos — typically 5-60 minutes for `.my.id`)
2. **T+1:** Change DNS at the registrar — apex `joguesthouse.my.id` → Vercel (registrar-specific: CNAME flattening, ALIAS, or two A records to Vercel's anycast IPs)
3. **T+2:** Add `joguesthouse.my.id` as production domain in Vercel dashboard, wait for SSL issuance (~2 min)
4. **T+3:** Verify the live `https://joguesthouse.my.id` serves the new site (cache-bust with `?v=1`)
5. **T+4:** Re-verify all 4 pixels fire on `joguesthouse.my.id` (not the vercel.app subdomain) using browser network tab + GTM Preview
6. **T+5:** Verify Google Ads conversion tag fires (Google Ads "Tag Assistant" extension)
7. **T+6:** Unpause campaigns
8. **T+24h:** Check Meta / TikTok / Google Ads dashboards — confirm conversions are reporting on the new domain
9. **T+7d:** Submit new `sitemap-index.xml` in Google Search Console; verify property ownership (DNS TXT or HTML file via `public/`)

**Branch protection:** require PR to merge into `main`, run `astro check` + `astro build` in CI on every PR.

**Rollback:** if anything goes wrong, change DNS back to WP host. Old site is still on the WP host until explicitly torn down — do NOT delete WP until 7 days of stable Vercel ad attribution.

---

## Build Order / Dependency Graph

This is the critical input for the roadmap. Items lower in the list depend on items above.

```
Phase A: Foundation (nothing else can ship without these)
    1. astro.config.mjs + tsconfig + Tailwind v4 + sitemap + vercel adapter
       → blocks everything
    2. BaseLayout.astro + global.css + fonts
       → blocks every section
    3. src/content/site.ts + strings.ts + lib/i18n.ts
       → blocks every component that renders copy

Phase B: Tracking + SEO scaffold (must be in place before any CTA work)
    4. GtmHead.astro + GtmBody.astro + lib/track.ts
       → blocks every interactive CTA (WA, OTA, form)
    5. SeoHead.astro + JsonLd.astro + lib/schema.ts
       → blocks production launch, but not section dev

Phase C: Core CTA primitive (everything pulls this in)
    6. WhatsAppButton.astro + lib/wa.ts + scripts/wa-cta.ts + lib/utm.ts
       → blocks Hero, StickyWaCta, Enquiry, Rooms, Footer
    7. Container.astro + SectionHeading.astro
       → blocks every section

Phase D: Image pipeline (parallel with C)
    8. Photo migration WP → src/assets/photos/ + crop/normalize
       → blocks Hero, Rooms, About, Facilities
    9. public/og-image.jpg pre-baked
       → blocks SEO launch verification

Phase E: Sections (mostly parallelizable — depend only on A/B/C/D)
    10. Hero.astro                     (depends on: 6, 8)
    11. Header.astro + scripts/mobile-nav.ts
    12. TrustBar.astro                 (depends on: site.ts only)
    13. About.astro                    (depends on: 8)
    14. Rooms.astro + RoomCard.astro   (depends on: rooms.ts, 8)
    15. Facilities.astro + FacilityItem.astro + astro-icon setup
    16. Location.astro                 (Google Maps iframe; depends on: site.ts geo)
    17. OtaBadges.astro + OtaBadge.astro + scripts/ota tracking
    18. Faq.astro + faq.ts (+ FAQPage JSON-LD)
    19. Footer.astro

Phase F: Sticky/floating UX
    20. StickyWaCta.astro + scripts/sticky-cta.ts  (depends on: 6, 10)

Phase G: Form
    21. Enquiry.astro + scripts/enquiry.ts + Web3Forms env var  (depends on: 6 for the WA link above the form)

Phase H: Launch
    22. Lighthouse mobile audit; Partytown fallback if TBT > 200ms
    23. Cutover (DNS swap, pixel re-verification, sitemap submission)
    24. Search Console + Bing Webmaster
    25. Decommission WP after 7 days of stable Vercel attribution
```

**Dependency hot spots:**
- `WhatsAppButton.astro` (item 6) is the single most-blocking primitive — it's pulled into ~6 sections. Build and unit-test it first after the foundation.
- `src/content/site.ts` (item 3) is read by ~15 files. Define the full shape early; resist the urge to add ad-hoc properties later.
- `lib/track.ts` (item 4) must exist before any CTA — otherwise WA clicks land but don't attribute.

---

## Scaling Considerations

This site will not "scale" in the typical sense — it's a 17-room guesthouse landing page, not a SaaS app. The scaling axes that matter:

| Scale axis | At v1 | At 5-10× ad spend | If owner adds rooms / locations |
|------------|-------|-------------------|----------------------------------|
| Traffic | 1-3k sessions/mo | 10-30k sessions/mo | same — content scales, not request volume |
| Vercel bandwidth | <500 MB/mo | <5 GB/mo (free tier ceiling 100 GB) | same |
| Vercel Analytics events | <3k/mo (free 2.5k tier may exceed) | 25-50k events (move to Plausible $9/mo) | same |
| Content surface | 1 page | 1 page | Consider per-room pages or multi-property switcher |
| Builds | ~30s | ~30s | ~45s if photos grow to 100+ |

**First bottleneck:** Vercel Analytics free tier (2,500 events/month). Mitigation: switch to Plausible or rely on GTM/GA4 only.
**Second bottleneck:** Owner asks for a second property. Mitigation: lift the content into a `properties.ts` array and add a route under `src/pages/[slug].astro`. Architecture supports this without restructuring — `site.ts` becomes `properties[0]`.

No real horizontal scaling concerns. Vercel edge CDN handles any traffic this guesthouse will ever generate.

---

## Anti-Patterns

### Anti-Pattern 1: Calling `fbq`/`ttq`/`gtag` directly from a component

**What people do:** Inline a `window.fbq('track', 'Contact')` call inside `Hero.astro` because "it's just one line."
**Why it's wrong:** Now four files (one per pixel) own that logic. Adding a new pixel requires editing every CTA. The owner cannot manage anything from GTM.
**Do this instead:** Push to `dataLayer` via `lib/track.ts`. Let GTM fan out.

### Anti-Pattern 2: Astro Content Collections for site copy

**What people do:** Set up `src/content/config.ts` with a Zod schema and `defineCollection` for site copy because the docs make it look standard.
**Why it's wrong:** Collections are designed for indexed lists of frontmatter-bearing markdown (blog posts, products with detail pages). For ~30 strings and a single page, the boilerplate (Zod schema, `getEntry`, slug routing) is a tax with no payoff. You also can't easily co-locate `id` and `en` values per string.
**Do this instead:** Plain typed `.ts` modules in `src/content/` (the directory name is conventional but no Collections API involved).

### Anti-Pattern 3: Adding React/Vue/Alpine for the lightbox or sticky CTA

**What people do:** `npx astro add react` because the lightbox needs state.
**Why it's wrong:** Adds 40+ KB of framework runtime to a site whose total JS budget should be under 10 KB. None of the interactions here need reactive state — they need event listeners and IntersectionObserver.
**Do this instead:** Vanilla TS in `src/scripts/*.ts`. Reach for a framework only when reactive state genuinely simplifies (e.g., a real availability widget with date range + room selection).

### Anti-Pattern 4: Putting photos in `public/`

**What people do:** Drop all 40 photos into `public/images/` because "it's just images."
**Why it's wrong:** `public/` files are served as-is — no AVIF, no WebP, no responsive `srcset`, no width/height optimization. LCP suffers, mobile bandwidth wastes. Indonesian users on 4G feel every wasted kilobyte.
**Do this instead:** Photos in `src/assets/photos/`, rendered via `<Image>` / `<Picture>` from `astro:assets`. Only the OG image (1200×630, needs stable URL) lives in `public/`.

### Anti-Pattern 5: Skipping the OG image because "we'll add it later"

**What people do:** Ship without a `public/og-image.jpg`; the OG meta points to nothing.
**Why it's wrong:** Every WhatsApp share, Facebook post, and TikTok bio link will render a broken card or fall back to a random page screenshot. This is one of the cheapest conversion wins available and it's free at build time.
**Do this instead:** Pre-bake `og-image.jpg` at 1200×630 with brand + hero photo + tagline before launch.

### Anti-Pattern 6: Building two routes (`/` and `/en/`) for v1

**What people do:** Set up Astro's i18n config with `/en/` mirror because "bilingual is best practice."
**Why it's wrong:** Audience is 90% Bahasa per PROJECT.md. Two routes fragment SEO authority, double the maintenance burden of every copy change, complicate the WhatsApp UTM strategy (which landing variant did the click come from?), and split ad-campaign landing-page URLs. The owner explicitly directed against it.
**Do this instead:** Bahasa primary; three or four EN strings inline via `t(key, 'en')`. Re-evaluate after 3 months of analytics.

### Anti-Pattern 7: Hardcoding the WhatsApp number / GTM ID across files

**What people do:** `https://wa.me/6285108002536` typed into 6 different `.astro` files.
**Why it's wrong:** Owner changes WA number → grep, replace, miss one, ad-traffic dies for that section silently.
**Do this instead:** Everything in `src/content/site.ts`. WhatsApp URL is built by `lib/wa.ts` which reads `site.ts`.

### Anti-Pattern 8: Letting JSON-LD drift from displayed content

**What people do:** Hand-author the JSON-LD blob separately from the address/phone strings shown on the page.
**Why it's wrong:** Owner updates the address in `site.ts`, the page shows the new one, but the schema still emits the old address. Google sees inconsistency, downranks for local SEO.
**Do this instead:** `lib/schema.ts → buildLodgingSchema()` reads from `site.ts`. One source.

---

## Integration Points

### External Services

| Service | Integration | Notes |
|---------|-------------|-------|
| GTM (`GTM-MK4WJPMF`) | `<script is:inline>` in `<head>` + `<noscript><iframe>` first in `<body>` | Already configured in owner's GTM account; no new tags to set up in v1 |
| Meta Pixel (`770014465395193`) | Configured inside GTM, fires on `whatsapp_click` + `enquiry_submit` | Pixel ID lives in GTM, NOT in code |
| TikTok Pixel (`D2MR5GRC77U4PA826B90`) | Same — GTM-routed | Same |
| Google Ads conversion (`AW-17438288457`) | Same — GTM-routed | Same |
| Google Maps Embed | Iframe `loading="lazy"` in `Location.astro` | Below the fold; no API key needed for basic embed |
| Web3Forms | `<form action="https://api.web3forms.com/submit" method="POST">` + `fetch()` in `enquiry.ts` for AJAX submit | Access key in `PUBLIC_WEB3FORMS_KEY` Vercel env var |
| WhatsApp (`wa.me`) | `<a href={buildWaUrl(...)} target="_blank" rel="noopener">` | Number format: `6285108002536` (no `+`, no spaces) |
| Vercel | Git push → auto-deploy | `output: 'static'`, Vercel adapter for native Analytics |
| Vercel Web Analytics | `@vercel/analytics` script tag in `BaseLayout` (or enabled in adapter config) | 2,500 events/mo free; sufficient for v1 |
| OTAs (Agoda, Traveloka, Booking) | Plain outbound `<a target="_blank">` from `otas.ts` | No API; URLs hand-curated, tracked on click |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Component ↔ content | `import { site } from '../../content/site'` (typed) | One-way, components never write content |
| Component ↔ tracking | `import { trackWaClick } from '../../lib/track'` (typed wrappers) | Components never touch `window.dataLayer` |
| Script ↔ DOM | Delegated event listeners on `document` with `data-*` selectors | Allows component-level `data-*` attributes to wire behavior without per-component scripts |
| Astro build ↔ Sharp | `astro:assets` calls Sharp under the hood at build time | Transparent; Vercel's Node 20 build image has Sharp's binaries |
| Page ↔ GTM | One-way: page pushes events to `dataLayer`; GTM reads | GTM never calls into page code |
| Form ↔ Web3Forms | `fetch()` POST with FormData; expects `{success: boolean}` JSON | Failure path: show error toast, fall back to "Lebih cepat via WhatsApp" message |

---

## Sources

- `.planning/research/STACK.md` (this project, 2026-05-12) — stack decisions referenced throughout (Astro 6, Tailwind v4 via Vite, GTM-as-router, Web3Forms, no Partytown by default, no i18n routing).
- `.planning/PROJECT.md` (this project, 2026-05-12) — scope and constraints driving the single-page, Bahasa-primary, WhatsApp-first architecture.
- Context7 `/withastro/docs` — Astro 6 project structure conventions (`src/pages`, `src/layouts`, `src/components`, `src/assets`, `public/`), `<script>` import pattern for client islands, `astro:assets` `<Image>`/`<Picture>` API. **HIGH** confidence.
- [Astro Project Structure](https://docs.astro.build/en/basics/project-structure/) — confirms `src/content/` is a conventional folder name (not exclusively for Content Collections); plain TS modules are fine. **HIGH** confidence.
- [Astro Server Islands & Scripts](https://docs.astro.build/en/guides/client-side-scripts/) — `<script>` blocks are processed by Vite, type-checked, and bundled; one script imported from multiple components is deduped. **HIGH** confidence.
- [WhatsApp Click-to-Chat URL format](https://faq.whatsapp.com/5913398998672934) — number format and URL-encoded pre-filled text. **HIGH** confidence.
- [Google Search Central — LocalBusiness structured data](https://developers.google.com/search/docs/appearance/structured-data/local-business) — `LodgingBusiness` schema fields and Rich Results validation. **HIGH** confidence.
- [Web3Forms Astro guide](https://docs.web3forms.com/how-to-guides/static-site-generators/astro) — form POST pattern, access key injection. **HIGH** confidence.
- [Vercel deploy + custom domain docs](https://vercel.com/docs/projects/domains) — DNS cutover pattern, SSL auto-issuance. **HIGH** confidence.

---

*Architecture research for: mobile-first conversion-focused single-page hospitality landing site (Indonesia)*
*Researched: 2026-05-12*
