## hijrah-attauhid

> Panduan untuk Claude Code saat bekerja di repo ini.

# CLAUDE.md

Panduan untuk Claude Code saat bekerja di repo ini.

## Tentang Proyek

**Hijrah At-Tauhid** — Sistem Informasi Sekolah (penerus konsep Jaringan Informasi Bersama Antar Sekolah/JIBAS), penulisan ulang JIBAS 32.0 (PHP + MySQL) ke stack modern:

- **TanStack Start** (full-stack React framework, SSR) + **TanStack Router** (file-based routing di `src/routes`)
- **React 18 + TypeScript**, Vite + Nitro
- **Supabase** (PostgreSQL, Auth JWT, migrasi di `supabase/migrations`)
- **shadcn/ui** (Radix UI + Tailwind CSS), TanStack Query, react-hook-form + zod

Modul utama (lihat `JIBAS_BLUEPRINT.md` untuk detail lengkap): Akademik, Keuangan, Kepegawaian, CBE, SIMTAKA (perpustakaan), Buletin, Pengaturan, Portal/Anjungan. Halaman ada di `src/pages/`, komponen di `src/components/`, integrasi Supabase di `src/integrations/`.

## Perintah

```sh
npm run dev        # dev server (auto-reload)
npm run build      # production build
npm run lint       # eslint
npm run test       # vitest (sekali jalan)
npm run test:watch # vitest watch mode
```

Jalankan `npm run lint` dan `npm run test` sebelum commit.

## Pemilihan Model Claude

Rekomendasi model saat memakai Claude Code di proyek ini (proyek ini adalah aplikasi CRUD TypeScript/React/Supabase yang konvensional — sebagian besar tugas tidak butuh model paling mahal):

| Jenis tugas | Model | Alasan |
|---|---|---|
| **Default harian** — fitur CRUD baru, halaman/komponen shadcn, form, query Supabase, perbaikan bug biasa | `claude-sonnet-5` | Kualitas coding mendekati Opus dengan biaya jauh lebih rendah dan respons lebih cepat; lebih dari cukup untuk pola-pola berulang di proyek ini |
| **Tugas berat** — refactor besar lintas modul, migrasi skema database, debugging rumit, desain arsitektur, migrasi fitur dari JIBAS 32.0 lama | `claude-opus-4-8` | Model Opus paling mumpuni untuk pekerjaan agentic jangka panjang dan penalaran kompleks |
| **Tugas ringan** — edit kecil, rename, update dokumentasi, pertanyaan singkat | `claude-haiku-4-5` | Paling cepat dan hemat untuk tugas sederhana |

Cara mengatur model:

- Interaktif: ketik `/model` di Claude Code, atau
- Permanen per-proyek: tambahkan di `.claude/settings.json`:

```json
{
  "model": "claude-sonnet-5"
}
```

## Kontinuitas Antar Sesi

- **Di awal sesi:** baca `SESI_NOTES.md` sebelum mulai bekerja — berisi status pekerjaan terakhir, keputusan desain penting, dan rencana yang sedang berjalan
- **Di akhir pengerjaan yang berarti:** update `SESI_NOTES.md` — catat apa yang selesai, keputusan yang diambil beserta alasannya, dan langkah berikutnya yang belum dikerjakan
- Jangan mencatat hal yang sudah terekam di git history (daftar file yang diubah, isi diff) — cukup konteks yang TIDAK terlihat dari kode: alasan keputusan, status data eksternal (VPS/Supabase), rencana yang belum jalan
- Commit dengan pesan yang jelas dan deskriptif — git history adalah catatan permanen utama

## Konvensi Kode

- Ikuti konvensi di `JIBAS_BLUEPRINT.md` §13 (Konvensi Kode)
- Bahasa UI dan istilah domain memakai Bahasa Indonesia (mis. "Kepegawaian", "Keuangan"); kode (nama variabel/fungsi) memakai Bahasa Inggris
- Routing file-based: rute baru dibuat di `src/routes/`
- Validasi form dengan zod + react-hook-form; data fetching dengan TanStack Query
- Perubahan skema database selalu lewat file migrasi di `supabase/migrations/`, jangan ubah langsung

---
> Source: [YayasanAt-Tauhid/hijrah-attauhid](https://github.com/YayasanAt-Tauhid/hijrah-attauhid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
