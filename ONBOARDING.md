# Jo Guest House — Onboarding Guide

> **For:** Michal's friend taking over the WordPress implementation.
> **From:** the working prototype at `prototype/index.html` in this repo.
> **Status:** brand-new Claude Code install. No prior tooling assumed.
>
> Each step is shown in **English** and then in **Bahasa Indonesia**. Pick whichever you prefer — they say the same thing.

---

## What you'll get by the end of this guide

**EN —** A fully working development setup: Claude Code installed, the project cloned from GitHub, the prototype running locally in your browser, and a clear path to rebuild the design inside the existing WordPress site at `joguesthouse.my.id`.

**ID —** Setup development yang lengkap dan berfungsi: Claude Code terinstall, project sudah di-clone dari GitHub, prototype berjalan di browser lokal Anda, dan jalur yang jelas untuk membangun ulang desain di dalam WordPress yang sudah ada di `joguesthouse.my.id`.

---

## 0. The big picture — what this project is

**EN —** The current WordPress site at `joguesthouse.my.id` is half-finished (default title `"My Blog – My WordPress Blog"`, lorem ipsum room descriptions, USD prices on an Indonesian budget hotel). The owner is spending money on TikTok / Meta / Google Ads sending traffic to that broken page. The prototype in this repo (`prototype/index.html`) is the proposed replacement design — real Bahasa content, WhatsApp-first booking, real Rp 200.000 pricing, and proximity-to-airport as the lead. Your job is to rebuild that design inside WordPress (or Astro — your choice) so the ad spend converts.

**ID —** Website WordPress saat ini di `joguesthouse.my.id` masih setengah jadi (judul default `"My Blog – My WordPress Blog"`, deskripsi kamar lorem ipsum, harga dalam USD untuk hotel budget Indonesia). Pemilik mengeluarkan uang untuk TikTok / Meta / Google Ads yang mengarahkan trafik ke halaman yang rusak itu. Prototype di repo ini (`prototype/index.html`) adalah desain pengganti yang diusulkan — konten Bahasa asli, booking lewat WhatsApp, harga asli Rp 200.000, dan kedekatan ke bandara sebagai fokus utama. Tugas Anda adalah membangun ulang desain itu di dalam WordPress (atau Astro — pilihan Anda) supaya iklan yang berjalan benar-benar menghasilkan booking.

---

## 1. Install the prerequisites

Three things must be installed before Claude Code can do anything useful. Open the **Start menu**, type each link below, and download from the **official** site.

**EN — required tools:**

| # | Tool | Why | Download |
|---|------|-----|----------|
| 1 | **Node.js LTS** (v22 or later) | Runtime for Claude Code, npm, Astro | <https://nodejs.org/> → green LTS button |
| 2 | **Git for Windows** | Version control + GitHub access | <https://git-scm.com/download/win> |
| 3 | **Visual Studio Code** | Code editor where Claude Code lives | <https://code.visualstudio.com/> |

**ID — alat yang wajib di-install:**

| # | Alat | Untuk apa | Link download |
|---|------|-----------|---------------|
| 1 | **Node.js LTS** (v22 atau lebih baru) | Runtime untuk Claude Code, npm, Astro | <https://nodejs.org/> → tombol hijau LTS |
| 2 | **Git for Windows** | Version control + akses GitHub | <https://git-scm.com/download/win> |
| 3 | **Visual Studio Code** | Editor tempat Claude Code berjalan | <https://code.visualstudio.com/> |

### Installation tips / Tips instalasi

**EN —** During Node.js install, accept defaults. During Git install, on the **"Adjusting your PATH environment"** screen, pick **"Git from the command line and also from 3rd-party software"** (the middle option). Everything else: defaults. Restart your PC after all three are installed so PATH variables refresh.

**ID —** Saat install Node.js, terima semua default. Saat install Git, di layar **"Adjusting your PATH environment"**, pilih **"Git from the command line and also from 3rd-party software"** (opsi tengah). Yang lain: default. Restart PC setelah ketiganya selesai supaya variabel PATH ter-refresh.

### Verify the install / Verifikasi instalasi

**EN —** Open **PowerShell** (Start menu → type "PowerShell") and run:

**ID —** Buka **PowerShell** (Start menu → ketik "PowerShell") lalu jalankan:

```powershell
node --version    # expected: v22.x or higher
npm --version     # expected: 10.x or higher
git --version    # expected: 2.x or higher
code --version    # expected: 1.x or higher
```

If any command says *"not recognized"*, that tool didn't install correctly — re-install it or restart your PC. / Kalau ada perintah yang bilang *"not recognized"*, alat itu tidak terinstall dengan benar — install ulang atau restart PC.

---

## 2. Install Claude Code

**EN —** Claude Code is Anthropic's AI coding agent. It runs inside your terminal and inside VS Code. Install it globally with npm:

**ID —** Claude Code adalah AI coding agent dari Anthropic. Dia berjalan di terminal Anda dan di dalam VS Code. Install secara global dengan npm:

```powershell
npm install -g @anthropic-ai/claude-code
```

**EN —** After install, log in. This opens a browser to authorize:

**ID —** Setelah install, login. Perintah ini akan membuka browser untuk otorisasi:

```powershell
claude
```

**EN —** Follow the browser prompt → log in with your Anthropic account (or sign up if you don't have one — there's a free tier and a $20/mo Pro plan). When the browser says "you can return to your terminal", press Enter.

**ID —** Ikuti browser → login pakai akun Anthropic Anda (atau daftar kalau belum punya — ada tier gratis dan paket Pro $20/bulan). Saat browser bilang "you can return to your terminal", tekan Enter.

### Install the VS Code extension / Install ekstensi VS Code

**EN —** In VS Code, open the Extensions panel (`Ctrl+Shift+X`), search for **"Claude Code"**, install the official one by Anthropic. Restart VS Code.

**ID —** Di VS Code, buka panel Extensions (`Ctrl+Shift+X`), cari **"Claude Code"**, install yang official dari Anthropic. Restart VS Code.

---

## 3. Get the project from GitHub

**EN —** This repo lives at: **`https://github.com/lpapax/jo-guest-house`** *(Michal will confirm the exact URL after he pushes — see "Owner's push checklist" at the bottom of this file.)*

**ID —** Repo ini ada di: **`https://github.com/lpapax/jo-guest-house`** *(Michal akan konfirmasi URL pastinya setelah dia push — lihat "Owner's push checklist" di bagian bawah file ini.)*

### Clone it / Clone-nya

**EN —** Pick a folder where your projects live (e.g. `C:\Users\YourName\Projects\`) and clone:

**ID —** Pilih folder untuk semua project Anda (mis. `C:\Users\NamaAnda\Projects\`) lalu clone:

```powershell
mkdir $HOME\Projects
cd $HOME\Projects
git clone https://github.com/lpapax/jo-guest-house.git
cd jo-guest-house
code .
```

**EN —** The last command opens this whole project in VS Code. You should see folders `prototype/`, `.planning/`, and a few `.md` files.

**ID —** Perintah terakhir membuka seluruh project di VS Code. Anda akan melihat folder `prototype/`, `.planning/`, dan beberapa file `.md`.

### What each folder contains / Isi tiap folder

| Folder | EN | ID |
|--------|----|----|
| `prototype/` | The working HTML preview — open `index.html` | Preview HTML yang berfungsi — buka `index.html` |
| `.planning/` | Project plan, requirements, roadmap, deep research | Rencana project, requirements, roadmap, riset |
| `ONBOARDING.md` | This file | File ini |
| `CLAUDE.md` | Auto-loaded instructions for Claude Code | Instruksi otomatis untuk Claude Code |

---

## 4. Run the prototype locally

**EN —** Two ways to view `prototype/index.html`:

**ID —** Dua cara untuk melihat `prototype/index.html`:

### Option A — just open the file / Opsi A — langsung buka file

**EN —** Double-click `prototype/index.html` in File Explorer. The browser opens it directly. ✓ Done.

**ID —** Klik dua kali `prototype/index.html` di File Explorer. Browser akan membukanya langsung. ✓ Selesai.

### Option B — run a local server (better) / Opsi B — jalankan server lokal (lebih baik)

**EN —** Some browser features (form submission, Google Maps iframe) work better over `http://`. Run a one-line server in PowerShell from inside the prototype folder:

**ID —** Beberapa fitur browser (submit form, iframe Google Maps) bekerja lebih baik lewat `http://`. Jalankan server satu baris di PowerShell dari dalam folder prototype:

```powershell
cd prototype
python -m http.server 5173
```

**EN —** Then open <http://localhost:5173> in your browser. If you don't have Python: `npx serve` works the same.

**ID —** Lalu buka <http://localhost:5173> di browser Anda. Kalau tidak punya Python: `npx serve` juga bisa.

---

## 5. Use Claude Code with this project

**EN —** Open PowerShell, navigate to the project, and start Claude:

**ID —** Buka PowerShell, masuk ke folder project, lalu start Claude:

```powershell
cd $HOME\Projects\jo-guest-house
claude
```

**EN —** When Claude starts it auto-reads `CLAUDE.md` in the project root. That file already contains the full context (what the project is, what stack, what's done, what's next). You can immediately ask things like:

**ID —** Saat Claude mulai, dia otomatis membaca `CLAUDE.md` di root project. File itu sudah berisi konteks lengkap (project ini apa, stack-nya apa, sudah selesai apa, selanjutnya apa). Anda bisa langsung tanya seperti:

```
> show me the prototype design — open it in puppeteer and screenshot the hero
> what's in .planning/ROADMAP.md — summarize phase 3
> help me rebuild the prototype hero in WordPress Elementor
> how do I install the WhatsApp button code into Elementor's custom HTML widget
```

### Optional: install MCP servers / Opsional: install MCP server

**EN —** MCP servers give Claude extra tools (browser control, docs lookup, etc.). Two that are useful for this project:

**ID —** MCP server memberikan Claude tool tambahan (kontrol browser, pencarian dokumentasi, dll). Dua yang berguna untuk project ini:

```powershell
# Context7 - up-to-date library/framework docs (useful when adapting code)
claude mcp add context7 -- npx -y @upstash/context7-mcp

# Puppeteer - browser automation, screenshots
claude mcp add puppeteer -- npx -y @modelcontextprotocol/server-puppeteer
```

**EN —** After adding, restart Claude (Ctrl+C, then `claude` again). Type `/mcp` inside Claude to verify they're loaded.

**ID —** Setelah ditambahkan, restart Claude (Ctrl+C, lalu `claude` lagi). Ketik `/mcp` di dalam Claude untuk memastikan keduanya sudah ter-load.

---

## 6. Implement the design in WordPress

**EN —** The existing site uses **WordPress 6.9+ with the Astra theme and Elementor page builder**. The cleanest path is to mirror the prototype section-by-section in Elementor, pulling colors / fonts / copy from `prototype/index.html`. You don't need to delete the existing WordPress — you replace the page content section by section.

**ID —** Site yang ada pakai **WordPress 6.9+ dengan tema Astra dan page builder Elementor**. Cara paling bersih adalah meniru prototype bagian-per-bagian di Elementor, mengambil warna / font / copy dari `prototype/index.html`. Tidak perlu hapus WordPress yang ada — Anda mengganti konten halaman bagian per bagian.

### Step-by-step / Langkah demi langkah

**EN —**
1. Get WordPress admin access from the owner (`https://joguesthouse.my.id/wp-admin/`).
2. Install **Astra Pro** (or stay on free Astra — both work) and confirm **Elementor** is active.
3. In **Appearance → Customize → Global Colors**, set the palette from the prototype:
   - Background: `#FAF7F2` (paper / cream)
   - Text: `#0B0B0B` (ink)
   - Accent: `#D97757` (terracotta)
   - WhatsApp: `#25D366`
4. **Appearance → Customize → Typography**, set body font to **Plus Jakarta Sans** (already in Google Fonts).
5. Open the homepage in Elementor. Delete the existing lorem ipsum sections one at a time.
6. For each section of the prototype, create a matching Elementor section:
   - **Hero** → Elementor Section with background image, headline, buttons widget
   - **Trust bar** → Inner Section, Icon List widgets in a row
   - **Rooms** → Posts widget OR 3 manual Card sections
   - **Facilities** → Icon Box widgets in a 3×2 grid
   - **Location** → Google Maps widget + Text widget for address + Buttons
   - **FAQ** → Toggle widget (built into Elementor)
   - **Contact** → Form widget (WPForms ID 336 still exists)
   - **Sticky mobile CTA** → Elementor's "Sticky" option on a footer bar OR an HTML widget with the code from `prototype/index.html` lines 392–407
7. **Fix the broken basics first** (this is the highest ROI, takes 10 minutes):
   - **Settings → General → Site Title**: change `"My Blog"` to `"Jo Guest House"`
   - **Settings → General → Tagline**: change `"My WordPress Blog"` to `"Penginapan Murah Dekat Bandara Soekarno-Hatta"`
   - **Yoast SEO plugin** (install if missing) → set the homepage title template and meta description in Bahasa
8. Test on mobile (Elementor preview → mobile icon) before publishing.
9. Confirm WhatsApp deep-links use `api.whatsapp.com/send?phone=6285108002536` (works on iOS Safari + TikTok/Instagram in-app browsers — `wa.me` does NOT, and the current site is using `wa.me`).
10. Verify tracking pixels still fire — `TikTok Pixel`, `Meta Pixel`, `Google Tag Manager (GTM-MK4WJPMF)`, `Google Ads (AW-17438288457)` are already installed via plugins; do **not** remove them.

**ID —**
1. Dapatkan akses admin WordPress dari pemilik (`https://joguesthouse.my.id/wp-admin/`).
2. Install **Astra Pro** (atau pakai Astra gratis — sama-sama bisa) dan pastikan **Elementor** aktif.
3. Di **Appearance → Customize → Global Colors**, atur palette dari prototype:
   - Background: `#FAF7F2` (paper / cream)
   - Teks: `#0B0B0B` (ink)
   - Aksen: `#D97757` (terakota)
   - WhatsApp: `#25D366`
4. **Appearance → Customize → Typography**, set font body ke **Plus Jakarta Sans** (sudah ada di Google Fonts).
5. Buka homepage di Elementor. Hapus section lorem ipsum yang ada satu per satu.
6. Untuk setiap section prototype, buat section Elementor yang cocok:
   - **Hero** → Elementor Section dengan background image, heading, buttons widget
   - **Trust bar** → Inner Section, Icon List widget dalam satu baris
   - **Rooms** → Posts widget ATAU 3 Card sections manual
   - **Facilities** → Icon Box widgets dalam grid 3×2
   - **Location** → Google Maps widget + Text widget untuk alamat + Buttons
   - **FAQ** → Toggle widget (built-in di Elementor)
   - **Contact** → Form widget (WPForms ID 336 masih ada)
   - **Sticky mobile CTA** → Opsi "Sticky" Elementor di footer bar ATAU HTML widget dengan kode dari `prototype/index.html` baris 392–407
7. **Perbaiki yang basic dulu** (ROI tertinggi, 10 menit):
   - **Settings → General → Site Title**: ubah `"My Blog"` jadi `"Jo Guest House"`
   - **Settings → General → Tagline**: ubah `"My WordPress Blog"` jadi `"Penginapan Murah Dekat Bandara Soekarno-Hatta"`
   - **Plugin Yoast SEO** (install kalau belum) → atur template judul homepage dan meta description dalam Bahasa
8. Test di mobile (preview Elementor → ikon mobile) sebelum publish.
9. Pastikan WhatsApp deep-link pakai `api.whatsapp.com/send?phone=6285108002536` (bekerja di iOS Safari + browser dalam TikTok/Instagram — `wa.me` TIDAK, dan site sekarang masih pakai `wa.me`).
10. Verifikasi tracking pixel masih jalan — `TikTok Pixel`, `Meta Pixel`, `Google Tag Manager (GTM-MK4WJPMF)`, `Google Ads (AW-17438288457)` sudah terpasang lewat plugin; **jangan** hapus.

### Asking Claude for help inside this workflow / Minta bantuan Claude di workflow ini

**EN —** Once Claude Code is running in the project folder, you can ask things like:

**ID —** Setelah Claude Code berjalan di folder project, Anda bisa bertanya seperti:

```
> open prototype/index.html and tell me which Tailwind colors map to Elementor global colors
> generate the JSON-LD LodgingBusiness markup for the WP custom-fields plugin
> what Elementor widget combination replicates the Rooms section?
> compare the existing wp-admin homepage to prototype/index.html — what's missing?
```

---

## 7. Optional — rebuild on Astro instead of WordPress

**EN —** If you ever want to ditch WordPress entirely (faster, cheaper, more flexible), the full `.planning/ROADMAP.md` lays out 6 phases for that path: Astro 6 + Tailwind v4 + Cloudflare Pages. Phases 3–5 turn this prototype into a real static site. You don't need to take that route unless the owner agrees — for now, the WordPress path is faster.

**ID —** Kalau Anda ingin meninggalkan WordPress sepenuhnya (lebih cepat, lebih murah, lebih fleksibel), `.planning/ROADMAP.md` punya 6 fase untuk jalur itu: Astro 6 + Tailwind v4 + Cloudflare Pages. Fase 3–5 mengubah prototype ini jadi static site sungguhan. Tidak perlu ambil jalan itu kecuali pemilik setuju — untuk sekarang, jalur WordPress lebih cepat.

---

## 8. Troubleshooting / Pemecahan masalah

| Problem (EN) | Masalah (ID) | Fix |
|---|---|---|
| `node` not recognized | `node` tidak dikenali | Restart PC after Node install; PATH didn't refresh |
| `claude` command hangs | Perintah `claude` mandek | Press Ctrl+C, run `claude` again; if login expired re-auth |
| Browser shows broken images in prototype | Browser tampilkan gambar rusak | Internet must be online — photos are hot-linked from joguesthouse.my.id |
| Form doesn't submit | Form tidak ter-submit | Form opens WhatsApp directly — that's intended behavior, not a bug |
| `git push` asks for password | `git push` minta password | GitHub no longer accepts passwords — run `gh auth login` and pick HTTPS |
| Emoji icons render as boxes | Ikon emoji jadi kotak | Headless Chromium quirk only — real browsers render correctly. Production will swap to SVG |

---

## 9. Owner's push checklist (Michal does this)

**EN —** Before sending this guide, Michal needs to push the local repo to GitHub:

**ID —** Sebelum mengirim guide ini, Michal perlu push repo lokal ke GitHub:

```powershell
# 1. authenticate (opens browser)
gh auth login
# choose: GitHub.com → HTTPS → Login with a web browser

# 2. create the repo and push
cd "C:\Users\micha\OneDrive - Univerzita Palackého v Olomouci\Plocha\Jonathan"
gh repo create jo-guest-house --public --source=. --remote=origin --push --description "Jo Guest House — landing site rebuild (prototype + plan)"

# 3. verify
gh repo view --web
```

**EN —** After that the URL `https://github.com/lpapax/jo-guest-house` works — replace any placeholders in this file if your GitHub username is different.

**ID —** Setelah itu URL `https://github.com/lpapax/jo-guest-house` aktif — ganti placeholder di file ini kalau username GitHub Anda berbeda.

---

## 10. Quick reference / Referensi cepat

```powershell
# Install everything (once)
node --version            # confirm prerequisite
npm install -g @anthropic-ai/claude-code

# Clone the project (once)
git clone https://github.com/lpapax/jo-guest-house.git
cd jo-guest-house

# Open in editor + Claude Code
code .
claude

# Run the prototype locally
cd prototype
python -m http.server 5173
# → open http://localhost:5173

# Pull updates from Michal
cd jo-guest-house
git pull

# Push your own changes
git add .
git commit -m "feat(wp): rebuilt hero section in Elementor"
git push
```

---

## You're set / Anda sudah siap

**EN —** Open VS Code, type `claude`, and ask: *"Show me what's in `.planning/ROADMAP.md` and what I should do next."* Claude will read the project context and walk you through the work. Anything unclear, ping Michal on WhatsApp.

**ID —** Buka VS Code, ketik `claude`, dan tanya: *"Show me what's in `.planning/ROADMAP.md` and what I should do next."* Claude akan baca konteks project dan menuntun Anda. Kalau ada yang belum jelas, hubungi Michal via WhatsApp.

---

*Last updated: 2026-05-12 · Generated for handover · Bilingual (EN / ID)*
