## gadangbarubah

> Monorepo ringan: **Vite + React 18** (frontend) dan **Express + TypeScript** (backend), database **Neon PostgreSQL** via **Drizzle ORM**. Deploy target: **Railway**. DNS/CDN: **Cloudflare**. Repo: **GitHub**.

# AGENTS.md — Panduan untuk Qodo & AI di repo ini

## Konteks proyek

Monorepo ringan: **Vite + React 18** (frontend) dan **Express + TypeScript** (backend), database **Neon PostgreSQL** via **Drizzle ORM**. Deploy target: **Railway**. DNS/CDN: **Cloudflare**. Repo: **GitHub**.

## Perintah yang diizinkan

Gunakan Terminal hanya untuk perintah proyek ini:

- `npm run dev`, `npm run build`, `npm run start`, `npm run check`, `npm run db:push`
- `npm install`, `npm ci`
- `npx drizzle-kit *`, `npx tsc --noEmit`
- `git status`, `git diff`, `git log` (read-only review)

Jangan jalankan perintah destructive (`rm -rf /`, `git reset --hard`, force push) tanpa konfirmasi eksplisit user.

## Saat review kode

1. Baca `best_practices.md` di root.
2. Prioritas: **security** → **bug** → **TypeScript** → kualitas → performa.
3. Cek apakah perubahan menyentuh `shared/schema.ts` — jika ya, ingatkan `npm run db:push`.
4. Cek env: jangan menambah secret ke source; gunakan variabel lingkungan.
5. UI: konsisten dengan Tailwind + Shadcn existing; teks user-facing dalam Bahasa Indonesia.

## Saat menulis kode

- Minimal diff; jangan refactor file tidak terkait task.
- Reuse hook/komponen/util di `client/src` dan helper di `server/`.
- Role-based routes: hormati pola auth yang ada di `server/routes.ts`.
- Untuk tanggal di Drizzle: `new Date(...)` bukan string ISO mentah pada insert.

## File penting

| Area | Path |
|------|------|
| Entry server | `server/index.ts` |
| API routes | `server/routes.ts` |
| DB | `server/db.ts`, `shared/schema.ts` |
| Drizzle config | `drizzle.config.ts` |
| Vite | `vite.config.ts` |
| Client entry | `client/src/main.tsx` |

## Workflow Qodo di proyek ini

Di panel Qodo → **Workflows**, gunakan workflow project:

- **typecheck** — jalankan `npm run check` dan rangkum error
- **review-uncommitted** — review perubahan git lokal (standar Qodo + konteks repo)

Login Qodo diperlukan (Sign in di sidebar Qodo) agar review berjalan.

---
> Source: [radenrakarw3/gadangbarubah](https://github.com/radenrakarw3/gadangbarubah) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
