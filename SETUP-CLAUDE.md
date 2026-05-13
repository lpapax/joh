# Claude Code — Full Setup Guide

> **For:** Michal's friend taking over this project.
> **Goal:** Install Claude Code, mirror Michal's plugin stack (tiered — start small, expand later), and set up smart model routing so you don't burn through your Pro-plan token budget.
> **Bilingual:** Every step in **English** then **Bahasa Indonesia**.

---

## Table of contents

0. [Pick how you'll run Claude Code](#0-pick-how-youll-run-claude-code)
1. [Install Claude Code (Terminal)](#1-install-claude-code--terminal-path)
2. [Install Claude Code (Antigravity IDE)](#2-install-claude-code--antigravity-ide-path)
3. [First login + project clone](#3-first-login--clone-the-project)
4. [Plugins — Tier 1 Essential](#4-plugins--tier-1-essential-install-these-first)
5. [Plugins — Tier 2 Recommended](#5-plugins--tier-2-recommended-when-you-want-more)
6. [Plugins — Tier 3 Optional](#6-plugins--tier-3-optional-specialized)
7. [MCP servers](#7-mcp-servers)
8. [Model routing — save Pro-plan tokens](#8-model-routing--save-pro-plan-tokens)
9. [Verify everything works](#9-verify-everything-works)
10. [Troubleshooting](#10-troubleshooting)

---

## 0. Pick how you'll run Claude Code

**EN —** Claude Code is the AI coding agent. You can run it two ways. Pick **one** as your primary; you can add the other later.

| Option | What it is | Best for |
|---|---|---|
| **A · Terminal** (PowerShell + VS Code) | `claude` runs in the terminal, edits files, you watch in VS Code | Power-users, full control, everything in one window |
| **B · Antigravity** (Google's AI IDE) | Antigravity is an editor with Claude Code built in | If you've never used a terminal before — pure GUI experience |

**ID —** Claude Code adalah AI coding agent. Anda bisa menjalankannya dengan 2 cara. Pilih **satu** sebagai utama; yang lain bisa ditambahkan nanti.

| Pilihan | Apa itu | Cocok untuk |
|---|---|---|
| **A · Terminal** (PowerShell + VS Code) | `claude` jalan di terminal, edit file, Anda lihat di VS Code | Power-user, kontrol penuh, semua di satu jendela |
| **B · Antigravity** (Google AI IDE) | Antigravity adalah editor dengan Claude Code di dalamnya | Kalau Anda belum pernah pakai terminal — pure GUI |

> **EN —** Recommended for first-timers: **Option A (terminal)** — it's what Michal uses, all troubleshooting docs assume terminal. Antigravity is fine but newer and less documented.
>
> **ID —** Direkomendasikan untuk pemula: **Opsi A (terminal)** — yang dipakai Michal, semua dokumentasi mengasumsikan terminal. Antigravity OK tapi lebih baru dan dokumentasinya sedikit.

---

## 1. Install Claude Code — Terminal path

**EN —** Prerequisites: Node.js ≥22, Git for Windows, VS Code (see `ONBOARDING.md` section 1 if you haven't done these). Then:

**ID —** Prasyarat: Node.js ≥22, Git for Windows, VS Code (lihat `ONBOARDING.md` bagian 1 kalau belum install). Lalu:

```powershell
# 1. Install Claude Code globally
npm install -g @anthropic-ai/claude-code

# 2. First run — opens browser to log in
claude

# 3. (Optional) Add the VS Code extension
#    VS Code → Extensions (Ctrl+Shift+X) → search "Claude Code" → install
```

**EN —** When the browser asks, sign in with your Anthropic account. If you don't have one yet, sign up at <https://claude.ai/> first — pick the **Pro plan ($20/mo)** so you have enough quota.

**ID —** Saat browser muncul, login pakai akun Anthropic. Kalau belum punya, daftar dulu di <https://claude.ai/> — pilih **paket Pro ($20/bulan)** supaya kuota cukup.

Skip to [section 3](#3-first-login--clone-the-project) once `claude --version` prints a version number.

---

## 2. Install Claude Code — Antigravity IDE path

**EN —** Antigravity is Google's AI-first IDE (free, in beta as of 2026). It bundles Claude Code natively — no separate `npm install` needed.

**ID —** Antigravity adalah IDE AI-first dari Google (gratis, masih beta per 2026). Claude Code sudah built-in — tidak perlu `npm install` terpisah.

### Install steps / Langkah install

1. **EN —** Download from <https://antigravity.google> → **Windows installer**. Run it, accept defaults.
   **ID —** Download dari <https://antigravity.google> → **Windows installer**. Jalankan, terima default.
2. **EN —** Open Antigravity → Settings → **AI Providers** → **Claude (Anthropic)** → sign in with your Anthropic account.
   **ID —** Buka Antigravity → Settings → **AI Providers** → **Claude (Anthropic)** → login pakai akun Anthropic.
3. **EN —** Open this project: **File → Open Folder** → pick the `joh` folder you cloned.
   **ID —** Buka project ini: **File → Open Folder** → pilih folder `joh` yang sudah di-clone.
4. **EN —** Press `Ctrl+Shift+L` to open the Claude panel. Type a question to test.
   **ID —** Tekan `Ctrl+Shift+L` untuk buka panel Claude. Ketik pertanyaan untuk tes.

> **EN —** Antigravity doesn't yet support the full Claude Code plugin/marketplace system (as of 2026-05). If you need plugins/skills like Michal's setup, use **Option A** instead. The two installations can coexist — open Antigravity for chat-style edits, drop into PowerShell for plugin-powered workflows.
>
> **ID —** Antigravity belum mendukung sistem plugin/marketplace Claude Code lengkap (per 2026-05). Kalau butuh plugin/skill seperti setup Michal, pakai **Opsi A**. Keduanya bisa coexist — Antigravity untuk edit chat-style, PowerShell untuk workflow dengan plugin.

---

## 3. First login + clone the project

**EN —** From PowerShell (Option A) or Antigravity's built-in terminal (Option B):

**ID —** Dari PowerShell (Opsi A) atau terminal built-in Antigravity (Opsi B):

```powershell
# 1. Make a projects folder if you don't have one
mkdir $HOME\Projects -ErrorAction SilentlyContinue
cd $HOME\Projects

# 2. Clone the repo
git clone https://github.com/lpapax/joh.git
cd joh

# 3. Open in your editor
code .         # if using VS Code
# OR just stay in Antigravity, it's already open

# 4. Start Claude
claude
```

**EN —** When Claude starts, it auto-reads `CLAUDE.md` in the project root — that's the full project context. You're ready to ask questions.

**ID —** Saat Claude mulai, dia otomatis baca `CLAUDE.md` di root project — itu konteks lengkap project. Anda siap bertanya.

---

## 4. Plugins — Tier 1 Essential (install these first)

**EN —** These 7 plugins are what you actually need for Jo Guest House work (Astro + Tailwind + WordPress + SEO). Install them first — they're enough to be productive.

**ID —** 7 plugin ini yang benar-benar dibutuhkan untuk Jo Guest House (Astro + Tailwind + WordPress + SEO). Install ini dulu — sudah cukup untuk produktif.

### How to install a plugin / Cara install plugin

**EN —** Inside Claude, run `/plugin` and follow the prompts, OR add the marketplace then install the plugin in one go from PowerShell. The commands below are the one-shot PowerShell version.

**ID —** Di dalam Claude, jalankan `/plugin` dan ikuti prompt, ATAU tambah marketplace lalu install plugin sekaligus dari PowerShell. Perintah di bawah adalah versi satu-shot PowerShell.

```powershell
# Start Claude, then run these inside Claude (type them at the prompt):
#   /plugin marketplace add anthropics/claude-plugins-official
#   /plugin marketplace add affaan-m/everything-claude-code
#   /plugin marketplace add alirezarezvani/claude-skills
#   /plugin marketplace add anthropics/knowledge-work-plugins
#   /plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
#   /plugin marketplace add thedotmack/claude-mem
#   /plugin marketplace add nowork-studio/toprank
```

**EN —** After the marketplaces are added, install each plugin with `/plugin install <name>`:

**ID —** Setelah marketplace ditambah, install tiap plugin dengan `/plugin install <name>`:

| Plugin | Marketplace | Why you need it (EN) | Untuk apa (ID) |
|---|---|---|---|
| `superpowers` | claude-plugins-official | Planning, TDD, code review workflows. **The most-used skill set.** | Workflow planning, TDD, code review. **Skill paling sering dipakai.** |
| `frontend-design` | claude-plugins-official | Generate distinctive components, avoid generic AI look | Generate komponen yang unik, hindari tampilan AI generik |
| `ui-ux-pro-max` | ui-ux-pro-max-skill | 50+ styles, 161 color palettes, 99 UX guidelines | 50+ style, 161 palet warna, 99 UX guideline |
| `searchfit-seo` | knowledge-work-plugins | SEO audits, content briefs, schema markup — critical for joguesthouse.my.id | SEO audit, content brief, schema markup — penting untuk joguesthouse.my.id |
| `everything-claude-code` | everything-claude-code | Huge bundle: GSD workflow, code review, build resolvers, language reviewers | Bundle besar: GSD workflow, code review, build resolver, language reviewer |
| `claude-mem` | thedotmack | Persistent memory across sessions — Claude remembers past decisions | Memory persisten antar sesi — Claude ingat keputusan sebelumnya |
| `skill-creator` | claude-plugins-official | Lets you build new skills when you find a gap | Bikin skill baru kalau ada celah |

```text
/plugin install superpowers@claude-plugins-official
/plugin install frontend-design@claude-plugins-official
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
/plugin install searchfit-seo@knowledge-work-plugins
/plugin install everything-claude-code@everything-claude-code
/plugin install claude-mem@thedotmack
/plugin install skill-creator@claude-plugins-official
```

> **EN —** After installing, restart Claude (`Ctrl+C`, then `claude` again). Type `/help` to see new commands.
>
> **ID —** Setelah install, restart Claude (`Ctrl+C`, lalu `claude` lagi). Ketik `/help` untuk lihat command baru.

---

## 5. Plugins — Tier 2 Recommended (when you want more)

**EN —** Add these once you're comfortable. They unlock specialized workflows (image generation, marketing copy, productivity).

**ID —** Tambahkan ini setelah nyaman. Membuka workflow spesifik (generasi gambar, marketing copy, produktivitas).

| Plugin | Marketplace | What it does (EN) | Untuk apa (ID) |
|---|---|---|---|
| `banana-claude` | banana-claude-marketplace | AI image generation via Gemini Nano Banana — for hero/room photos | Generate gambar AI lewat Gemini Nano Banana — untuk foto hero/kamar |
| `toprank` | nowork-studio | Google Ads + Meta Ads + SEO automation, GA/GSC analysis | Otomasi Google Ads + Meta Ads + SEO, analisis GA/GSC |
| `figma` | claude-plugins-official | Read Figma designs, convert to code | Baca desain Figma, convert ke kode |
| `agenthub` | claude-code-skills | Multi-agent orchestration | Orkestrasi multi-agent |
| `engineering` | knowledge-work-plugins | Architecture, deploy checklist, debug, system design | Arsitektur, deploy checklist, debug, system design |
| `design` | knowledge-work-plugins | Design critique, design system, accessibility review | Kritik desain, design system, accessibility review |
| `marketing` | knowledge-work-plugins | Brand review, campaign plan, content creation | Review brand, rencana kampanye, bikin konten |
| `productivity` | knowledge-work-plugins | Task management, memory management | Task management, memory management |

```text
/plugin marketplace add AgriciDaniel/banana-claude
/plugin install banana-claude@banana-claude-marketplace
/plugin install toprank@nowork-studio
/plugin install figma@claude-plugins-official
/plugin install agenthub@claude-code-skills
/plugin install engineering@knowledge-work-plugins
/plugin install design@knowledge-work-plugins
/plugin install marketing@knowledge-work-plugins
/plugin install productivity@knowledge-work-plugins
```

### If you swap WordPress for a headless CMS later / Kalau ganti WordPress ke headless CMS nanti

**EN —** If the owner ever agrees to move off WordPress (or you want a smoother content workflow than Elementor), these two unlock a Sanity + Cloudinary content pipeline. Both are free at this scale.

**ID —** Kalau pemilik setuju pindah dari WordPress (atau Anda ingin workflow konten yang lebih mulus dari Elementor), dua plugin ini buka pipeline Sanity + Cloudinary. Keduanya gratis di skala ini.

| Plugin | Marketplace | What it does (EN) | Untuk apa (ID) |
|---|---|---|---|
| `sanity-plugin` | knowledge-work-plugins | Headless CMS — owner edits rooms / FAQ / pricing in a clean UI, Astro pulls via API | Headless CMS — pemilik edit kamar / FAQ / harga di UI bersih, Astro tarik via API |
| `cloudinary` | knowledge-work-plugins | Image CDN + on-the-fly transforms (avif/webp, resize, crop) for room photos | CDN gambar + transformasi on-the-fly (avif/webp, resize, crop) untuk foto kamar |

```text
/plugin install sanity-plugin@knowledge-work-plugins
/plugin install cloudinary@knowledge-work-plugins
```

> **EN —** Don't install these for v1 — current path is WordPress reskin. Add only if you migrate to Astro and the owner wants a CMS UI.
>
> **ID —** Jangan install untuk v1 — jalur sekarang adalah reskin WordPress. Tambah hanya kalau pindah ke Astro dan pemilik ingin UI CMS.

---

## 6. Plugins — Tier 3 Optional (specialized)

**EN —** Michal has 40+ extra plugins for narrow use-cases (legal, finance, bio-research, customer-support, etc.). Install only if you actually need them — they add slash-command clutter otherwise. Full list reference:

**ID —** Michal punya 40+ plugin tambahan untuk use-case sempit (legal, finance, bio-research, customer-support, dll). Install hanya kalau benar-benar butuh — kalau tidak hanya menambah slash command yang tidak terpakai. Daftar referensi lengkap:

<details>
<summary><b>Full plugin list Michal uses (click to expand)</b></summary>

```text
# anthropics/knowledge-work-plugins (40 plugins):
enterprise-search · cowork-plugin-management · sales · finance · data · legal
marketing · customer-support · product-management · bio-research
slack-by-salesforce · apollo · common-room · engineering · human-resources
design · operations · brand-voice · zoom-plugin · bigdata-com · miro
adspirer-ads-agent · sanity-plugin · zoominfo · mintlify · daloopa · intercom
cockroachdb · prisma · fastly-agent-toolkit · cloudinary · nimble · atlan
product-tracking-skills · figma · adobe-for-creativity · pdf-viewer · box
searchfit-seo

# alirezarezvani/claude-skills:
agenthub · engineering-skills · google-workspace-cli
marketing-skills · self-improving-agent

# coreyhaines31/marketingskills:
marketing-skills

# affaan-m/everything-claude-code:
everything-claude-code  (one big bundle)
```

**EN —** Install any of these the same way: `/plugin install <name>@<marketplace>`.

**ID —** Install salah satu dengan cara yang sama: `/plugin install <name>@<marketplace>`.

</details>

---

## 7. MCP servers

**EN —** MCP (Model Context Protocol) servers give Claude extra superpowers — browser control, live docs lookup, code-graph navigation. Michal's setup uses 5. Install at least the first two for this project.

**ID —** MCP (Model Context Protocol) server memberikan Claude kekuatan ekstra — kontrol browser, cari dokumentasi live, navigasi code-graph. Setup Michal pakai 5. Install minimal 2 yang pertama untuk project ini.

```powershell
# Inside Claude, type these (one at a time):

# REQUIRED for this project:
/mcp add context7 -- npx -y @upstash/context7-mcp
/mcp add puppeteer -- npx -y @modelcontextprotocol/server-puppeteer

# OPTIONAL (Michal also has):
/mcp add 21st-dev-magic -- npx -y @21st-dev/magic
# codegraph + refero need separate accounts — skip for v1
```

| MCP | Purpose (EN) | Untuk apa (ID) |
|---|---|---|
| `context7` | Pulls current docs for any library (React, Astro, Tailwind, etc.) | Ambil dokumentasi terbaru untuk library apapun (React, Astro, Tailwind, dll) |
| `puppeteer` | Browser automation — Claude can open URLs, screenshot, click | Otomasi browser — Claude bisa buka URL, screenshot, klik |
| `21st-dev-magic` | Generate React/Tailwind components from descriptions | Generate komponen React/Tailwind dari deskripsi |

**EN —** Verify with `/mcp` (lists active servers). Restart Claude if anything is missing.

**ID —** Verifikasi dengan `/mcp` (daftar server aktif). Restart Claude kalau ada yang hilang.

---

## 8. Model routing — save Pro-plan tokens

**EN —** The Pro plan has a 5-hour rolling token budget. **Opus burns budget ~5× faster than Sonnet.** Use Opus only when you actually need its reasoning depth; use Sonnet for everything else.

**ID —** Paket Pro punya budget token 5 jam berjalan. **Opus habiskan budget ~5× lebih cepat dari Sonnet.** Pakai Opus hanya saat benar-benar butuh penalaran dalam; pakai Sonnet untuk yang lain.

### Two ways to switch / Dua cara switch

**EN — A. Inline slash command (recommended):**

**ID — A. Slash command inline (direkomendasikan):**

```text
/sonnet         # → switches the active session to Sonnet 4.6 (cheap, fast)
/opus           # → switches the active session to Opus 4.7 (deep reasoning)
/auto           # → shows the routing cheat-sheet — when to use which
```

**EN —** These three slash commands are already in this repo at `.claude/commands/*.md` — they work the moment you clone.

**ID —** Tiga slash command ini sudah ada di repo di `.claude/commands/*.md` — langsung jalan setelah clone.

**EN — B. Default in settings:** edit `~/.claude/settings.json` (your global config, not the repo) and set `"model": "claude-sonnet-4-6"` to default-on-Sonnet. Override per-task with `/opus` when needed.

**ID — B. Default di settings:** edit `~/.claude/settings.json` (config global, bukan repo) dan set `"model": "claude-sonnet-4-6"` untuk default Sonnet. Override per-task dengan `/opus` kalau perlu.

### When to use which / Kapan pakai apa

| Task type | Model | Reason (EN) | Alasan (ID) |
|---|---|---|---|
| Read a file, summarize, list TODOs | **Sonnet** | Pure recall — no reasoning needed | Hanya recall — tidak perlu nalar |
| Rename variables, format code, add comments | **Sonnet** | Mechanical edits | Edit mekanis |
| Run lint, fix simple errors | **Sonnet** | Pattern-match only | Hanya match pattern |
| Write a CSS class, a Tailwind tweak | **Sonnet** | Reference work | Pekerjaan referensi |
| Translate copy EN↔ID | **Sonnet** | Translation = bread-and-butter | Translasi = standar |
| Build a new component from scratch | **Sonnet 4.6** | Sonnet 4.6 is the best coding model | Sonnet 4.6 adalah model coding terbaik |
| Plan a multi-phase feature | **Opus** | Architectural reasoning pays off | Penalaran arsitektur berharga |
| Refactor across many files | **Opus** | Needs full mental model | Butuh model mental penuh |
| Debug a subtle bug with multiple causes | **Opus** | Deep reasoning saves time | Penalaran dalam hemat waktu |
| Critical security review before commit | **Opus** | Cost of a miss > cost of tokens | Biaya miss > biaya token |
| Initial architecture decisions | **Opus** | One-shot, high-leverage | Sekali jadi, high-leverage |

**EN — Rule of thumb:** Start every session on **Sonnet**. Switch to Opus only if you notice Sonnet missing context or oversimplifying — then switch back. **Never leave a session on Opus all day.**

**ID — Aturan praktis:** Mulai setiap sesi di **Sonnet**. Switch ke Opus hanya kalau Sonnet kehilangan konteks atau menyederhanakan — lalu switch balik. **Jangan biarkan sesi di Opus seharian.**

---

## 9. Verify everything works

**EN —** Run these inside Claude in the `joh` folder. Each should return a sensible answer; if not, see Troubleshooting.

**ID —** Jalankan di dalam Claude di folder `joh`. Tiap perintah harus jawab masuk akal; kalau tidak, lihat Troubleshooting.

```text
/help                       # lists all commands — should be 50+ entries
/mcp                         # lists active MCP servers — should include context7, puppeteer
/plugin                      # lists installed plugins — should include 7 Tier-1 plugins
/sonnet                      # switches to Sonnet, confirms model
> Show me what CLAUDE.md says about this project's tracking pixels.
```

**EN —** If the last prompt returns the GTM ID `GTM-MK4WJPMF` and the Meta/TikTok pixel IDs, your setup is fully working.

**ID —** Kalau prompt terakhir menjawab dengan GTM ID `GTM-MK4WJPMF` dan pixel ID Meta/TikTok, setup Anda berfungsi penuh.

---

## 10. Troubleshooting

| Problem (EN) | Masalah (ID) | Fix |
|---|---|---|
| `/plugin marketplace add` fails with `403` | `/plugin marketplace add` gagal `403` | Run `gh auth login` first — some marketplaces require GitHub auth |
| `/plugin install` says "marketplace not found" | "marketplace not found" | Add the marketplace first with `/plugin marketplace add <user>/<repo>` |
| Claude uses Opus even after `/sonnet` | Claude pakai Opus setelah `/sonnet` | `Ctrl+C`, restart Claude, run `/sonnet` again — sessions are sticky |
| MCP server not in `/mcp` list | MCP server tidak muncul di `/mcp` | Restart Claude. If still missing: `/mcp remove <name>` then re-add |
| Pro-plan quota hit fast | Kuota Pro plan cepat habis | You're on Opus. Switch with `/sonnet`. See [section 8](#8-model-routing--save-pro-plan-tokens) |
| Antigravity can't find plugins | Antigravity tidak temukan plugin | Antigravity doesn't (yet) load Claude Code marketplaces — use the terminal install for plugin-heavy work |
| `npm install -g` fails permission error | `npm install -g` gagal permission | Run PowerShell as **Administrator** for the install only — never for running `claude` |

---

## Quick reference / Referensi cepat

```powershell
# === Install (once) ===
npm install -g @anthropic-ai/claude-code
claude                                    # log in
git clone https://github.com/lpapax/joh.git
cd joh
claude                                    # start in project

# === Inside Claude — first-time plugin setup ===
/plugin marketplace add anthropics/claude-plugins-official
/plugin marketplace add affaan-m/everything-claude-code
/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
/plugin marketplace add anthropics/knowledge-work-plugins
/plugin marketplace add thedotmack/claude-mem
/plugin install superpowers@claude-plugins-official
/plugin install frontend-design@claude-plugins-official
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
/plugin install searchfit-seo@knowledge-work-plugins
/plugin install everything-claude-code@everything-claude-code
/plugin install claude-mem@thedotmack
/plugin install skill-creator@claude-plugins-official

# === Inside Claude — MCP setup ===
/mcp add context7 -- npx -y @upstash/context7-mcp
/mcp add puppeteer -- npx -y @modelcontextprotocol/server-puppeteer

# === Daily use ===
/sonnet                    # cheap default
/opus                      # only for hard problems
/auto                      # cheat-sheet for model choice
```

---

*Last updated: 2026-05-13 · For Jo Guest House (`joh`) project · Bilingual (EN / ID)*
