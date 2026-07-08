## for-you-always

> **Aldo** adalah Solo Founder dari **For you, Always.** — sebuah brand digital yang menjual produk kado & surat interaktif premium. Aldo mengatur seluruh visi, arah produk, dan estetika desain.

# 🤖 PANDUAN ONBOARDING — AI AGENT
### Platform: **For you, Always.** — Digital Atelier
### Dokumen ini wajib dibaca tuntas sebelum menyentuh kode apapun.

---

## 1. IDENTITAS & ALUR KERJA

### Siapa yang kamu bantu?
**Aldo** adalah Solo Founder dari **For you, Always.** — sebuah brand digital yang menjual produk kado & surat interaktif premium. Aldo mengatur seluruh visi, arah produk, dan estetika desain.

**Aldo tidak melakukan coding sendiri.** Tugas kamu sebagai AI Agent adalah murni sebagai **eksekutor teknis**: membaca instruksi Aldo, memahami konteks codebase, lalu menulis/mengubah kode sesuai permintaan tanpa merusak hal lain.

### Prinsip Kerja Wajib
- JANGAN menghapus atau merombak kode yang tidak diminta.
- JANGAN mengambil keputusan besar (arsitektur, hapus produk) tanpa konfirmasi eksplisit dari Aldo.
- SELALU cek `git log` sebelum membuat perubahan besar — Aldo sangat memperhatikan git history dan sering minta revert ke commit tertentu.
- SELALU lakukan `git add . && git commit -m "..." && git push origin main` setelah setiap perubahan selesai.
- Jika ada kebingungan atau instruksi ambigu, tanya dulu — jangan asumsi sendiri.
- Pastikan perubahan yang diminta sudah benar-benar terefleksi di file sebelum commit.

### Gaya Komunikasi
Aldo berbicara dalam **Bahasa Indonesia** santai. Balas juga dalam Bahasa Indonesia, ringkas, dan langsung ke poin.

---

## 2. ARSITEKTUR PLATFORM — GAMBARAN BESAR

Platform ini terdiri dari **dua lapisan utama**:

```
+----------------------------------------------------------+
|  LAYER 1: Valentine-Platform (Next.js)                   |
|  Landing page, katalog produk, checkout flow             |
|  Domain: for-you-always.my.id                            |
+---------------------------+------------------------------+
                            | user membeli -> redirect ke produk
                            v
+----------------------------------------------------------+
|  LAYER 2: Produk Individual (Vanilla JS / Next.js)       |
|  Setiap produk punya repo & subdomain sendiri            |
|  Contoh: letter.for-you-always.my.id                     |
+----------------------------------------------------------+
                            ^
                            |
+----------------------------------------------------------+
|  pakasir-gateway (Cloudflare Worker)                     |
|  Payment gateway terpusat - menghubungkan semua produk   |
+----------------------------------------------------------+
```

---

## 3. PEMETAAN FOLDER PROJECT

Semua folder proyek berada di: `C:\Users\aldor\OneDrive\Desktop\`

---

### [Valentine-Platform/] — INDUK UTAMA (Next.js 14, TypeScript)
> INI ADALAH REPO YANG PALING SERING DIUBAH. Fokus utama kamu.

Deploy  : for-you-always.my.id
Stack   : Next.js 14 (App Router), TypeScript, Vanilla CSS, Vercel

#### Struktur Internal:

```
Valentine-Platform/
├── app/
│   ├── layout.tsx              <- Root layout: font, metadata SEO, schema.org, analytics
│   ├── globals.css             <- Global CSS variables & resets
│   ├── robots.ts               <- SEO robots config
│   ├── sitemap.ts              <- Sitemap otomatis
│   ├── (landing)/              <- Semua halaman publik (route group, tidak muncul di URL)
│   │   ├── page.tsx            <- HOMEPAGE UTAMA — sangat besar (~1100 baris)
│   │   ├── landing.css         <- CSS khusus landing page
│   │   ├── layout.tsx          <- Layout wrapper untuk semua halaman landing
│   │   ├── LetterEnvelopePreview.tsx  <- Komponen preview amplop interaktif
│   │   ├── catalog/
│   │   │   ├── page.tsx        <- Halaman /catalog — grid semua produk
│   │   │   ├── letter/         <- Halaman detail: Letter Edition
│   │   │   ├── voices/         <- Halaman detail: Voices Gift
│   │   │   ├── mixtape/        <- Halaman detail: Mixtape Edition
│   │   │   ├── invitation/     <- Halaman detail: Invitation Edition
│   │   │   ├── arcade/         <- Halaman detail: Arcade Edition
│   │   │   ├── retro/          <- Halaman detail: Retro Edition
│   │   │   ├── wrapped/        <- Halaman detail: Wrapped Edition
│   │   │   └── memoria/        <- Halaman detail: Memoria (Premium)
│   │   ├── letter/             <- Landing page khusus Letter Edition (bukan katalog)
│   │   ├── voices/             <- Landing page khusus Voices Gift
│   │   ├── arcade/             <- Landing page khusus Arcade Edition
│   │   ├── wrapped/            <- Landing page khusus Wrapped Edition
│   │   ├── bundle/             <- Halaman paket bundle produk
│   │   └── order-status/       <- Halaman konfirmasi status pembayaran pasca checkout
│   ├── components/             <- Komponen React reusable
│   │   ├── LandscapeProductCard.tsx  <- Kartu produk landscape (TERKOMPLEKS, ~800 baris)
│   │   ├── CompactProductCard.tsx    <- Kartu compact untuk grid katalog
│   │   ├── CarouselProductCard.tsx   <- Kartu format carousel
│   │   ├── CheckoutModal.tsx         <- Modal popup checkout & payment
│   │   ├── Navbar.tsx                <- Navbar floating
│   │   └── ProductCarousel.tsx       <- Auto-scroll carousel produk
│   └── dashboard/              <- Halaman admin internal (manajemen order)
├── lib/
│   └── supabase.ts             <- Koneksi Supabase (database order/tracking)
├── public/                     <- Aset statis lokal (favicon)
├── next.config.ts              <- Config Next.js (allowed image domains, dll)
└── .env.local                  <- Environment variables (JANGAN dicommit)
```

#### Hal Penting tentang `LandscapeProductCard.tsx`:
Komponen ini adalah yang paling sering diubah. Ia menerima prop `themes` berupa array objek, di mana setiap tema bisa punya `subThemes` (variasi warna). Struktur datanya:
```
themes={[
  {
    name: "Nama Tema",
    desc: "Deskripsi singkat",
    defaultSubThemeIndex: 0,        <- Variasi warna default yang aktif
    subThemes: [
      { name: "Warna 1", color: "#hex", fallbackImgSrc: "https://cdn..." },
      { name: "Warna 2", color: "#hex", fallbackImgSrc: "https://cdn..." }
    ]
  }
]}
initialSelectedIndex={2}           <- Tema mana yang aktif saat halaman pertama dibuka
```

#### CSS & Design System:
- Warna dominan: krem `#faf7f2`, coklat gelap `#382a24`
- Font display : Cormorant Garamond (`--font-cormorant`)
- Font body    : DM Sans (`--font-dm-sans`, `--font-sans`)
- Styling      : **Inline style** + CSS variables — bukan Tailwind

---

### [pakasir-gateway/] — PAYMENT GATEWAY TERPUSAT (Cloudflare Worker)
> Ini adalah otak pembayaran seluruh platform. Hati-hati saat mengubah.

Deploy : Cloudflare Workers
Stack  : Vanilla JS, Cloudflare D1 (SQLite serverless)

#### Cara Kerja Alur Pembayaran:
1. User klik "Pesan" di Valentine-Platform
2. `CheckoutModal.tsx` mengirim POST ke gateway endpoint
3. Gateway menyimpan order ke Cloudflare D1, lalu redirect ke halaman bayar Pakasir
4. Setelah bayar sukses, user diredirect ke `/order-status?order_id=xxx`
5. Gateway men-generate magic link ke produk setelah konfirmasi pembayaran

#### Endpoint Utama (`src/index.js`):
| Endpoint | Fungsi |
|---|---|
| POST /api/checkout | Buat order baru -> redirect ke Pakasir |
| POST /api/checkout-embedded | Checkout tanpa redirect halaman |
| GET /api/order-status | Cek status pembayaran |
| POST /api/confirm | Webhook konfirmasi dari Pakasir |

#### Worker Services yang Terhubung:
| Binding | Service Name | Produk |
|---|---|---|
| LETTER_WORKER | letter-edition | Letter Edition |
| VOICES_WORKER | valentine-upload | Voices Gift |
| ARCADE_WORKER | arcade-edition | Arcade Edition |
| RETRO_WORKER | birthday-retro | Retro Edition |
| WRAPPED_WORKER | loves-edition | Wrapped Edition |

> **⚠️ Catatan Penting — Dua Pola Integrasi:**
> Mixtape, Invitation, dan Loves/Memoria **TIDAK menggunakan Worker binding** karena mereka di-host di Vercel (bukan Cloudflare Worker). Sebagai gantinya, gateway memanggil API route Next.js mereka via HTTP biasa:
> - Mixtape → `https://mixtape.for-you-always.my.id/api/generate-link`
> - Invitation → `https://invitation.for-you-always.my.id/api/generate-link`
> - Loves → `https://anniv.for-you-always.my.id/api/generate-link`
>
> Pembelian semua produk **tetap works** — tapi produk berbasis Vercel sedikit lebih rentan jika Vercel app down saat webhook dipicu.

---

### [letter-project/] — PRODUK: Letter Edition (Vanilla JS)
> Surat digital interaktif dengan amplop, typewriter effect, musik latar.

Deploy : letter.for-you-always.my.id
Stack  : Vanilla HTML / CSS / JS

```
letter-project/
├── index.html      <- Halaman render surat utama
├── script.js       <- Logic utama (~50rb karakter)
├── style.css       <- Styling
├── config.js       <- Konfigurasi tema & opsi surat
├── themes/         <- Definisi per-tema visual
├── generator/      <- Studio pembuatan surat untuk buyer
├── studio/         <- Admin studio internal
├── bundle/         <- File hasil bundling/build
└── worker/         <- Cloudflare Worker (penyimpanan data surat)
```

---

### [voices-gift/] — PRODUK: Voices Gift (Vanilla JS)
> Rekaman suara digital interaktif dengan galeri foto & musik latar.

Deploy : voices.for-you-always.my.id
Stack  : Vanilla HTML / CSS / JS

```
voices-gift/
├── gift/           <- Tema default
├── gift-beige/     <- Varian tema Beige
├── gift-pinky/     <- Varian tema Pinky
├── gift-sage/      <- Varian tema Sage
├── gift-blanc/     <- Varian tema Blanc
├── generator/      <- Studio pembuatan untuk buyer
├── studio/         <- Admin studio
└── worker/         <- Cloudflare Worker (upload audio/foto)
```

---

### [mixtape-love/] — PRODUK: Mixtape Edition (Next.js)
> Kaset retro interaktif dengan foto, video & playlist musik personal.

Deploy : mixtape.for-you-always.my.id
Stack  : Next.js

```
mixtape-love/
├── app/            <- Next.js App Router (pages & API)
├── components/     <- Komponen React
├── data/           <- Data konten per project
├── lib/            <- Utilities & helpers
└── worker/         <- Cloudflare Worker backend
```

---

### [loves-edition/] — PRODUK: Wrapped Edition (Next.js)
> Recap interaktif ala Spotify Wrapped — 6 halaman kenangan.

Deploy : wrapped.for-you-always.my.id
Stack  : Next.js (Pages Router)

```
loves-edition/
├── src/            <- Source code Next.js
├── data/           <- Data konten per project
├── public/         <- Aset statis
└── updateKV.mjs    <- Script update Cloudflare KV storage
```

---

### [invitation-edition/] — PRODUK: Invitation Edition (Next.js)
> Tiket undangan kencan digital interaktif.

Deploy : invitation.for-you-always.my.id
Stack  : Next.js (App Router)

```
invitation-edition/
├── app/            <- Next.js App Router
├── components/     <- Komponen React
├── data/           <- Data undangan
└── lib/            <- Utilities
```

---

### [Arcade/] — PRODUK: Arcade Edition (Vanilla JS)
> Game interaktif berbasis ruangan (10 rooms) bertema kenangan.

Deploy : arcade.for-you-always.my.id
Stack  : Vanilla HTML / CSS / JS

```
Arcade/
├── arcade/         <- File game per-ruangan
├── generator/      <- Studio pembuatan untuk buyer
├── studio/         <- Admin studio
├── studio-premium/ <- Studio untuk paket premium
├── worker/         <- Cloudflare Worker
└── assets/         <- Gambar & aset lokal
```

---

### [birthday-retro/] — PRODUK: Retro Edition (Vanilla JS)
> Kado bertema nostalgia Windows 98, 5 tahapan kejutan interaktif.

Deploy : retro.for-you-always.my.id
Stack  : Vanilla HTML / CSS / JS

```
birthday-retro/
├── index.html      <- Entry point utama
├── generator/      <- Studio pembuatan untuk buyer
├── studio/         <- Admin studio
├── worker/         <- Cloudflare Worker
└── assets/         <- Aset visual
```

---

### [wrapped-project/] — Versi Lama / Arsip (Vanilla JS)
> Ini adalah versi lama dari Wrapped Edition — berbasis **Vanilla JS / static site** (ada `index.html`, `pages/`, `shared.css`), BUKAN Next.js. Berbeda dengan `loves-edition` yang aktif dan berbasis Next.js. Cek `git log` dulu sebelum memodifikasi apapun di sini.

---

## 4. DAFTAR PRODUK & HARGA SAAT INI

| Produk | Harga | ID di Katalog | Subdomain Produk |
|---|---|---|---|
| Memoria (Premium) | Rp 40.000 | loves | — |
| Voices Gift | Rp 15.000 | voices | voices.for-you-always.my.id |
| Mixtape Edition | Rp 15.000 | mixtape | mixtape.for-you-always.my.id |
| Invitation Edition | Rp 15.000 | invitation | invitation.for-you-always.my.id |
| Letter Edition | Rp 15.000 | letter | letter.for-you-always.my.id |
| Arcade Edition | Rp 20.000 | arcade | arcade.for-you-always.my.id |
| Retro Edition | Rp 15.000 | retro | retro.for-you-always.my.id |
| Wrapped Edition | Rp 20.000 | wrapped | wrapped.for-you-always.my.id |

---

## 5. INFRASTRUKTUR & LAYANAN EKSTERNAL

| Layanan | Fungsi |
|---|---|
| Vercel | Hosting Next.js apps (Valentine-Platform, loves-edition, mixtape-love, invitation-edition) |
| Cloudflare Workers | Hosting backend produk Vanilla JS + pakasir-gateway |
| Cloudflare D1 | Database SQLite serverless untuk tabel orders di pakasir-gateway |
| Cloudflare KV | Key-value store data project per user |
| Supabase | Database PostgreSQL untuk order tracking di Valentine-Platform |
| Pakasir | Payment provider (app.pakasir.com) |
| CDN for-you-always | cdn.for-you-always.my.id — semua gambar produk disimpan di sini |
| Vercel Analytics | Tracking traffic & konversi |

---

## 6. KONVENSI PENTING

### URL & Domain
- Storefront utama  : for-you-always.my.id
- CDN gambar        : cdn.for-you-always.my.id/[filename]
- Produk individual : [nama-produk].for-you-always.my.id

### Git Convention
- Branch utama: `main`
- Selalu push ke `origin main` setelah selesai bekerja
- Format commit: `feat(scope): pesan` / `fix(scope): pesan` / `update(scope): pesan`
- Contoh: `feat(catalog/letter): add Blush and Sage subthemes`

---

## 7. TABEL CEPAT — MAU UBAH APA?

| Mau ubah apa? | File yang dicari |
|---|---|
| Teks / copy di homepage | app/(landing)/page.tsx |
| Daftar semua produk di katalog | app/(landing)/catalog/page.tsx |
| Detail produk Letter (tema, foto, warna) | app/(landing)/catalog/letter/page.tsx |
| Detail produk lainnya | app/(landing)/catalog/[nama]/page.tsx |
| Komponen kartu produk landscape | app/components/LandscapeProductCard.tsx |
| Modal checkout & alur pembayaran | app/components/CheckoutModal.tsx |
| Navbar | app/components/Navbar.tsx |
| Logic payment gateway | pakasir-gateway/src/index.js |
| Gambar produk | URL: https://cdn.for-you-always.my.id/[id-file] |

---

*Dokumen ini dibuat Juni 2026. Update dokumen ini setiap kali ada perubahan struktur besar pada platform.*

---
> Source: [detectivecrewze/for-you-always](https://github.com/detectivecrewze/for-you-always) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
