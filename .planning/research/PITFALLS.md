# Pitfalls Research — Jo Guest House

**Domain:** Cutover of a WordPress marketing site (with active paid-traffic spend) to a static Astro+Tailwind landing page, Indonesian audience, WhatsApp-first booking, free `.my.id` TLD, Vercel deploy.
**Researched:** 2026-05-12
**Confidence:** HIGH (cutover/tracking/SEO/Vercel ToS) · HIGH (WA + OTA brand mechanics) · MEDIUM (UU PDP cookie applicability; Partytown flakiness; static testimonial perception) · MEDIUM-LOW (exact Indonesian-mobile RUM percentiles — single-property RUM is private)

The single most under-appreciated risk in this rebuild is **not** anything in the code — it's the seven days surrounding DNS cutover, where four ad campaigns must keep firing the right conversion events against a domain whose pixels were silently re-implemented by a different developer. Everything below is sorted with that lens.

---

## Critical Pitfalls

### Pitfall 1: Vercel Hobby plan prohibits commercial use — current deploy plan violates Vercel ToS

**What goes wrong:**
STACK.md recommends Vercel free tier (Hobby) with `output: 'static'`. Vercel's [Hobby plan terms](https://vercel.com/docs/plans/hobby) and Fair Use guidelines **explicitly prohibit commercial use** — including marketing sites that drive revenue, client work, and any site running paid advertising campaigns. Jo Guest House is a paying business with active TikTok/Meta/Google Ads spend; deploying it on a Hobby account is a ToS violation that triggers account suspension when Vercel's abuse detection notices the commercial intent (ad referrers, `LodgingBusiness` schema, business address in footer). Account suspension takes the site dark mid-campaign with zero notice.

**Why it happens:**
"Free tier" gets conflated with "free for any use" in builder shorthand. The Hobby plan's 100 GB bandwidth / 1M edge requests fit the site's traffic profile easily, so the trap is silent — there is no usage-based warning, only an abuse-detection warning that arrives after suspension.

**How to avoid:**
- Owner pays for **Vercel Pro ($20/user/month)** OR the site is hosted on a free tier that *does* allow commercial use:
  - **Cloudflare Pages** — free tier explicitly permits commercial use; 500 builds/month, unlimited bandwidth, single custom domain free. Best free-tier alternative.
  - **Netlify Starter** — free tier permits commercial use up to 100 GB bandwidth/month and 300 build minutes; commercial-friendly.
  - **GitHub Pages** — free, permits commercial use for public repos; static only, no edge functions, no preview deploys per PR — workable for v1 but worse DX.
- Recommend: **switch the deploy target to Cloudflare Pages** for v1. Cloudflare's Singapore POP gives marginally better Indonesian latency than Vercel SIN1 in practice (verify with WebPageTest from Jakarta). Astro builds for CF Pages with `@astrojs/cloudflare` or just static output + `wrangler pages publish`.
- If owner agrees to pay: Vercel Pro is fine and preserves STACK.md as-is (Vercel Analytics, image optimization, adapter). Decision must be logged before phase that touches deploy config.

**Warning signs:**
- Builder account is `michalpavelec@...` personal Vercel, project is in builder's team — bus-factor + ToS exposure simultaneously
- Any Vercel email about "Fair Use" or "commercial usage" — immediate red flag
- Owner's name on the Vercel project = correct; builder's = ToS risk

**Phase to address:**
**Phase 0 — Project Setup / Decision Log.** Resolve hosting target before any DNS work. Reopen the `Key Decisions` table in `PROJECT.md` to log this — current STACK.md is silent on the ToS issue.

---

### Pitfall 2: Pixel IDs silently swapped or events renamed during reinstall — ad campaigns go dark

**What goes wrong:**
The current site has four live tracking surfaces — TikTok Pixel `D2MR5GRC77U4PA826B90`, Meta Pixel `770014465395193`, GTM container `GTM-MK4WJPMF`, Google Ads conversion `AW-17438288457`. On rebuild, the developer copy-pastes from a tutorial or last project and accidentally:
- types the wrong character into a pixel ID,
- forgets the GTM container ID and uses a new one,
- routes through a fresh GTM container instead of the existing one,
- pushes a `dataLayer` event named `whatsapp_click` when the existing GTM tag is listening for `wa_click` (or vice versa),
- forgets the `<noscript>` GTM body fragment so ~5–10% of traffic (no-JS / strict privacy) drops out of conversion attribution silently,
- installs the Meta Pixel as base-code only and skips the `Lead` / `Contact` event that the existing campaign optimizer is using as its conversion goal.

Campaigns continue spending against an optimizer that has stopped receiving conversion signal. Within 48–72 hours Meta/TikTok auto-bidding degrades to garbage placements; Google Ads pauses for "Conversion tracking inactive" warnings; recovery requires re-learning the bidding model.

**Why it happens:**
- IDs look like random strings; a one-character typo isn't visible to the eye and isn't caught by any linter.
- GTM is "managed by the owner" so the existing event names are tribal knowledge — the rebuild dev never opens the existing GTM UI.
- The owner can't QA the pixels themselves; the dev marks the task done; nobody actually verified live fires.

**How to avoid:**
- **Pre-cutover: audit the existing GTM container before touching the new site.** Log into the GTM UI (owner shares access), screenshot every tag, every trigger, every variable. Document the exact `dataLayer` event names the existing tags listen to. This is the source of truth — STACK.md's recommendation to "route everything through GTM" only works if the new site fires the *same* event names the existing tags already use.
- **Treat pixel IDs as build-time constants in one file** (`src/lib/tracking.ts`) — single source of truth, code-reviewed once, never inlined elsewhere.
- **Verification protocol before DNS cutover** (run against `joguesthouse.vercel.app` or staging URL):
  1. **GTM Preview mode** (Tag Assistant) — load the page, scroll through, click each WA CTA, submit the form. Confirm every tag fires. Screenshot the Tag Assistant timeline.
  2. **Meta Pixel Helper** (Chrome extension) — confirm `PageView` fires, `Lead` (or whichever event the existing campaign uses) fires on WA click. Confirm pixel ID matches `770014465395193`.
  3. **TikTok Pixel Helper** — confirm `Pageview` + click event, ID `D2MR5GRC77U4PA826B90`.
  4. **Google Tag Assistant** — confirm Google Ads conversion `AW-17438288457` fires on WA click with the existing conversion label.
  5. **Network panel** — search for `gtm.js?id=GTM-MK4WJPMF`, `connect.facebook.net`, `analytics.tiktok.com`, `googleads.g.doubleclick.net`. Confirm 200 responses, not blocked.
  6. **Meta Events Manager / TikTok Events / Google Ads conversion diagnostics** — wait 30 minutes after the test, log in, confirm test events are visible server-side. This is the only verification that catches CSP / referrer / cookie-blocking issues.
- **Build a one-page `/cutover-checklist.md`** in `.planning/` enumerating these checks; sign off (initials + timestamp) before flipping DNS.

**Warning signs:**
- Meta Events Manager "Diagnostics" tab shows "0 events in last 24 hours" on staging during testing — pixel is mis-installed
- GTM Preview mode shows tags firing but Tag Assistant shows pixel IDs you don't recognize — wrong container loaded
- Google Ads campaign UI shows "Conversion tracking inactive" or "Recently received conversions: 0" — broken conversion path
- TikTok Ads Manager → Events → "Last received" timestamp older than 6 hours during active testing

**Phase to address:**
**Phase: Tracking & Cutover.** Must run *before* DNS swap. Block cutover if any check fails. Recovery cost after going dark is dramatically higher than verification cost (re-training Meta/TikTok bidding takes 7–14 days at full budget).

---

### Pitfall 3: DNS cutover with no rollback path — broken launch takes the business offline

**What goes wrong:**
The new Astro site is deployed; builder updates `joguesthouse.my.id` DNS to point at Cloudflare/Vercel; within 30 minutes some critical bug appears (WA link broken on iOS Safari, hero image not loading on Android Chrome, GTM container failing to load due to CSP). The old WordPress install is still on the previous host but no one can route traffic back to it quickly because:
- DNS TTL was left at 24 hours (default), so even reverting DNS takes a day to propagate
- the WordPress host's nameservers were the source-of-truth and the new DNS provider doesn't have the old A records snapshotted
- the WP install's IP address was never written down

Active ad spend continues against a broken site for 12–48 hours.

**Why it happens:**
- "Rollback plan" is a checkbox skipped under time pressure
- DNS TTL is invisible until you need it
- The builder believes their pre-flight checks are sufficient (they usually are — until the one time they aren't)

**How to avoid:**
- **48 hours before cutover: lower DNS TTL to 300 seconds (5 min)** at the current registrar. After cutover stabilizes (24h post-launch), raise it back to 3600+.
- **Document the old WordPress origin IP / hosting nameservers in `.planning/cutover-checklist.md`** before any DNS change. Screenshot the DNS zone *as it currently exists*.
- **Keep the WordPress install running** for at least 14 days post-cutover, on its own subdomain (`old.joguesthouse.my.id` or just-IP-only). Do not delete the WP container until after Google Search Console shows old URLs successfully redirected and indexed in their new form.
- **Define rollback criteria in advance:** WA CTA tap rate drops to zero / Lighthouse mobile LCP > 5s in production / GTM container 404s for >50% of users / Meta Events Manager shows no events for 60 minutes — any of these = roll back DNS immediately.
- **Cutover at off-peak hours for the audience** — Indonesian budget-travel traffic is heaviest evening (19:00–22:00 WIB) and weekends. Cut over Tuesday 02:00–05:00 WIB; have the builder awake and watching for 4 hours after.

**Warning signs:**
- DNS TTL was never lowered → automatic 24h+ rollback latency
- "We don't need a staging URL, we'll just deploy and check" → no comparison baseline
- Builder is asleep / unavailable in the 4 hours after DNS swap
- No documented "if X then roll back" criteria

**Phase to address:**
**Phase: Cutover.** Cutover checklist MUST include TTL pre-lowering (T-48h), origin-IP snapshot (T-48h), rollback criteria (T-24h), monitoring window assignment (T+0 to T+4h).

---

### Pitfall 4: Old WordPress URLs still indexed in Google — orphan 404s burn SEO and OG previews

**What goes wrong:**
Google's index still contains entries like:
- `https://joguesthouse.my.id/sample-page/` with title `Sample Page – My WordPress Blog`
- `https://joguesthouse.my.id/?p=2` or `https://joguesthouse.my.id/2026/03/12/hello-world/`
- the homepage itself crawled with title `My Blog – My WordPress Blog`

After cutover to the static Astro site, those URLs return 404 (Astro `output: 'static'` has no router for arbitrary WP-style paths). Side effects:
- ad-traffic deep links from any historical share return 404 → wasted ad clicks
- Google's site:joguesthouse.my.id results show the embarrassing old titles for 6–12 weeks until re-crawl
- WhatsApp link-preview cache (extremely persistent — sometimes 30+ days) still serves the old OG-less preview when users share the URL
- the sitemap previously pointed at WordPress URLs Google now treats as soft-404s, hurting site quality score

**Why it happens:**
- Static site builders forget that Google's crawl backlog includes paths they never created
- WhatsApp's link cache is opaque and aggressive; nobody tests it
- the old WordPress sitemap (`/wp-sitemap.xml`) is still being requested by Googlebot for weeks

**How to avoid:**
- **Inventory the existing Google index pre-cutover:**
  - Run `site:joguesthouse.my.id` in Google and Bing, screenshot all results
  - Pull crawled URLs from Google Search Console → Pages → Indexed (if access exists; if not, [acquire from owner](https://search.google.com/search-console))
  - Pull from `https://joguesthouse.my.id/wp-sitemap.xml` (or `/sitemap_index.xml`) — copy every URL into the cutover plan
- **At cutover, implement 301 redirects for every known WP path → `/` (or to the nearest section anchor):**
  - On Cloudflare Pages: use `_redirects` file at site root: `/sample-page/ / 301`, `/?p=* / 301`, etc.
  - On Vercel: use `vercel.json` redirects array
  - Catch-all rule: any unknown path → `/` with 301 (acceptable for a one-page site; do NOT use 404 for old WP routes during the first 6 months)
- **Submit new `sitemap.xml` to Google Search Console immediately after cutover** + use **Change of Address tool only if domain changes** (we're not changing domain, so skip — instead force re-crawl via "Inspect URL" → "Request indexing" on the homepage)
- **Force WhatsApp to re-crawl the OG card:** [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/) → paste URL → "Scrape Again" twice. WhatsApp's preview infrastructure shares Facebook's cache; this is the only reliable way to invalidate it.
- **Keep redirects in place 6–12 months minimum** (per Google's guidance) — do not delete `_redirects` early.

**Warning signs:**
- Search Console "Pages" report shows >5 "Not Found (404)" URLs in week 1 post-cutover
- Sharing the URL on WhatsApp still shows "My Blog – My WordPress Blog" preview after re-scrape
- Direct search `site:joguesthouse.my.id` still surfaces `/sample-page/` after 4 weeks

**Phase to address:**
**Phase: Cutover (redirects + sitemap) + Phase: Post-launch monitoring (week 1–4).** The Search Console verification is owner-facing and must be set up *before* DNS flip so old-URL traffic can be measured.

---

### Pitfall 5: Pre-filled WhatsApp message broken on iOS Safari desktop / web.whatsapp.com

**What goes wrong:**
`wa.me/6285108002536?text=Halo%20Jo%20Guest%20House...` works perfectly on Android. On **iOS Safari desktop** and **Firefox desktop**, clicking the link either:
- opens `web.whatsapp.com` without the pre-filled text (the `?text=` parameter is dropped),
- opens a "WhatsApp is not installed" prompt even when WhatsApp Desktop is installed,
- or just navigates to `wa.me`'s landing page asking the user to install WhatsApp.

This affects roughly 5–10% of an Indonesian-leaning audience (iPhone share + the small but real desktop browsing segment), but disproportionately includes higher-spend OTA-comparison-shopper traffic.

Worse: pre-filled messages with **newlines** (`%0A`) — used in the structured booking template (DF-02) — get mangled by some Safari versions, arriving as `\n` literals or stripped entirely. Owner sees `"Halo Jo Guest House saya mau pesan kamarTanggal check-in: ___Tanggal check-out: ___..."` — unusable.

**Why it happens:**
- `wa.me` is a redirect service; different browsers handle the redirect with different `Referer` and query-string preservation policies
- WhatsApp Web's URL handler is more strict about the `text` parameter than the mobile app
- iOS Safari's "Prevent Cross-Site Tracking" can strip query params on certain redirect chains
- Newline encoding (`%0A` vs `%0D%0A` vs literal `\n`) is inconsistent across WA's parsing

**How to avoid:**
- **Use the `api.whatsapp.com/send` endpoint as the primary URL, not `wa.me`:**
  - `https://api.whatsapp.com/send?phone=6285108002536&text=...` — more reliable preservation of `text` across desktop browsers (per community fixes, MEDIUM confidence)
  - OR: detect platform client-side and rewrite:
    ```js
    const isMobile = /Android|iPhone|iPad/i.test(navigator.userAgent);
    href = isMobile ? `https://wa.me/${num}?text=${msg}` : `https://web.whatsapp.com/send?phone=${num}&text=${msg}`;
    ```
- **Test the pre-fill on a real device matrix:**
  - Android Chrome + WhatsApp app installed ✓
  - iOS Safari mobile + WhatsApp app installed ✓
  - iOS Safari mobile + no WhatsApp app (should redirect to App Store)
  - macOS Safari desktop + WhatsApp Desktop installed
  - macOS Safari desktop without WhatsApp
  - Windows Chrome → web.whatsapp.com
  - Windows Firefox → web.whatsapp.com
  - Inside Instagram in-app browser (huge for TikTok/Meta ad traffic on iOS — opens links in IG's own webview)
  - Inside Facebook in-app browser
- **Verify newline rendering:** the structured pre-fill arrives with real line breaks in WhatsApp, not literal `\n`. If `%0A` fails, try `%0D%0A`; if that fails, build the message as one comma-separated line and accept the UX loss.
- **Have a fallback CTA:** if `wa.me` fails, show the phone number as plain copyable text + `tel:` link. Don't trust a single deep-link to work universally.

**Warning signs:**
- Owner reports receiving messages with `\n` literals or blank `text` parameter
- Mobile Safari users tap CTA, land on `wa.me` info page, bounce
- In-app browser (TikTok / Instagram / Meta ads) users never reach WhatsApp at all

**Phase to address:**
**Phase: WhatsApp Integration + Pre-launch QA.** The device matrix test is non-negotiable; add to the cutover checklist.

---

### Pitfall 6: OTA brand logos used without permission — listing demotion or delisting risk

**What goes wrong:**
The site displays Agoda, Traveloka, and Booking.com logos as "OTA badges" (TS-09 in FEATURES.md). The builder grabs the logos from Google Image search, embeds them as PNGs, links them out to the OTA listing. This is **technically a trademark use**, and each OTA has explicit logo-usage guidelines:
- [Agoda explicitly requires written permission for commercial logo use](https://www.agoda.com/press/agoda-logo-guidelines/) and prohibits any modification
- Booking.com's brand guidelines define clear-space and minimum-size rules; misuse can be flagged via the partner portal
- Traveloka is similarly protected (under Agoda corporate)

For a listed property, the **real** business risk is not a cease-and-desist (rare) — it's that **the OTA can demote your listing visibility or terminate your contract** for unauthorized brand use, parity violations, or rate-undercutting. Property contracts include morality/brand clauses; field reps occasionally crawl partner sites and report issues.

**What goes wrong specifically:**
1. Logos used at wrong colors / on wrong backgrounds / below minimum size — minor trademark violation
2. Logos used alongside language like "lebih murah di sini" or "harga terbaik daripada Agoda" — **rate parity violation**, contract-terminable
3. Logos imply endorsement when they're just outbound links — gray area, usually fine but auditable

**Why it happens:**
- Builder treats OTA logos as "stock content" not as licensed trademarks
- Copy that ranks well ("cheaper than OTAs") is the same copy that gets you delisted
- Owner doesn't know the contract clauses they signed when listing

**How to avoid:**
- **Use the OTA name as plain text or a generic icon, NOT the logo, unless owner can produce written permission from each OTA.** Examples:
  - "Tersedia juga di **Agoda**, **Traveloka**, **Booking.com**" — text-only, hyperlinked. Trademark-safe.
  - Generic globe / link icon next to each name — visual without using the protected mark.
- **If logos are used:**
  - Get from official press kits ([Booking.com brand assets via the partner portal](https://www.bookingholdings.com/wp-content/uploads/2023/07/BHI_BrandGuidelines_ForMediaRoom.pdf), [Agoda press](https://www.agoda.com/press/agoda-logo-guidelines/), Traveloka via the affiliate portal)
  - Maintain clear-space, minimum size, approved color variants
  - Never modify (no recoloring, no cropping, no effects)
  - Display alongside other OTAs (not in isolation, which suggests exclusive partnership)
- **Audit ALL copy for parity violations before launch.** Banned phrases:
  - "lebih murah dari [OTA]", "harga terbaik", "cheapest direct", "OTA price + X%"
  - Any explicit price comparison with an OTA
- **Safe direct-booking framing** (per DF-14):
  - "Pesan langsung via WhatsApp — respon cepat, fleksibel" ✓
  - "Hubungi langsung pemilik untuk pertanyaan khusus" ✓
  - "Konfirmasi cepat via WhatsApp" ✓
- **Confirm owner has a clean property listing on each OTA** — if Agoda/Traveloka/Booking rating is below 4.0, the OTA badge promise of trust is broken; verify before referencing ratings (DF-06).

**Warning signs:**
- Builder asks "where can I download the Booking.com logo?" → flag for trademark conversation
- Marketing copy includes any direct OTA price comparison
- Owner does not know the names of their OTA account managers

**Phase to address:**
**Phase: Content Audit (pre-launch).** Trademark and parity review is part of content sign-off, not engineering — but the eng phase implementing OTA badges (TS-09) is the gate that enforces it.

---

### Pitfall 7: Repeating the current site's Lorem Ipsum disaster on the new build

**What goes wrong:**
The current WP site has visible placeholder copy ("A Title to Turn the Visitor Into a Lead", "Add Your Heading Text Here", Lorem Ipsum paragraphs). The rebuild is supposed to fix this. But owner has explicitly stated **no fresh photos or copy will be delivered** ("we don't need it now"). Without fresh content, the rebuild defaults to:
- Re-using the same Lorem Ipsum the builder grabs from the WP HTML as a "temporary" placeholder
- Generating AI Bahasa copy that sounds slightly off ("Selamat datang di hotel kami yang menyediakan pengalaman menginap yang tak terlupakan...") — Indonesian users instantly clock this as machine-generated
- Promising features the property doesn't actually have ("kolam renang", "sarapan gratis", "antar-jemput bandara") because the builder copied a template

The new site looks "complete" at staging, ships, and is just as embarrassing as the old one — just with better fonts.

**Why it happens:**
- Owner is non-technical and conflates "ready to ship" with "looks designed"
- Builder fills gaps with placeholder to demo design progress, forgets to flag for owner
- Bahasa AI-generated copy is often grammatically perfect but tonally generic — looks fine to a non-native reviewer
- The deadline pressure ("paid traffic is burning") incentivizes shipping over content quality

**How to avoid:**
- **Create a content inventory before any layout work begins** (`/.planning/content-inventory.md`):
  - Hero headline (Bahasa) — exact final string
  - Hero subhead — exact final string
  - Each amenity label — owner-confirmed final string
  - Each FAQ Q&A — owner-confirmed answers
  - 3 testimonial quotes — real first name, city, month
  - Distance/time claim to CGK — owner-verified with traffic context
- **Block ship on placeholder text.** Build a tiny pre-deploy script that greps `dist/` for: `Lorem`, `ipsum`, `Add Your`, `Sample Page`, `My Blog`, `placeholder`, `TODO`. Any match = build fails.
- **For unavoidable content gaps, use honest interim copy** (in Bahasa): "Foto kamar segera diperbarui." "Detail menyusul." Not Lorem Ipsum. Not AI hallucination. Honest > polished-but-wrong.
- **Repurpose the existing 40 WP photos** explicitly, with manual selection and ordering. Don't dump all 40 into a gallery — pick the 12–18 best, reject any with visible Lorem Ipsum signage, branding errors, USD prices, or empty rooms with disarrayed beds.
- **Owner must sign off on every visible string** before ship (single PDF or Google Doc with all final strings, owner ticks each). If owner refuses, document it and the builder takes responsibility for the content choice.

**Warning signs:**
- Builder's design mocks contain Lorem Ipsum past the first design iteration
- "We'll finalize copy at the end" — never happens
- Bahasa copy reads grammatically perfect but tonally generic — paste it into ChatGPT and ask "did an AI write this?" If yes, rewrite
- Owner has not seen final strings during preview-deploy review

**Phase to address:**
**Phase: Content Inventory (Phase 1 — must precede layout) + Pre-launch QA gate.** Add the `grep dist/` placeholder check to CI.

---

### Pitfall 8: Schema.org JSON-LD invalid / Rich Results Test fails — no rich result, no local pack

**What goes wrong:**
STACK.md proposes a `LodgingBusiness` JSON-LD block with hard-coded `latitude: -6.165, longitude: 106.738` and amenity array. Common ways this breaks:
- Coordinates are *approximate* (anyone copying from STACK.md ships the placeholder) — Google Maps shows the property in a random Cengkareng street, not the actual Ruko Plaza de Lumina
- `priceRange: "Rp"` is invalid — Google expects either `$`/`$$`/`$$$` style OR a specific numeric range like `Rp 200000-Rp 200000`. A bare currency symbol fails Rich Results Test silently (no rich result, no error to owner)
- `image` field points to relative URL `og-image.jpg` instead of absolute `https://joguesthouse.my.id/og-image.jpg` — Rich Results Test fails
- `addressCountry: "ID"` is correct ISO; `addressCountry: "Indonesia"` is also accepted but inconsistent — pick one and stick
- `telephone` formatted as `+62-851-0800-2536` (dashes) sometimes parses; `+6285108002536` is safer per Google's docs
- The `LodgingBusiness` schema is rendered but never wrapped in `<script type="application/ld+json">` (Astro template error — `is:inline` interaction with `set:html`)

**Why it happens:**
- JSON-LD doesn't error visibly in the browser — it just doesn't enhance results
- Rich Results Test is a separate manual step that gets skipped
- Geo coordinates are tedious to verify; "close enough" feels acceptable but tanks Google Maps relevance

**How to avoid:**
- **Get the exact lat/long from Google Maps:** search "Ruko Plaza de Lumina, Jl. Outer Ring Road No.7, Duri Kosambi", right-click the pin, copy coordinates. Use 6 decimal places (`-6.165432, 106.738214`). Verify by pasting back into Google Maps search.
- **Validate with [Google Rich Results Test](https://search.google.com/test/rich-results)** — paste the staging URL, screenshot the result, fix every error and warning. Re-run after each change.
- **Use absolute URLs in JSON-LD** for `image`, `url`, `@id`. Astro: build a `<SEO>` component that prepends `Astro.site` to every URL field.
- **`priceRange` valid values:** Google accepts `"$"` / `"$$"` / `"$$$"` symbols OR a string like `"Rp 200000"` (single price OK). Recommend: `"priceRange": "Rp 200.000"` — explicit, scannable.
- **Verify the script tag wrapper** — view source on production, search for `application/ld+json`, paste content into [JSON-LD Playground](https://json-ld.org/playground/) to confirm valid JSON syntax.
- **Submit the canonical URL to Google Search Console** post-launch and check the "Enhancements" section for `LodgingBusiness` recognition (takes 1–4 weeks).

**Warning signs:**
- Rich Results Test returns "Page is eligible for rich results" but lists 0 detected items → tag isn't being parsed
- "Rich result not supported" with red error → required property missing
- Geo coordinates land in middle of a different street → bad coords
- Property doesn't appear in `site:joguesthouse.my.id` Google business panel after 4 weeks

**Phase to address:**
**Phase: SEO & Schema implementation + Pre-launch QA.** Rich Results Test pass is a hard launch gate.

---

### Pitfall 9: Rupiah price written in wrong locale format — "$200" or "Rp 200,000" looks like 200 dollars / 200K dollars

**What goes wrong:**
The current site shows USD prices ($125 / $255 / $375 / $425) — a 10× error vs. real Rp 200.000 (~$13 USD). The new site must format correctly:
- **Correct:** `Rp 200.000` (dot = thousands separator, Indonesian convention)
- **Wrong:** `Rp 200,000` (comma = US thousands separator; in Bahasa, comma is the DECIMAL separator — `Rp 200,00` means "200 rupiah and 00 cents")
- **Wrong:** `Rp200.000` (no space — looks cramped, non-standard)
- **Wrong:** `IDR 200.000` (technically correct, but `Rp` is universal in consumer-facing Indonesian copy)
- **Wrong:** `Rp 200K` (English shorthand)
- **Acceptable in body copy only:** `Rp 200rb` (`rb` = "ribu" = thousand) — colloquial, never in hero/headline
- **Wrong:** `$13 USD` shown anywhere as the primary price

If you use JavaScript's `Number.toLocaleString('id-ID', { style: 'currency', currency: 'IDR' })`, the output is `Rp 200.000,00` — with `,00` decimal that looks like an OTA error.

**Why it happens:**
- Designer uses `toLocaleString('en-US')` by default → comma separator
- Auto-formatting libraries (libphonenumber, etc.) get the locale wrong
- Designer's mockup looks fine in design tool but Tailwind class `number-format` doesn't exist (this is a string concern, not CSS)

**How to avoid:**
- **Hard-code the price as a string in source:** `<p class="text-3xl font-bold">Rp 200.000<span class="text-base">/malam</span></p>` — no runtime formatting. Single price, single source of truth.
- **Do NOT use `Intl.NumberFormat` or `toLocaleString`** for the displayed price. Use a string. If the price changes, change one constant in `src/data/property.ts` and rebuild.
- **Owner-confirm the price string** in the content inventory.
- **Audit for stale USD references** — grep `dist/` for `$`, `USD`, `dollar` before deploy. Fail build if found.

**Warning signs:**
- Hero price shows `Rp 200,000` → US format leak
- Hero price shows `Rp 200.000,00` → `Intl.NumberFormat('id-ID')` default leak
- Any visible `$` in production
- Content inventory still shows old USD prices

**Phase to address:**
**Phase: Content Inventory + Layout Implementation.** Static-string approach is the simplest and most reliable.

---

### Pitfall 10: Distance/time claim to CGK is false ("5 menit dari Bandara")

**What goes wrong:**
The hero says "±5 menit dari Bandara Soekarno-Hatta" because that's what the owner casually claimed in conversation. In reality, the Ruko Plaza de Lumina is ~10–12 km from CGK, which is 15–25 minutes via Jakarta Outer Ring Road in normal traffic, 30–45 minutes in rush hour. The claim:
- Sets unrealistic expectations → first guest leaves a 2-star OTA review citing "much further than promised, missed flight"
- Triggers ad platform policy (Meta's "misleading content" rules) on closer scrutiny — rare but possible
- Erodes the entire site's credibility once a single user verifies on Google Maps

**Why it happens:**
- Owners exaggerate proximity instinctively
- "5 menit" is a marketing trope in Indonesian airport-hotel copy that gets copied across listings
- Nobody actually drives the route at multiple times of day

**How to avoid:**
- **Verify with Google Maps** at three times: 06:00 WIB (off-peak), 12:00 WIB (midday), 18:00 WIB (rush). Record actual range.
- **Phrase the claim as a range with traffic context:** `±20 menit ke Bandara Soekarno-Hatta (jam normal); ±35 menit jam sibuk` — accurate AND addresses the airport-anxiety audience directly.
- **Include the JORR (Jakarta Outer Ring Road) reference** — it's a credibility signal that the property knows the route.
- **Cross-reference OTA listings** — what do Agoda/Traveloka say about the property's airport distance? If they say "15 menit," don't say "5 menit" on the direct site (parity-adjacent issue + creates internal-inconsistency suspicion).

**Warning signs:**
- Owner's claim and Google Maps disagree by >50%
- Distance claim is rounder than physics allows ("exactly 5 minutes")
- No specific traffic-time qualifier ("during off-peak" / "in normal traffic")

**Phase to address:**
**Phase: Content Inventory — owner-verification gate.**

---

### Pitfall 11: UU PDP cookie consent — partial applicability is being treated as "doesn't apply"

**What goes wrong:**
STACK.md and FEATURES.md (AF-16) recommend skipping a cookie consent banner because "UU PDP doesn't mandate cookie banners the way GDPR does." This is **mostly correct but oversimplified**. Indonesia's [PDP Law](https://fpf.org/blog/indonesias-personal-data-protection-bill-overview-key-takeaways-and-context/) — fully enforceable since **17 October 2024** — does require **valid consent for processing personal data**, and cookies/pixels that *identify individuals* are personal data under the law. The Meta Pixel, TikTok Pixel, and Google Ads conversion tracking specifically build user profiles → they fall under the consent requirement.

**Risk reality (MEDIUM confidence on enforcement, HIGH confidence on legal text):**
- Enforcement against a 17-room guesthouse is unlikely in 2026 — regulators focus on large platforms first
- BUT: PDP allows class actions by data subjects and administrative sanctions up to 2% of annual revenue
- AND: Meta/TikTok ad accounts may eventually be paused if regulators issue a directive against a property using their pixels without compliant consent

**Why it happens:**
- "Indonesia doesn't have a cookie banner culture" is partially true (consumer expectation is low) but legally wrong (compliance requirement exists)
- Builder applies GDPR-style thinking, sees no immediate enforcement, defaults to "skip it"

**How to avoid:**
- **Pragmatic middle path** (recommended for a 17-room guesthouse with no EU traffic):
  1. **Minimal privacy policy page** linked from footer — describes pixel use, lists Meta/TikTok/Google as data processors, explains user can opt out by clearing cookies / using browser extensions
  2. **NO blocking banner** — the conversion impact of a banner on Indonesian mobile ad traffic is severe (10–30% pixel signal loss is typical)
  3. **Implied consent model with clear disclosure** — the privacy link is visible, the user proceeds with use = implied consent. Defensible under PDP's "legitimate interest" basis for marketing analytics, though that's still legally contested.
- **If owner has any EU customer exposure** (foreign tourists from EU listed on Booking.com): full GDPR-compliant CMP becomes required — at that point reconsider. Current traffic profile says no.
- **Document the decision** in `PROJECT.md` Key Decisions table with the explicit reasoning.

**Warning signs:**
- Owner expands to target EU tourists → compliance bar rises
- Regulator (Kominfo / PDP authority) issues guidance specifically calling out pixels
- A privacy complaint is filed → respond within mandated window

**Phase to address:**
**Phase: Legal & Content (low-effort: ~30 min for a privacy page) + Decision Log update.**

---

### Pitfall 12: Image-heavy gallery on low-end Indonesian Android — page never loads

**What goes wrong:**
There are 40+ WP-era photos. Default behavior — dumping them into the page as `<img src>` or even `<Image>` without size discipline — produces a page that's 25+ MB on first load on cheap Android over 3G/4G. LCP > 8s, INP > 500ms, user gives up before hero renders. Ad spend has converted to "negative brand impression."

Specific failure modes:
- All 40 photos rendered eagerly → 25 MB transfer
- Gallery uses `loading="lazy"` but each image is 3 MB → opens lightbox, waits 6s for first photo
- AVIF generated correctly but `<picture>` fallback chain is wrong → old WebView serves JPEG fallback that's 4× larger than necessary
- Hero photo not marked `fetchpriority="high" loading="eager"` → competes with below-fold images for bandwidth, LCP spikes
- `srcset` widths chosen for desktop (1600w, 2400w) → mobile downloads a 1600w image to display in a 360px viewport

**Why it happens:**
- Builder tests on broadband + new MacBook + good Wi-Fi — Lighthouse Mobile throttling doesn't simulate Indonesian network reality precisely
- "Astro handles images" → trust without verification
- 40 photos × 4 widths × 2 formats = 320 generated assets; nobody audits the actual sizes

**How to avoid:**
- **Curate to 12–18 photos maximum** for v1 (per FEATURES.md TS-08). Reject blurry, dark, watermarked, USD-priced, empty-room photos.
- **Pre-process before committing to `src/assets/`:**
  - Hero: 1600×900, target JPEG quality 78, AVIF q70 — output ~80–120 KB
  - Room cards: 800×600 max source, output ~40–80 KB at AVIF q65
  - Gallery thumbs: 400×300 source, output ~20–40 KB
- **Set `widths={[400, 800, 1200]}` on `<Picture>` — do NOT include 1600+ widths for non-hero images. Set `sizes="(max-width: 768px) 100vw, 33vw"` so mobile fetches the 400w variant.
- **Hero `loading="eager" fetchpriority="high"` and below-fold lazy.** Verified pattern from `astro:assets`.
- **Test with Lighthouse Mobile + 4G throttling AND with Chrome DevTools "Slow 3G" preset.** Target LCP <2.5s on 4G, <4s on Slow 3G. INP <200ms.
- **Audit total page weight:** open Network panel, filter "Img", sum sizes. Total image weight on first paint should be <300 KB (one hero + 3 above-fold cards lazy-displayed).
- **For the gallery lightbox:** load thumbnails first; full-res image fetched only on tap. CSS scroll-snap, no JS carousel library.

**Warning signs:**
- `dist/_astro/*.jpg` files >500 KB
- Lighthouse Mobile LCP > 3s
- Real device test (cheap Android, throttled 4G) takes >5s to first paint
- Total transferred bytes on first load > 1 MB

**Phase to address:**
**Phase: Image Pipeline + Performance Verification.** Lighthouse + real-device test on launch checklist.

---

### Pitfall 13: GTM + 3 pixels tank Lighthouse mobile TBT → ad attribution gap

**What goes wrong:**
Even with STACK.md's "inline GTM, no Partytown" recommendation, four pixel networks loading on a budget Android over 4G can push **Total Blocking Time (TBT) past 600ms** and **INP past 300ms**. The page renders fine but is unresponsive for 600ms after main-thread parsing — exactly when the user tries to tap the WA CTA. Tap registers as a delayed click; some ad platform attribution windows close; conversion is lost.

**Why it happens:**
- GTM's async load is fast — but the *tags it injects* (Meta Pixel, TikTok Pixel, Google Ads remarketing) each parse and run their own init JS, each blocking main thread for 50–150ms
- TikTok Pixel specifically is the heaviest of the four (multiple network requests + fingerprinting code)
- Mobile Chromium on cheap Android has ~3× the parse cost of desktop
- The site has minimal first-party JS, so 90% of CPU budget is consumed by pixels

**How to avoid:**
- **Defer the entire GTM `<script>` until `requestIdleCallback` or 2s timeout** — the WA-click conversion event still fires correctly because it pushes to `window.dataLayer` (which exists from page load) and GTM picks up the queued event when it initializes:
  ```html
  <script is:inline>
    window.dataLayer = window.dataLayer || [];
    function loadGTM() {
      var s = document.createElement('script');
      s.src = 'https://www.googletagmanager.com/gtm.js?id=GTM-MK4WJPMF';
      s.async = true;
      document.head.appendChild(s);
    }
    if ('requestIdleCallback' in window) {
      requestIdleCallback(loadGTM, { timeout: 2500 });
    } else {
      setTimeout(loadGTM, 1500);
    }
  </script>
  ```
- **Verify conversion still fires:** with the deferred load, click the WA CTA immediately after page load (before the 2500ms timeout). The event must still be in the `dataLayer` queue when GTM loads. Test with GTM Preview mode — confirm the click event appears in the timeline.
- **Measure on a real cheap Android device** (Redmi 9, Samsung A14, etc.) with 4G — not just Lighthouse synthetic. Use [WebPageTest from Jakarta](https://www.webpagetest.org/) with a mobile Moto G4 profile for realism.
- **If TBT still >300ms post-deferral:** evaluate Partytown despite STACK.md's caution. Run a 24h A/B (deferred-inline vs Partytown) and compare actual Meta Events received per session — that's the only metric that matters.
- **Strip unused GTM tags.** If the existing GTM container has legacy tags from earlier experiments (Hotjar, Microsoft Clarity, etc.), disable them — each adds main-thread cost.

**Warning signs:**
- Lighthouse Mobile TBT >300ms after pixel install
- INP measurements >200ms on real device (Chrome DevTools Performance Insights)
- WA-click conversion rate noticeably lower than expected vs. ad clicks reported by ad platforms
- User session recordings (if owner adds Hotjar later) show rage-clicks on CTA

**Phase to address:**
**Phase: Tracking Implementation + Performance Verification.**

---

### Pitfall 14: Owner editing content later breaks deploys / site goes stale

**What goes wrong:**
Site ships. Owner asks to update the price, change check-in time, add a new photo, fix a typo. Owner has no git/CLI/Astro knowledge. Three failure modes:
1. **Bus factor** — builder is the only one who can change anything; owner reaches out for every typo; builder ghosts after 6 months; site is frozen in time.
2. **Owner attempts edit via Vercel/Cloudflare web UI** — breaks something, can't roll back without builder.
3. **Content drifts from reality** — price changes from Rp 200k to Rp 220k in real life, site still shows 200k for 8 months; first guest argues at check-in.

**Why it happens:**
- Static-site rebuild trades editorial flexibility for performance + simplicity
- Owner's "we don't need a CMS" stance assumes content is forever static (it isn't)
- No handover doc, no edit-request workflow

**How to avoid (in increasing complexity):**
- **Lowest-effort:** centralize all editable content in **one file**, `src/data/property.ts`:
  ```typescript
  export const property = {
    name: 'Jo Guest House',
    pricePerNight: 'Rp 200.000',
    whatsappNumber: '6285108002536',
    checkInTime: '14:00',
    checkOutTime: '12:00',
    distanceToCGK: '±20 menit (jam normal), ±35 menit jam sibuk',
    // ...everything that might change
  };
  ```
  Document in `README.md`: "to update price/hours/distance, edit `src/data/property.ts` and commit. Site auto-deploys."
- **Medium-effort:** owner edits via **GitHub web UI** — log in, navigate to the file, edit, commit. Builder writes a 1-page screenshot guide.
- **Medium-effort + low-cost CMS:** add **Decap CMS** (formerly Netlify CMS) or **TinaCMS** — git-backed, free, gives owner a friendly edit UI without leaving the static-site model. Adds ~1 day of setup. Worth it if owner expects to edit more than monthly.
- **Highest-effort:** rebuild as Astro Content Collections + a server-side admin (Pocketbase / Sanity free tier) — out of scope for v1.
- **Handover documentation:** screen-recorded 5-min video showing how to make a price change. Stored in `.planning/handover/`.
- **Schedule a 30-day review** with owner: confirm what's stale, fix in batch. Don't let owner accumulate 6 months of "small things."

**Warning signs:**
- Owner emails/WhatsApps the builder for every typo
- Site content visibly diverges from reality (price, photos, hours)
- No CMS, no edit guide, builder is single point of failure

**Phase to address:**
**Phase: Handover.** Decap CMS is a v1.x consideration unless content velocity is high. The `property.ts` consolidation is a v1 must.

---

### Pitfall 15: Web3Forms abuse / spam / Indonesian phone validation failure

**What goes wrong:**
The contact form (secondary CTA, FEATURES.md backup) uses Web3Forms. Three failure modes:
1. **Spam flood** — Web3Forms is targeted by spam bots; without captcha, owner's inbox fills with `Re: Increase your sales 1000%` garbage and real WA-fallback leads get buried.
2. **Phone number field accepts garbage** — user types "08123" or "+62812345" or "ya nanti aja"; owner gets unusable lead.
3. **Form submission silently fails** — Web3Forms access key expired / quota hit / network blocked on Indonesian mobile (some Indonesian ISPs block POSTs to certain third-party APIs intermittently); user sees "success" but owner never sees the message.

**Why it happens:**
- Default Web3Forms install has no captcha enabled
- Phone validation is "type=tel" which is non-validating
- No error handling on the fetch failure case
- No success verification — form doesn't tell user "we got it, expect WA reply within X hours"

**How to avoid:**
- **Enable hCaptcha** on the Web3Forms form (zero-config — add `<div class="h-captcha" data-captcha="true"></div>` plus their script). Confirmed in [Web3Forms docs](https://docs.web3forms.com/getting-started/customizations/spam-protection/hcaptcha).
- **Validate Indonesian phone format client-side:**
  ```js
  // Accept: 08XXXXXXXXX (10-13 digits) or +62XXXXXXXXXXX (11-14 digits)
  const idnPhone = /^(\+62|62|0)8[1-9]\d{7,11}$/;
  ```
- **Handle submission failure:** wrap the `fetch` in try/catch; if it throws or returns non-200, show "Pengiriman gagal. Silakan hubungi kami via WhatsApp:" with a WA deep-link as fallback. Never let a failed submission look like a success.
- **Submit-time email check:** Web3Forms supports configured email destinations; verify owner's email actually receives a test submission BEFORE launch (send three test messages from three devices, confirm owner sees them).
- **Set up a notification email** that BCCs the builder for the first 30 days to catch silent failures.
- **De-prioritize the form.** WA is primary (per FEATURES.md). The form sits at the bottom with copy "Lebih cepat via WhatsApp →" linking to wa.me. If <5 form submissions/month after 90 days, drop the form entirely.

**Warning signs:**
- Web3Forms dashboard shows submissions but owner doesn't receive emails → email config issue
- Owner reports "I never get form messages" 2 weeks in → silent failure
- Spam volume > 5/day → captcha not enabled or bypassed

**Phase to address:**
**Phase: Form Integration + Pre-launch QA.**

---

### Pitfall 16: Google Fonts CDN flakiness on Indonesian networks — FOUT/FOIT on hero

**What goes wrong:**
STACK.md recommends self-hosting Plus Jakarta Sans via `@fontsource/plus-jakarta-sans` — correct. If anyone "improves" this by switching back to Google Fonts CDN (because the `<link>` tag is "easier"), the hero LCP regresses by 200–600ms on Indonesian mobile because:
- `fonts.googleapis.com` is occasionally rate-limited or slow from Indonesian ISPs
- Two roundtrips required (CSS, then font binary)
- No `preload` control
- FOIT (invisible text) or FOUT (flash of unstyled text) hits the hero — terrible LCP and CLS

**Why it happens:**
- "Add to head" Google Fonts CDN is the muscle memory
- Self-hosting feels heavier (until you measure)

**How to avoid:**
- **Lock the Fontsource pattern** in STACK.md and the layout component. Comment: `// DO NOT switch to Google Fonts CDN — measured ~400ms LCP regression on ID mobile.`
- **Preload the 400-weight WOFF2** explicitly:
  ```html
  <link rel="preload" as="font" type="font/woff2" crossorigin
        href="/_astro/plus-jakarta-sans-latin-400-normal.woff2">
  ```
- **`font-display: swap`** in the `@font-face` rule so text appears in fallback before custom font loads.
- **Verify in production:** Lighthouse "Avoid enormous network payloads" + "Preload Largest Contentful Paint image" + "Ensure text remains visible during webfont load."
- **Subset to Latin only.** Bahasa uses no diacritics in standard orthography; Latin subset is ~25 KB vs Latin-Ext ~45 KB.

**Warning signs:**
- Network panel shows requests to `fonts.googleapis.com` or `fonts.gstatic.com`
- Lighthouse "FOIT" warning on hero text
- Hero text shifts after ~500ms (CLS spike)

**Phase to address:**
**Phase: Typography & Performance Verification.**

---

### Pitfall 17: Static testimonials read as fake → trust regression vs no testimonials at all

**What goes wrong:**
Per FEATURES.md DF-05, the site includes 3–5 static testimonials. Done badly, they look more dishonest than having none:
- "Pelayanan sangat baik, kamar bersih, harga terjangkau. Sangat direkomendasikan! ★★★★★ — Andi Wijaya"  → reads like AI / Fiverr testimonial farm
- Stock photos for avatars → instant trust collapse
- Five-star uniform inflation → "no critical comments = curated = fake"
- No date, no city, no specific detail → unverifiable
- Western names ("John D. — Singapore") on a Bahasa-primary site → wrong audience signal

Indonesian budget-travel shoppers are highly attuned to OTA-review skepticism (years of fake-review wars) and bring that filter to static testimonials.

**Why it happens:**
- "Filler testimonials" treated as design placeholders, never replaced with real ones
- AI-generated testimonial copy is grammatically perfect and tonally generic — a tell
- 5-star uniformity feels safer than honesty

**How to avoid:**
- **Don't ship testimonials in v1 if owner can't produce real ones.** Empty section > fake section.
- **If shipped, make them obviously specific:**
  - Real first name, city, month (`Mas Andi, Surabaya — April 2026`)
  - Specific detail (`menginap 2 malam menjelang penerbangan pagi ke Singapore`)
  - Mix 4 and 5 star wording / include mild critique (`Kamar agak kecil tapi nyaman, dekat bandara, harga oke`) — 4.7-star average reads as genuine
  - No stock-photo avatars — initials in a circle, or no avatar at all
- **Linked OTA rating callout (DF-06) is safer than testimonials** — it borrows external verification. Use it instead, or in addition.
- **Get real testimonials post-launch:** owner asks 5 happy WA-booked guests for a one-line review, with first-name consent. Add to v1.x.

**Warning signs:**
- All testimonials are 5-star
- All names are Western
- All testimonials are first-month vague ("sangat baik", "recommended", "good place")
- Stock photo avatars (Unsplash faces detectable on TinEye)

**Phase to address:**
**Phase: Content Inventory (v1 decision) + v1.x post-launch testimonial collection.**

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Hardcoded WA number in 5 components instead of central constant | Faster initial scaffold | Number change = 5-place find/replace, easy to miss one | Never — always `src/data/property.ts` from day one |
| Skip the `cutover-checklist.md` document | Saves 30 min of writing | First missing-step incident costs 24h of broken ad spend | Never |
| Use builder's Vercel/Cloudflare account "for now" | No owner onboarding step | Bus factor; ToS violation under Hobby plan; account loss = site loss | Acceptable for 1-week staging; never for production |
| Skip GTM container audit, install new tags | Saves 1 hour | Conversion attribution silently breaks | Never |
| Static testimonials with placeholder copy "to be replaced" | Looks complete in demo | Ships if forgotten — credibility damage | Acceptable only if pre-deploy grep blocks them |
| Single hero image, no responsive `srcset` | One asset to manage | LCP regression on mobile, 4× bandwidth waste | Only on raw thumbnails (sub-100 KB) |
| Inline JSON-LD without validation | Saves 5 minutes | No rich results, no local pack visibility | Never — Rich Results Test is free |
| Skip iOS Safari + IG in-app browser test | Saves 2 hours | 5–15% of paid traffic gets broken WA experience | Never for the primary CTA flow |
| Ship without privacy policy page | Saves 30 min | UU PDP exposure (small) + future EU traffic blocked | Acceptable for 30 days post-launch; add by month 2 |
| Use the existing 40 WP photos without curation | Saves 3 hours of photo selection | Mediocre gallery, large bundle, slow LCP | Never — curate to 12–18 max |
| Old WP site deleted at cutover | Cleaner slate | No rollback, no historical reference, breaks any orphan deep link | Never within 14 days of cutover |
| Hardcode pixel IDs inline instead of central tracking module | Faster to write each tag | Typo risk, hard to swap pixels later, no single source of truth | Never |

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| **WhatsApp `wa.me`** | Number includes `+`, dashes, leading zero (`+62 851-0800-2536` or `085108002536`) | Strip to `6285108002536` — no `+`, no spaces, no dashes, no leading zero |
| **WhatsApp pre-fill** | Newlines as raw `\n` or unencoded `\n` | URL-encode as `%0A`; test rendering on actual devices including in-app browsers |
| **WhatsApp Web (desktop)** | `wa.me` link fails to pre-fill `text` on Safari/Firefox desktop | Use `api.whatsapp.com/send?phone=...&text=...` or platform-detect and rewrite to `web.whatsapp.com/send` |
| **WhatsApp link preview cache** | Sharing URL on WA shows old OG-less preview from WP days | Force re-scrape via [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/) — WA shares FB's cache |
| **GTM (existing container)** | New site uses different event names than existing tags listen for | Audit existing GTM container BEFORE coding; use exact existing event names |
| **GTM Preview Mode** | Verified on staging URL but production uses different container | Always re-verify on `joguesthouse.my.id` post-DNS-cutover |
| **Meta Pixel** | Base code installed, event code (`Lead`/`Contact`) forgotten | Configure event via GTM trigger; confirm in Events Manager → Test Events |
| **TikTok Pixel** | Pixel ID typo; or `track('CompleteRegistration')` instead of `track('Contact')` | Source-of-truth in `src/lib/tracking.ts`; test in TikTok Events |
| **Google Ads conversion** | Conversion label missing from `gtag('event', 'conversion', { send_to: 'AW-XXX/YYY' })` | Always include both ID and label; verify in Google Ads UI |
| **Google Maps iframe** | Embed URL pulled from "Share → Embed a map" but coords don't match property | Use the exact pin on the property building; verify by clicking "Open in Maps" from the iframe |
| **Schema.org `priceRange`** | `"$"` symbol used on an Indonesian site | Use explicit `"Rp 200.000"` |
| **Schema.org `image`** | Relative URL `og-image.jpg` | Absolute `https://joguesthouse.my.id/og-image.jpg` |
| **Vercel/Cloudflare DNS** | TTL left at default 24h, can't roll back quickly | Lower TTL to 300s 48h before cutover |
| **OG image** | Resolution 1200×630 but image cropped by WA preview rendering | Test in actual WhatsApp share preview; central content area within 1200×600 safe zone |
| **`tel:` link** | Number formatted with spaces breaks dialer on some Androids | `tel:+6285108002536` — single `+`, no spaces |
| **Vercel Hobby plan** | Used for commercial site → ToS violation | Cloudflare Pages (free, commercial OK) or Vercel Pro ($20/mo) |
| **Web3Forms** | No captcha enabled → spam flood | Add hCaptcha (zero-config) |
| **Web3Forms** | Owner email not verified post-deploy → submissions vanish | Send 3 test messages from 3 devices, confirm owner receives |

---

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| 40-photo gallery loaded eagerly | LCP > 5s; total transfer > 10 MB | Curate to 12–18 photos; `loading="lazy"` below fold; `<Picture>` with `widths={[400,800,1200]}` | First load on any 4G connection |
| Hero image without `fetchpriority="high"` | LCP regression by 400–800ms | Set `loading="eager" fetchpriority="high"` on hero only | Mobile, throttled networks |
| Google Fonts CDN instead of self-host | FOIT/FOUT, LCP regression, sporadic timeouts on ID networks | `@fontsource/plus-jakarta-sans` + preload | Indonesian mobile networks; varies by ISP |
| All 4 pixels loaded synchronously | TBT > 500ms; INP > 300ms; conversion taps lost | Defer GTM via `requestIdleCallback`; verify dataLayer queue still flushes | Cheap Android + 4G; ~30% of audience |
| Map iframe loaded eagerly | LCP destroyed (+700 KB above fold) | `loading="lazy"`; place below fold | Always — move below fold mandatory |
| GSAP loaded eagerly for sub-fold animations | +30 KB JS blocking before first paint | Dynamic import on `IntersectionObserver` trigger, or skip GSAP entirely for v1 | Below ~500 sessions/day, mostly invisible; above, real |
| JS-heavy carousel library for gallery | +60 KB; lazy load defeated | CSS `scroll-snap-type: x mandatory` — zero JS | Any mobile with slow CPU |
| 1600w `srcset` on cards displayed at 360px | Downloads 8× more pixel data than needed | `widths={[400,800,1200]}` + `sizes` attribute | All mobile traffic |
| Auto-formatting Rp via `Intl.NumberFormat` at runtime | Adds 30+ KB ICU polyfill on older browsers; trailing `,00` | Hardcode `Rp 200.000` as string | Older Android WebView |
| Schema.org as a runtime hydration island | JSON-LD not in initial HTML → not seen by Googlebot first-pass | Inline `<script type="application/ld+json">` in head, build-time | SEO discoverability |
| Vercel Hobby 100 GB bandwidth cap | Site goes 404 mid-month at spike | Cloudflare Pages (unlimited bandwidth) OR Vercel Pro | At ~30k sessions/month with image-heavy page |

---

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Web3Forms access key exposed in source (visible in `view-source`) | Spam abuse against owner's email | Keys are necessarily public for client-side forms — mitigate via hCaptcha; rotate if abused |
| Tracking pixel IDs hardcoded everywhere | Not a security issue but a maintenance/integrity issue | Single `src/lib/tracking.ts` source of truth |
| Building OG image with user-uploaded content | Phishing / XSS via image filename | Not applicable here — OG image is static asset |
| No CSP header | XSS risk; also blocks legitimate GTM if too strict | Permissive CSP allowing `*.googletagmanager.com`, `*.google-analytics.com`, `connect.facebook.net`, `analytics.tiktok.com`, `googleads.g.doubleclick.net`. Add via `vercel.json` / `_headers` |
| `referrerpolicy` not set on outbound OTA links | OTA can't always attribute clicks (minor — they have their own click-out tracking) | `rel="noopener"`, omit `noreferrer` so OTAs can see referrer |
| Map iframe without `sandbox` | Theoretical XSS via Google Maps origin | Google's domain is trusted; sandbox would break the embed. Skip. |
| Privacy policy claims data is encrypted but it isn't | UU PDP risk + reputation | Be honest in privacy page — describe pixel-based tracking, third-party processors, retention |
| Owner shares WA number publicly — fine; but exposing personal phone | Phishing / harassment | The WA number is intentional contact. Use a dedicated business WA (already is `+62 851 0800 2536`) — confirm it's not the owner's personal number |
| GitHub repo public with `.env` committed | Web3Forms key, GTM ID exposed (GTM is meant to be public; key is not sensitive) | None of these are real secrets, but: don't commit `.env`; use Cloudflare/Vercel env vars |
| Sitemap exposing draft / orphan pages | SEO risk, not security | Manually curate `astro.config.mjs` sitemap or use `@astrojs/sitemap` filter |
| Source maps in production | Reveals component structure | Astro doesn't ship source maps by default — verify in `dist/` |
| Pixel cookies set without consent (UU PDP) | Regulatory exposure (low currently) | Privacy policy page + non-blocking disclosure; revisit if EU traffic |

---

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| WhatsApp CTA opens in same tab → user can't return to site | Loses 30–50% who don't know to swipe back | `target="_blank" rel="noopener"` |
| Sticky bottom bar covers content / overlaps last section | Bottom CTA on rooms section unreadable | Bottom 80px padding on `<main>` to compensate for fixed bar height |
| Tap targets <44×44 px (Apple HIG) / <48dp (Material) | Mis-taps, frustration | All CTAs ≥48px square; verify with Chrome DevTools "Tap target size" audit |
| Hero CTA below the fold on 360×640 viewport | Primary action invisible without scroll | Verify hero composition at exactly 360×640 — name + slogan + price + distance + WA CTA all visible |
| WhatsApp pre-fill template too long | Truncated on Android keyboard preview | Keep pre-fill under 200 chars when possible; structured-template version goes to 300, acceptable but test |
| Lightbox gallery without close button at the top | iOS users can't close (no system back) | Explicit `×` close button top-right, 48dp, plus tap-outside-to-close |
| "Buka di Google Maps" link in same tab | Loses user to Google Maps | `target="_blank"` |
| Calling `tel:` link from desktop browser | "What does this do?" confusion | OK to show — desktop browsers either ignore or open Skype/FaceTime |
| Language toggle UI not present + English speaker can't understand any Bahasa | Foreign OTA-referred users bounce | Add small English subtitle ONLY on hero ("Comfortable budget hotel near Soekarno-Hatta airport") — no full toggle (per AF-02) |
| Price shown 3 times with slight differences | Distrust | Display once, prominently, hardcoded |
| FAQ accordion all collapsed by default | Users don't realize there's content | Open first FAQ by default; or render all expanded under 800px viewport |
| Smooth scroll on anchor links + sticky header → header overlaps section top | Section heading hidden under header | `scroll-margin-top: 80px` on each section, or remove sticky header on mobile |
| No "back to top" on long scroll page | Mobile users have to swipe many times | Either a back-to-top button or rely on sticky CTA which gives target above the keyboard |
| Loading spinner on slow image fetch | Doesn't help; users blame the site | Render image placeholder (blurhash or LQIP) + lazy load |
| Form success without confirmation copy | User submits, sees nothing, submits again | Inline success message: "Terima kasih, kami akan balas via WhatsApp dalam X jam." |
| OTA badges placed BEFORE WhatsApp CTA | Sends traffic away to OTAs before owner gets the lead | OTA badges placed AFTER WA CTA (per FEATURES.md TS-09) |
| Hero photo with text overlay unreadable on Android Chrome's low-contrast displays | Hero message unreadable | Test with dark mode, sunlight simulation; ensure text contrast ≥7:1 over hero photo |

---

## "Looks Done But Isn't" Checklist

Pre-launch verification gates. Each item is a thing that frequently ships broken because it "appears" to work in dev.

- [ ] **GTM container loads on production** — open network tab on `joguesthouse.my.id`, search for `gtm.js?id=GTM-MK4WJPMF`, confirm 200 and matches exact ID
- [ ] **All 4 pixel networks fire on PageView** — Meta Pixel Helper, TikTok Pixel Helper, Google Tag Assistant, GTM Preview all green
- [ ] **WhatsApp click event fires in dataLayer AND reaches GTM AND reaches all 4 pixels** — verify in each platform's Test Events
- [ ] **Google Ads "Conversion tracking inactive" warning gone within 24h** — recheck Google Ads UI day after launch
- [ ] **WA pre-fill message renders correctly on Android Chrome** — newlines preserved, no `\n` literals
- [ ] **WA pre-fill works on iOS Safari** — both mobile and desktop
- [ ] **WA pre-fill works in TikTok in-app browser** (ad traffic) — open the URL inside TikTok, tap CTA, verify
- [ ] **WA pre-fill works in Instagram/Facebook in-app browser** — same test
- [ ] **`tel:` link works on Android dialer** — actual tap test
- [ ] **OG card renders correctly when URL shared on WhatsApp** — force re-scrape via FB Debugger, share, check
- [ ] **OG image displays in Twitter/X share preview**
- [ ] **Schema.org Rich Results Test passes with zero errors** — paste production URL
- [ ] **Schema geo coordinates point to the actual building on Google Maps** — copy lat/long, paste in Maps, verify pin
- [ ] **Hero LCP <2.5s on Lighthouse Mobile (4G simulation)**
- [ ] **Hero LCP <4s on real-device test (Redmi/Samsung A1x, Jakarta WebPageTest)**
- [ ] **Total page weight first paint <500 KB** — Network panel sum
- [ ] **No `Lorem`, `ipsum`, `Sample Page`, `My Blog`, `$`, `Add Your` strings in production** — grep `dist/`
- [ ] **Price shows as `Rp 200.000` (dot separator)** — view source check
- [ ] **Distance/time claim to CGK is range-with-traffic, not single fake number**
- [ ] **All testimonials have name+city+month OR section is removed**
- [ ] **OTA badges link out to actual property listings, not generic OTA homepages** — click each
- [ ] **OTA badges use approved logos OR text-only OR generic icons** (no random Google Image grab)
- [ ] **No copy violates OTA rate parity** — grep for "lebih murah", "cheaper than", "harga terbaik"
- [ ] **Web3Forms submission delivered to owner email** — send 3 test messages
- [ ] **hCaptcha visible on form**
- [ ] **Privacy policy page exists and is linked from footer**
- [ ] **301 redirects for known old WP URLs configured** — test each: `curl -I https://joguesthouse.my.id/sample-page/` returns 301
- [ ] **Catch-all 301 for unknown paths → `/`**
- [ ] **Sitemap.xml accessible at `/sitemap-index.xml`** — fetch and validate
- [ ] **`robots.txt` allows crawl, references sitemap**
- [ ] **Google Search Console verified for `joguesthouse.my.id`** — TXT record present
- [ ] **Submit new sitemap in Search Console**
- [ ] **DNS TTL lowered to 300s before cutover, raised back after**
- [ ] **Old WordPress server still alive** (subdomain or IP) for 14-day rollback window
- [ ] **Vercel/Cloudflare project owned by owner's account, not builder's**
- [ ] **GitHub repo collaborator access for owner (read-only minimum)**
- [ ] **Handover document in `.planning/handover/`** with: edit guide, rollback procedure, support contacts
- [ ] **Rollback criteria documented and acknowledged before cutover**
- [ ] **Cutover window scheduled for off-peak hours (Tue 02:00 WIB), builder on standby for 4h post**

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| **Pixel ID typo, conversions broken in production** | LOW (24h to fix and verify) | Identify correct IDs via GTM container audit → fix in `src/lib/tracking.ts` → redeploy → verify in each platform's Test Events; ad platforms backfill conversions within 24–72h but bidding model re-learns over 7–14 days |
| **DNS cutover broke site, no quick rollback** | HIGH | Wait for DNS TTL to expire; meanwhile pause all ad campaigns to stop bleeding; investigate cause; redeploy with fix; resume ads only after verification |
| **Vercel Hobby account suspended for commercial use** | MEDIUM (8h migration) | Deploy same `dist/` to Cloudflare Pages → update DNS (use lowered TTL); communication: owner sees ~30 min downtime |
| **WhatsApp preview cache stuck on old WP OG** | LOW (1h) | Force re-scrape via FB Sharing Debugger 2× → wait 24h for WA cache to refresh → some links may need manual share-and-resave |
| **Old WP URLs 404 → SEO regression** | LOW-MEDIUM (2h) | Add catch-all 301 redirect → submit affected URLs to Search Console "Request indexing" → wait 2–4 weeks for re-crawl |
| **Schema.org rejected by Rich Results Test** | LOW (1h) | Read error, fix specific field, re-test → redeploy → confirm green |
| **Lighthouse mobile LCP >4s** | LOW-MEDIUM (4–8h) | Audit images (sizes, formats, lazy load) → audit JS budget (defer GTM further) → check `fetchpriority` on hero |
| **OTA reports trademark/parity violation** | MEDIUM (1d + relationship) | Remove offending logo/copy → acknowledge with OTA account manager → wait for clearance; potentially lose ranking temporarily |
| **Owner can't edit content** | MEDIUM (1d if Decap CMS retrofit) | Short term: builder edits on owner's behalf; medium term: add Decap CMS to `src/content/` |
| **Form submissions silently failing** | LOW (2h) | Check Web3Forms dashboard → re-verify access key + email destination → send tests → if persistent, switch to Formspree free tier as fallback (50/mo) |
| **Static testimonials called out as fake by an actual reviewer** | MEDIUM (reputation) | Remove section immediately → publish a real-testimonials-only policy in privacy/about copy → collect real ones over 60 days |
| **Site loads slowly in production despite passing Lighthouse on staging** | MEDIUM (1d) | Run WebPageTest from Jakarta, identify the regression delta → likely CSP / GTM / pixel network issue specific to production CDN |
| **Pixel-related ad campaign optimizer degraded** | HIGH (7–14 day relearn at full spend) | Fix the underlying tracking; ad platforms need re-learning window; consider running broader campaigns at lower budget during relearn |

---

## Pitfall-to-Phase Mapping

Implied roadmap phases (derived from STACK.md + FEATURES.md priorities):
- **Phase 0:** Project setup, decision log, hosting choice
- **Phase 1:** Content inventory & owner-verified strings
- **Phase 2:** Design exploration (`/design-direction`)
- **Phase 3:** Layout implementation (Astro components, Tailwind, hero, rooms, facilities, location, gallery, contact)
- **Phase 4:** Image pipeline (curate, optimize, integrate `<Image>`/`<Picture>`)
- **Phase 5:** WhatsApp integration (CTAs, sticky bar, pre-fill templates, UTM)
- **Phase 6:** SEO & Schema (title, meta, OG, JSON-LD, sitemap, robots)
- **Phase 7:** Tracking & GTM (pixel install, conversion events, deferred load, verification)
- **Phase 8:** Form integration (Web3Forms + hCaptcha)
- **Phase 9:** Pre-launch QA (full checklist, Lighthouse, device matrix, Rich Results)
- **Phase 10:** Cutover (DNS, redirects, monitoring window)
- **Phase 11:** Handover (docs, edit guide, owner onboarding)
- **Phase 12:** Post-launch monitoring (Search Console, ad platform diagnostics, rollback readiness)

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| 1. Vercel Hobby ToS violation | Phase 0 | Decision logged in `PROJECT.md`; hosting account in owner's name; commercial-permissive host chosen |
| 2. Pixel IDs swapped / events renamed | Phase 7 (gate before Phase 10) | GTM Preview + Meta/TikTok/Google Test Events all show events live in their respective dashboards |
| 3. DNS cutover with no rollback | Phase 10 | TTL lowered at T-48h; rollback criteria signed off at T-24h; old WP alive 14 days post |
| 4. Old WP URLs orphan 404s | Phase 10 + Phase 12 | `_redirects` file deployed; Search Console "Pages" report shows zero 404s by week 2 |
| 5. WA pre-fill breaks on iOS/desktop/in-app | Phase 5 + Phase 9 | Device matrix test (8+ environments) documented and passed |
| 6. OTA logo / parity violation | Phase 1 + Phase 3 | Trademark/copy audit completed before Phase 3 layout work; OTA badges code-reviewed |
| 7. Lorem Ipsum / placeholder ships | Phase 1 (gate) + Phase 9 (grep) | Content inventory signed off by owner; CI grep blocks placeholder strings |
| 8. Schema.org invalid | Phase 6 + Phase 9 | Rich Results Test passes; geo verified on Google Maps |
| 9. Wrong Rupiah format | Phase 1 + Phase 3 | Price hardcoded as string; grep blocks `$`/`USD` |
| 10. False distance claim | Phase 1 | Owner-verified at 3 traffic times; phrased as range |
| 11. UU PDP cookie policy gap | Phase 6 (privacy page) + Phase 0 (decision) | Privacy page deployed; decision logged |
| 12. Image-heavy gallery kills LCP | Phase 4 + Phase 9 | Lighthouse Mobile LCP <2.5s; WebPageTest from Jakarta <4s |
| 13. Pixel weight tanks TBT/INP | Phase 7 + Phase 9 | TBT <300ms post-pixel install; deferred-load pattern in place |
| 14. Owner can't edit content | Phase 11 | `property.ts` consolidation done; handover guide + video delivered |
| 15. Web3Forms spam / silent failure | Phase 8 + Phase 9 | hCaptcha enabled; 3 test submissions confirmed received by owner |
| 16. Google Fonts CDN flakiness | Phase 4 (typography) | Self-hosted via Fontsource; preload tag verified |
| 17. Static testimonials look fake | Phase 1 (v1 decision) + v1.x post-launch | Testimonials either real or omitted; never placeholder |

---

## Sources

- Vercel ToS / commercial use restrictions:
  - [Vercel Hobby plan documentation](https://vercel.com/docs/plans/hobby)
  - [Vercel Pricing 2026](https://costbench.com/software/developer-tools/vercel/)
  - [Vercel Free Plan 2026 — Hobby Limits & When to Upgrade](https://costbench.com/software/developer-tools/vercel/free-plan/)
- Indonesia UU PDP (Personal Data Protection Law):
  - [Indonesia's PDP Bill Overview — Future of Privacy Forum](https://fpf.org/blog/indonesias-personal-data-protection-bill-overview-key-takeaways-and-context/)
  - [Indonesia Personal Data Protection Law — Securiti](https://securiti.ai/indonesia-personal-data-protection-law/)
  - [PDP Cookie Compliance — CookieHub](https://www.cookiehub.com/pdpl-indonesia)
  - [DLA Piper — Indonesia data protection](https://www.dlapiperdataprotection.com/index.html?t=law&c=ID)
- OTA brand guidelines & rate parity:
  - [Booking.com Brand Guidelines (Booking Holdings)](https://www.bookingholdings.com/wp-content/uploads/2023/07/BHI_BrandGuidelines_ForMediaRoom.pdf)
  - [Booking.com Online Check-in Brand Guidelines](https://checkin.booking.com/hc/en-us/articles/360060330592-3-Brand-Guidelines-)
  - [Agoda Logo Guidelines (official)](https://www.agoda.com/press/agoda-logo-guidelines/)
  - [Hotel Rate Parity — SiteMinder](https://www.siteminder.com/r/hotel-rate-parity/)
  - [Regaining Control Over Direct Pricing — HotelChamp](https://www.hotelchamp.com/blog/bloghow-hotels-can-regain-control-over-direct-pricing)
- WhatsApp click-to-chat cross-browser issues:
  - [How to Fix WhatsApp API on Desktop Browsers — Medium](https://medium.com/@jeanlivino/how-to-fix-whatsapp-api-in-desktop-browsers-fc661b513dc)
  - [Click to Chat link not finding WhatsApp Messenger — copyprogramming](https://copyprogramming.com/howto/how-to-fix-whatsapp-click-to-chat-link-not-finding-whatsapp-messenger-on-phones)
  - [4 Fixes for Links Not Opening in WhatsApp on iPhone and Android](https://www.guidingtech.com/fix-links-not-opening-working-in-whatsapp/)
- WordPress → static migration & SEO:
  - [Google Search Console — Change of Address tool](https://support.google.com/webmasters/answer/9370220?hl=en)
  - [How to Use 301 Redirects When Redesigning or Migrating — ParallelDevs](https://www.paralleldevs.com/blog/how-use-301-redirects-when-redesigning-or-migrating-wordpress-site-without-losing-seo/)
  - [301 Redirection & Site Migration — Wadi's Blog](https://wadidigital.com/blog/301-page-redirection-and-site-migration-never-lose-your-google-status/)
- Google Ads conversion + GTM:
  - [Master Google Ads Enhanced Conversions with GTM — ClickSambo](https://clicksambo.com/blog-detail/google-ads-conversion-tracking-gtm)
  - [Track WhatsApp Conversions in Google Ads — WP Newsify](https://wpnewsify.com/blog/how-to-track-whatsapp-conversions-in-google-ads-full-setup-guide-with-metrics)
  - [Track WhatsApp Button Clicks as Conversions in Google Ads — Lead Ember](https://www.waconversiontracking.com/blog/track-whatsapp-button-clicks-conversions-google-ads)
- Schema.org / Rich Results:
  - [Google Rich Results Test](https://search.google.com/test/rich-results)
  - [Google Search Central — LocalBusiness structured data](https://developers.google.com/search/docs/appearance/structured-data/local-business)
  - [Schema.org LodgingBusiness](https://schema.org/LodgingBusiness)
- Web3Forms:
  - [Web3Forms hCaptcha integration](https://docs.web3forms.com/getting-started/customizations/spam-protection/hcaptcha)
  - [Web3Forms Captcha & SPAM](https://docs.web3forms.com/getting-started/customizations/spam-protection)
- `.my.id` TLD & SEO:
  - [.my.id Domain — Niagahoster](https://www.niagahoster.co.id/blog/domain-my-id/)
  - [Apakah Domain My.id Bagus — Hostnic](https://www.hostnic.id/blog/berita/teknologi/apakah-domain-my-id-bagus-untuk-website-anda-temukan-jawabannya-di-sini/)
- WhatsApp OG share preview / Facebook cache:
  - [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/)

---

*Pitfalls research for: Indonesian budget-guesthouse landing site rebuild (Astro + Tailwind, Vercel/Cloudflare deploy, WordPress cutover with active paid traffic)*
*Researched: 2026-05-12*
