# Project State: Jo Guest House — Website Rebuild

**Initialized:** 2026-05-12
**Mode:** yolo
**Granularity:** coarse

## Project Reference

**Core Value:** Turn existing TikTok / Meta / Google Ads paid traffic into actual WhatsApp bookings — converting better than the current half-finished WordPress site.

**Current Focus:** Roadmap created. Ready to begin Phase 1 (Setup + Key Decisions): hosting choice and design direction must be locked before any code is written.

## Current Position

- **Phase:** 1 — Setup + Key Decisions (not started)
- **Plan:** None (Phase 1 not yet planned)
- **Status:** Roadmap created; awaiting `/gsd-plan-phase 1`
- **Progress:** ░░░░░░░░░░░░░░░░░░░░ 0 / 6 phases

## Performance Metrics

| Metric | Target | Current |
|--------|--------|---------|
| Phases complete | 6 | 0 |
| v1 requirements shipped | 66 | 0 |
| Lighthouse Mobile Performance | ≥ 90 | — |
| Lighthouse Mobile SEO | ≥ 95 | — |
| LCP (Slow 4G, Moto G4) | < 2.5s | — |
| First-party JS budget | < 50 KB | — |
| All 4 pixels firing on prod | yes | no |

## Accumulated Context

### Decisions Logged

| Decision | Rationale | Source |
|----------|-----------|--------|
| 6-phase horizontal-layer structure | Cutover-day constraint forces horizontal layers (setup → content → foundation → primitive → sections → cutover); no incremental user-value slicing applies to a marketing-site rebuild | ROADMAP.md |
| Hard gate: owner content sign-off between P2 and P3 | Lorem Ipsum / USD-price ship-failure is the project's #1 risk per PITFALLS.md; only owner sign-off mitigates it | ROADMAP.md, SUMMARY.md |
| Verifier gate between P5 and P6 | `workflow.verifier=true` + pixel mis-install on cutover = 7–14 day ad-bidding re-training cost | ROADMAP.md, config.json |

### Open Decisions (Phase 1 work)

- Hosting: Cloudflare Pages (free, commercial-OK) vs Vercel Pro (USD 20/mo) — Vercel Hobby is rejected (ToS violation)
- Visual design direction — generate 3 via `/design-direction`, owner picks one
- Brand tokens (colors, typography, spacing) — finalize from chosen direction

### Todos

- [ ] Plan Phase 1 (`/gsd-plan-phase 1`)
- [ ] Owner: confirm hosting choice (Cloudflare Pages default unless preference stated)
- [ ] Owner: confirm Web3Forms email destination address (gap from research)
- [ ] Owner / builder: capture exact 6-decimal lat/lng of building pin on Google Maps (gap from research)
- [ ] Owner: share GTM container access for pre-Phase-3 audit (gap from research)
- [ ] Owner: confirm OTA listing URLs are live (Agoda, Traveloka, Booking.com)

### Blockers

None currently. Phase 1 can start as soon as owner is available for the design-direction selection.

### Known Gaps (from research SUMMARY.md)

1. Exact GTM container configuration unknown until owner shares access
2. Exact geo-coordinates unverified (placeholder lat: -6.165, lng: 106.738 in research notes)
3. OTA listing URLs not yet confirmed live
4. Owner email for Web3Forms not yet collected
5. Design direction not yet chosen — Phase 1 resolves this

## Session Continuity

**Last action:** Roadmap created with 6 phases, 66/66 requirements mapped, hard gates documented.

**Next action:** Run `/gsd-plan-phase 1` to decompose Phase 1 (Setup + Key Decisions) into executable plans.

**Files of record:**
- `.planning/PROJECT.md` — core value, constraints, key decisions
- `.planning/REQUIREMENTS.md` — 66 v1 REQ-IDs with traceability table
- `.planning/ROADMAP.md` — 6-phase structure with success criteria
- `.planning/research/SUMMARY.md` — cross-cutting findings, phase suggestions
- `.planning/research/{STACK,FEATURES,ARCHITECTURE,PITFALLS}.md` — deep research
- `.planning/config.json` — granularity=coarse, mode=yolo, verifier=true

---
*State initialized: 2026-05-12*
*Last updated: 2026-05-12*
