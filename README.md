# Jo Guest House — Website Rebuild

Conversion-focused rebuild of `joguesthouse.my.id` — a 17-room budget guesthouse near Soekarno-Hatta Airport (Jakarta). The current WordPress site burns paid ad spend (TikTok, Meta, Google Ads) on a half-finished page; this repo holds the proposed replacement.

**Status:** prototype designed + planned. WordPress rebuild assignable to new team member.

---

## 30-second overview

| What | Where |
|---|---|
| Working HTML prototype | [`prototype/index.html`](./prototype/index.html) — open in any browser |
| Onboarding for new team member | [`ONBOARDING.md`](./ONBOARDING.md) — bilingual EN / ID |
| Claude Code setup (plugins, MCP, model routing) | [`SETUP-CLAUDE.md`](./SETUP-CLAUDE.md) |
| Full project plan & research | [`.planning/`](./.planning/) — PROJECT, REQUIREMENTS, ROADMAP, research |
| Auto-loaded AI context | [`CLAUDE.md`](./CLAUDE.md) |

## I'm new here — what do I open first?

1. Read **[`ONBOARDING.md`](./ONBOARDING.md)** end-to-end (10 min). It's the canonical start.
2. While you wait for tools to install, skim **[`prototype/index.html`](./prototype/index.html)** in the browser — that's the design target.
3. When Claude Code is installed, jump to **[`SETUP-CLAUDE.md`](./SETUP-CLAUDE.md)** for plugins + model routing.

## Stack

Astro 6 + Tailwind v4 + Vercel (static) for the future rebuild. Current shippable target is reskinning the existing **WordPress + Astra + Elementor** site so ad spend converts immediately. Full decisions in `CLAUDE.md` and `.planning/`.

## Project owner

Michal Pavelec — Pavelec Studio.

---

*Public repo: never commit `.env`, passwords, or DB dumps. See `ONBOARDING.md` security section.*
