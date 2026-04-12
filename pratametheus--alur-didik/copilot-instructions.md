## alur-didik

> Sistem manajemen RPP (Rencana Pelaksanaan Pembelajaran) dan data guru untuk SD dengan Kurikulum Merdeka (2022).

@AGENTS.md

# RPP Manager

Sistem manajemen RPP (Rencana Pelaksanaan Pembelajaran) dan data guru untuk SD dengan Kurikulum Merdeka (2022).

## Stack

- **Framework:** Next.js 16.2.0 (App Router, Turbopack)
- **Language:** TypeScript
- **Styling:** Tailwind CSS 4 + inline style (warna brand)
- **Icons:** lucide-react (optimized via `optimizePackageImports`)
- **Auth:** Multi-role JWT via `jose` (HS256, 8h expiry), cookie `rpp_sid`. Dua role: `admin` dan `guru`
- **Data:** Google Sheets API via `googleapis` + service account
- **Email:** Nodemailer + SMTP (opsional, graceful no-op)
- **Scripts:** `tsx` + `dotenv` untuk migration scripts

## Struktur Project

```
src/
├── app/
│   ├── layout.tsx              # Root layout (html/body, tanpa sidebar)
│   ├── login/page.tsx          # Halaman login (di luar route group)
│   ├── actions/
│   │   ├── auth.ts             # Server Action: login (dual-path admin+guru), logout, changePassword
│   │   ├── guru.ts             # Server Action: saveGuru, deleteGuruAction, resetGuruPassword
│   │   ├── rpp.ts              # Server Action: saveRPP, updateRPPStatus, deleteRPPAction
│   │   ├── kelas.ts            # Server Action: saveKelas, deleteKelasAction
│   │   ├── mapel.ts            # Server Action: saveMapel, deleteMapelAction
│   │   ├── search.ts           # Server Action: globalSearch (omnisearch, role-scoped)
│   │   ├── notifications.ts   # Server Action: getNotifications (role-scoped)
│   │   ├── audit.ts           # Server Action: logAudit, getAuditLogsForRpp
│   │   ├── email.ts           # Server Action: sendRppStatusEmail, sendRppSubmittedEmail (fire-and-forget)
│   │   └── pengaturan.ts      # Server Action: savePengaturanSekolah, savePengaturanAdmin, savePengaturanNotifikasi
│   ├── global-error.tsx        # Error boundary root-level (fallback)
│   └── (app)/                  # Route group — semua halaman di sini punya Sidebar
│       ├── layout.tsx          # Server layout: reads session, wraps in SessionProvider
│       ├── AppShell.tsx        # Client layout: sidebar + hamburger + main content
│       ├── loading.tsx         # Suspense skeleton (shared semua route)
│       ├── error.tsx           # Error boundary shared (app group)
│       ├── page.tsx            # Dashboard (role-aware: admin sees all, guru sees own RPP)
│       ├── guru/               # page.tsx (server) + GuruTable.tsx (client) — admin only
│       ├── rpp/                # page.tsx (server, role-scoped) + RppGrid.tsx (client, role-aware)
│       ├── kelas/              # page.tsx (server) + KelasGrid.tsx (client, read-only for guru)
│       ├── mata-pelajaran/     # page.tsx (server) + MataPelajaranGrid.tsx (client, read-only for guru)
│       └── pengaturan/         # page.tsx (server) + PengaturanForm.tsx (client) — admin only
├── components/
│   ├── charts/
│   │   ├── BarChart.tsx        # Pure SVG bar chart (trend RPP per bulan)
│   │   └── DonutChart.tsx      # Pure SVG donut chart (distribusi per mapel)
│   ├── ConfirmDialog.tsx       # Dialog konfirmasi hapus (reusable)
│   ├── ServiceWorkerRegister.tsx # PWA service worker registration
│   ├── SessionProvider.tsx     # React Context: SessionInfo (userId, role, userName)
│   ├── Header.tsx              # Header bar dengan search + notif (role-aware)
│   ├── Sidebar.tsx             # Navigasi sidebar (role-aware: hides admin-only items for guru)
│   └── StatusBadge.tsx         # Badge status (aktif/draft/diajukan/dll)
├── lib/
│   ├── auth-helpers.ts         # requireAuth(), requireAdmin() — server action guards
│   ├── password.ts             # PBKDF2 hashPassword/verifyPassword (Web Crypto API)
│   ├── rate-limiter.ts         # Rate limiter (Upstash Redis + in-memory fallback)
│   ├── session.ts              # JWT encrypt/decrypt, createSession, getSession. SessionPayload: {userId, role, userName}
│   ├── sheets.ts               # Google Sheets client factory (service account, supports inline JSON + file path)
│   ├── sheets-helpers.ts       # CRUD helpers: findRowById, appendRow, updateRow, deleteRow, updateCells
│   ├── guru-repository.ts      # getGuruList() (cached, strips passwordHash), getGuruByEmail() (uncached, for auth)
│   ├── rpp-repository.ts       # getRPPList() — baca dari Sheet "RPP"
│   ├── kelas-repository.ts     # getKelasList() — baca dari Sheet "Kelas"
│   ├── mata-pelajaran-repository.ts  # getMataPelajaranList() — baca dari Sheet "MataPelajaran"
│   ├── pengaturan-repository.ts     # getPengaturan() — baca dari Sheet "Pengaturan" (key-value)
│   ├── analytics.ts                 # groupByMonth(), groupByMapel(), getApprovedThisMonth() — pure functions, terima RPP[]
│   ├── audit-repository.ts         # getAuditLogByTargetId(), getAllAuditLogs() — baca dari Sheet "AuditLog"
│   ├── mailer.ts                    # Nodemailer transporter factory (graceful no-op jika SMTP env vars tidak ada)
│   ├── email-templates.ts           # HTML email templates: rppApproved, rppRejected, rppSubmitted
│   └── utils.ts                # cn() helper (clsx + tailwind-merge)
├── types/index.ts              # Guru, RPP, Kelas, MataPelajaran, ActionState, UserRole, SessionInfo
└── proxy.ts                    # Route protection + role-based access (admin-only routes)
```

## Arsitektur & Pola

### Multi-Role Auth
- **Admin** (kepala sekolah): Credentials dari env vars. Full access semua fitur.
- **Guru**: Credentials dari Google Sheets (email + PBKDF2 password hash di kolom K). Bisa buat/edit RPP sendiri, view kelas & mapel (read-only). Default password = NIP.
- **Login flow**: `auth.ts` cek admin (timing-safe) DAN guru (Sheets lookup) di setiap request untuk hindari timing attack.
- **Route protection**: `proxy.ts` block guru dari `/guru` dan `/pengaturan`. Server actions juga enforce via `requireAdmin()`.
- **Session**: `SessionPayload = { userId, role: "admin"|"guru", userName }`. Passed ke client via `SessionProvider` context.

### Server/Client Split
- **Server Components** (async): Semua `page.tsx` — fetch data dari Sheets + session, pass sebagai props
- **Client Components** (`"use client"`): Semua Grid/Table — handle state, search, filter, modal, form
- **Server Actions** (`"use server"`): Semua mutasi data — pakai `requireAdmin()` atau `requireAuth()` di awal

### Authorization Pattern
| Action | Guard | Notes |
|--------|-------|-------|
| guru CRUD | `requireAdmin()` | Admin only |
| kelas CRUD | `requireAdmin()` | Admin only |
| mapel CRUD | `requireAdmin()` | Admin only |
| pengaturan | `requireAdmin()` | Admin only |
| rpp save | `requireAuth()` | Guru: forced own identity, status restricted to draft/diajukan |
| rpp approve/reject | `requireAdmin()` | Admin only |
| rpp delete | `requireAuth()` | Guru: only own draft RPP |
| search | `requireAuth()` | Guru: no guru results, only own RPP |
| notifications | `requireAuth()` | Guru: own RPP status changes |

### CRUD Pattern
Semua operasi CRUD mengikuti pola yang sama:
1. Server Action verifikasi session + role via `requireAdmin()` atau `requireAuth()`
2. Validasi input (wajib isi, panjang)
3. Operasi ke Google Sheets via `sheets-helpers.ts`
4. `revalidatePath()` + `revalidateTag()` untuk refresh data
5. Return `ActionState` (`{ success: true }` atau `{ error: "..." }`)

Client component menggunakan `useTransition` + `startTransition` untuk pending state.

### Data Flow
```
Google Sheets → Repository (server-only) → Server Component → Client Component (props)
                                                                    ↓
Client Form → Server Action → sheets-helpers → Google Sheets → revalidatePath
```

## Environment Variables (.env.local)

```
SESSION_SECRET=...           # 256-bit random key untuk JWT
ADMIN_EMAIL=...              # Email admin untuk login
ADMIN_PASSWORD=...           # Password admin
GOOGLE_SERVICE_ACCOUNT_KEY_PATH=./rpp-manager-114ad261b470.json  # Lokal: file path
GOOGLE_SERVICE_ACCOUNT_JSON=...  # Production (Vercel): inline JSON string (alternatif KEY_PATH)
GOOGLE_SPREADSHEET_ID=...   # ID spreadsheet Google Sheets
UPSTASH_REDIS_REST_URL=...  # Opsional: Upstash Redis untuk rate limiter
UPSTASH_REDIS_REST_TOKEN=... # Opsional: Upstash Redis token
SMTP_HOST=...               # Opsional: SMTP server (e.g. smtp.gmail.com)
SMTP_PORT=587               # Opsional: SMTP port (587=STARTTLS, 465=SSL)
SMTP_USER=...               # Opsional: SMTP username (e.g. email@gmail.com)
SMTP_PASS=...               # Opsional: SMTP password / app password
SMTP_FROM=...               # Opsional: Sender address (fallback ke SMTP_USER)
```

## Google Sheets Schema

### RPP (kolom A-Q)
`id | judul | guruId | guruNama | mataPelajaran | kelas | semester | tahunAjaran | fase | capaianPembelajaran | tujuanPembelajaran | materi | metode | alokasiwaktu | status | createdAt | updatedAt`

### Guru (kolom A-K)
`id | nip | nama | email | mataPelajaran | kelas | status | jabatan | telepon | createdAt | passwordHash`

**Migration**: Jalankan `npm run migrate:guru-passwords` untuk tambah kolom passwordHash (K) dengan default password = NIP.

### Kelas (kolom A-G)
`id | nama | tingkat | waliKelas | jumlahSiswa | tahunAjaran | fase`

### MataPelajaran (kolom A-F)
`id | kode | nama | tingkatan | kktp | deskripsi`

### AuditLog (kolom A-F)
`timestamp | userId | userName | action | targetId | detail`

**Actions**: `create_rpp`, `edit_rpp`, `approve_rpp`, `reject_rpp`, `delete_rpp`

**Setup**: Jalankan `npm run migrate` untuk setup semua sheets sekaligus (termasuk AuditLog).

## Fitur yang Sudah Selesai

### Autentikasi & Keamanan
- [x] Multi-role auth: admin (env vars) + guru (Google Sheets + PBKDF2 hash)
- [x] JWT stateless + HttpOnly cookie (8h expiry)
- [x] Role-based route protection via `proxy.ts` (admin-only: `/guru`, `/pengaturan`)
- [x] Server action authorization via `requireAdmin()` / `requireAuth()`
- [x] Guru ownership enforcement: hanya bisa edit/hapus RPP sendiri
- [x] Timing-safe password comparison (admin) + PBKDF2 (guru)
- [x] Rate limiter: Upstash Redis (fallback in-memory)
- [x] Security headers (CSP nonce-based, HSTS, X-Frame-Options)
- [x] Password management: admin reset guru password, guru change own password

### Dashboard
- [x] Admin: stat cards (Total Guru, Total RPP, Total Kelas, RPP Menunggu), guru table
- [x] Guru: stat cards (RPP Saya, Diajukan, Draft, Disetujui), RPP terbaru sendiri
- [x] Chart status RPP (progress bar per status)
- [x] Parallel data fetching via `Promise.all`

### CRUD Guru (admin only)
- [x] Tabel guru dengan search + filter status
- [x] Modal detail guru + tombol Reset Password
- [x] Form modal Tambah/Edit guru dengan password field
- [x] Dialog konfirmasi Hapus guru

### CRUD RPP (role-aware)
- [x] Admin: full CRUD + approve/reject semua RPP
- [x] Guru: buat/edit RPP sendiri (status: draft/diajukan), hapus draft sendiri
- [x] Card grid RPP dengan search + filter 5 status
- [x] Modal detail RPP, form modal Buat/Edit RPP
- [x] Tombol Setujui/Tolak (admin only)

### CRUD Kelas & Mata Pelajaran
- [x] Admin: full CRUD
- [x] Guru: read-only (tanpa tombol Tambah/Edit/Hapus)

### Header (role-aware)
- [x] Search global — role-scoped (guru: tanpa guru results, hanya RPP sendiri)
- [x] Notifikasi — role-scoped (admin: RPP pending, guru: status RPP sendiri)
- [x] Avatar — role-aware profile info, role-aware menu items

### Pengaturan (admin only)
- [x] Profil Sekolah — simpan ke Google Sheets
- [x] Profil Admin — simpan ke Google Sheets
- [x] Toggle Notifikasi — persisted, auto-save on toggle

### PWA (Progressive Web App)
- [x] Web App Manifest, Service Worker, PWA meta tags, SVG icons

### Aksesibilitas & Responsif
- [x] ARIA attributes di semua dialog, icon-only buttons, toggles
- [x] Escape key handler, skip navigation link
- [x] Mobile-responsive: hamburger sidebar, responsive padding, stacking form grids
- [x] Touch-friendly target sizes

### Production Readiness
- [x] CSP nonce-based
- [x] Caching: `'use cache'` + `cacheLife('minutes')` + `cacheTag()` + `revalidateTag()`
- [x] Error boundaries: `(app)/error.tsx` + `global-error.tsx`
- [x] Testing: Vitest (unit) + Playwright (E2E)

### Export / Reporting (Task 6 — SELESAI)
- [x] Export CSV: client-side, zero dependency, BOM prefix untuk Excel Windows UTF-8
- [x] `src/lib/export-csv.ts` — utility `exportRppToCsv(filtered)`
- [x] Export PDF: browser print via Route Handler, letterhead dari `getPengaturan()`, ownership enforced
- [x] `src/app/api/rpp/[id]/pdf/route.ts` — GET, auth + role check, render HTML printable
- [x] `src/app/(app)/rpp/RppGrid.tsx` — tombol "Export CSV" di toolbar, tombol Printer icon di tiap card

### Notifikasi Real-Time (Task #5 — SELESAI)
- [x] `src/app/api/notifications/count/route.ts` — GET Route Handler lightweight, role-scoped, no-store cache
- [x] `src/hooks/useNotificationCount.ts` — Client hook: poll setiap 60s + refresh on tab focus
- [x] `src/components/Header.tsx` — Badge dot auto-update via polling; fixed bug: dulu badge selalu muncul sebelum klik pertama

### Export/Reporting (Task #6 — SELESAI)
- [x] `src/lib/export-csv.ts` — Export RPP ke CSV; BOM prefix untuk Excel Windows; zero dependency
- [x] `src/app/api/rpp/[id]/pdf/route.ts` — Route Handler renders HTML printable RPP; auto-trigger `window.print()`; letterhead dari `getPengaturan()`; role enforcement (guru 403 jika bukan pemilik)
- [x] `src/app/(app)/rpp/RppGrid.tsx` — Export CSV button (exports `filtered`, disabled jika kosong) + Printer icon per card

### Audit Log / History (Task #7 — SELESAI)
- [x] Sheet "AuditLog" (kolom A-F): timestamp, userId, userName, action, targetId, detail
- [x] `src/types/index.ts` — interface `AuditLog`
- [x] `src/lib/audit-repository.ts` — `getAuditLogByTargetId()`, `getAllAuditLogs()` (server-only)
- [x] `src/app/actions/audit.ts` — `logAudit()` (fire-and-forget append), `getAuditLogsForRpp()` (fetch by RPP id)
- [x] `src/app/actions/rpp.ts` — Audit logging terintegrasi: create, edit, approve, reject, delete RPP
- [x] `src/app/(app)/rpp/RppGrid.tsx` — Timeline riwayat aktivitas di modal detail RPP (fetch on-demand)

### Notifikasi Email (Task #8 — SELESAI)
- [x] `src/lib/mailer.ts` — Nodemailer transporter factory (server-only, graceful no-op jika SMTP env vars tidak ada)
- [x] `src/lib/email-templates.ts` — 3 HTML template mobile-friendly (inline CSS, max-width 600px): rppApproved, rppRejected, rppSubmitted
- [x] `src/app/actions/email.ts` — `sendRppStatusEmail()` (ke guru saat approve/reject), `sendRppSubmittedEmail()` (ke admin saat guru ajukan RPP)
- [x] `src/app/actions/rpp.ts` — Email fire-and-forget: setelah audit log di `updateRPPStatus` dan `saveRPP` (status=diajukan)
- [x] Respects `notifEmailHarian` toggle dari Pengaturan — cek sebelum kirim
- [x] Env vars: `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM` (semua opsional)

### Dashboard Analytics Lanjutan (Task #9 — SELESAI)
- [x] `src/lib/analytics.ts` — Pure functions: `groupByMonth()`, `groupByMapel()`, `getApprovedThisMonth()` (server-only, terima RPP[])
- [x] `src/components/charts/BarChart.tsx` — Pure SVG bar chart, "use client", accessible (role="img", aria-label), mobile-friendly
- [x] `src/components/charts/DonutChart.tsx` — Pure SVG donut chart + legend, "use client", accessible, mobile-friendly
- [x] `src/app/(app)/page.tsx` — Integrasi analytics: stat card "Disetujui Bulan Ini" (admin), bar chart trend 12 bulan, donut distribusi mapel
- [x] Role-aware: admin = semua RPP, guru = RPP sendiri
- [x] Zero dependency — pure SVG, no chart library

### Deployment & CI/CD (Task #10 — SELESAI)
- [x] `.github/workflows/ci.yml` — GitHub Actions: lint → test → e2e (push only). Node 20, ubuntu-latest, npm cache
- [x] `vercel.json` — Vercel config: Next.js framework, security headers, `check-env` sebelum build
- [x] `scripts/check-env.ts` — Validasi env vars sebelum build; exit 1 jika required vars missing
- [x] `DEPLOYMENT.md` — Step-by-step guide: Vercel setup, env vars, Google Sheets, migration, troubleshooting
- [x] `src/lib/sheets.ts` — Support `GOOGLE_SERVICE_ACCOUNT_JSON` (inline JSON) sebagai alternatif file path
- [x] `package.json` — Script `check-env` dan `migrate` ditambahkan

## Deployment

Lihat **[DEPLOYMENT.md](./DEPLOYMENT.md)** untuk panduan lengkap deploy ke Vercel.

## Status Saat Ini (Sesi 2026-03-21)

### Yang Sudah Berjalan
- Semua 10 task original selesai (auth, CRUD, notif realtime, export,
  audit log, email, analytics, deployment)
- Full adaptasi SD Kurikulum Merdeka: fase A/B/C, capaianPembelajaran,
  KKTP, data seed SD
- Security audit: input validation, HTML escaping, email injection prevention
- Accessibility audit: focus trap, touch targets 44px, aria-live, aria-label

### Known Issues (Belum Fix)
- [ ] Service Worker cache stale setelah refactoring besar —
      FIX: bump CACHE_NAME ke v2 di public/sw.js (sedang dikerjakan)
- [ ] zlib.bytesRead DeprecationWarning di Node.js — minor, tidak affect
      production, suppress dengan NODE_NO_WARNINGS=1 di npm run dev

### Next Steps (Sesi Berikutnya)
1. **Push ke GitHub** — belum ada commit sejak Task #7. Jalankan:
   git add . && git commit -m "feat: SD Kurikulum Merdeka refactor + security/a11y audit" && git push origin main

2. **Jalankan npm run migrate** — reset Google Sheets dengan data SD:
   - Sheet RPP: kolom baru fase + capaianPembelajaran (A-Q)
   - Sheet Kelas: kolom baru fase (A-G), hapus jurusan
   - Sheet MataPelajaran: kktp menggantikan kkmScore

3. **Testing end-to-end lokal** sebelum deploy:
   - Login sebagai admin dan guru
   - Buat RPP baru dengan field fase + capaianPembelajaran
   - Cek dashboard charts (bar chart + donut)
   - Cek export CSV dan PDF

4. **Deploy ke Vercel** — ikuti DEPLOYMENT.md:
   - Set semua env vars di Vercel dashboard
   - Gunakan GOOGLE_SERVICE_ACCOUNT_JSON (inline JSON) bukan file path
   - Jalankan npm run migrate:guru-passwords di production

5. **Security + Accessibility prompt** — belum dijalankan (interrupted):
   Prompt sudah siap, jalankan di sesi berikutnya setelah app berjalan
   bersih tanpa error.

## Konvensi

- **File baru di `(app)/`**: Selalu `page.tsx` (server) + `NamaGrid.tsx` atau `NamaTable.tsx` (client)
- **Server Action baru**: Taruh di `src/app/actions/`, selalu pakai `requireAdmin()` atau `requireAuth()`, return `ActionState`
- **Repository baru**: Taruh di `src/lib/`, gunakan `"server-only"` import
- **Hapus row**: Gunakan `deleteRow` dari `sheets-helpers.ts` (batchUpdate, bukan clear)
- **Partial update**: Gunakan `updateCells` dari `sheets-helpers.ts`
- **Warna brand**: `#1e3a5f` (sidebar), `#2563eb` (primary blue)
- **Password hashing**: PBKDF2-SHA256, 100k iterations, 32-byte salt, Web Crypto API. Stored as `base64(salt).base64(hash)`
- **Auth guard**: `requireAdmin()` untuk admin-only actions, `requireAuth()` untuk any authenticated user
- **Route Handler baru di `/api/`**: Middleware tidak melindungi `/api/` — selalu panggil `getSession()` manual di dalam handler dan return 401 jika null
- **Export CSV**: Gunakan `exportRppToCsv()` dari `src/lib/export-csv.ts`; BOM `\uFEFF` wajib untuk Excel Windows UTF-8
- **Params Next.js 16**: Route Handler dynamic params adalah `Promise` — selalu `await params` sebelum destructure
- **Polling hook**: `src/hooks/useNotificationCount.ts` — interval 60s (aligned `cacheLife("minutes")`), cleanup via `visibilitychange`
- **Audit log**: `logAudit()` dipanggil setelah setiap mutasi RPP (create, edit, approve, reject, delete). Fire-and-forget — jangan block main action
- **Email notifikasi**: Fire-and-forget via `.catch(() => {})`. Selalu cek `notifEmailHarian` toggle sebelum kirim. Graceful no-op jika SMTP belum dikonfigurasi
- **Analytics**: Pure functions di `src/lib/analytics.ts` — terima `RPP[]` sebagai input, jangan fetch sendiri dari Sheets. Chart components di `src/components/charts/` — pure SVG, zero dependency
- **Google Sheets auth**: Support 2 cara — `GOOGLE_SERVICE_ACCOUNT_JSON` (inline JSON, untuk Vercel) atau `GOOGLE_SERVICE_ACCOUNT_KEY_PATH` (file path, untuk lokal). Inline JSON dicek duluan
- **Pre-build check**: `npm run check-env` dijalankan sebelum build (via `vercel.json` buildCommand). Script di `scripts/check-env.ts`
- **Sheet baru**: Tambahkan ke `scripts/migrate-all.ts` array `SHEETS` (name, headers, rows). Jalankan `npm run migrate` untuk setup semua sheets sekaligus
- **Kurikulum**: Kurikulum Merdeka SD. Fase A=kelas 1-2, B=kelas 3-4, C=kelas 5-6. 1 JP = 35 menit. KKTP menggantikan KKM. Capaian Pembelajaran (CP) menggantikan Kompetensi Dasar (KD).
- **Input validation**: Semua Server Action wajib validasi panjang input (max length) dan format numerik sebelum operasi Sheets. Lihat contoh di `guru.ts`, `rpp.ts`
- **HTML escaping**: Semua data dinamis di email templates wajib di-escape via `escapeHtml()` di `email-templates.ts`
- **Email validation**: Validasi format email (`/^[^\s@]+@[^\s@]+\.[^\s@]+$/`) sebelum login dan sebelum kirim email (prevent SMTP header injection)
- **Focus trap**: Semua modal wajib pakai `useFocusTrap()` dari `src/hooks/useFocusTrap.ts` — trap Tab/Shift+Tab, auto-focus first element, restore focus on close
- **Touch target**: Tombol icon-only wajib `min-w-[44px] min-h-[44px]` (WCAG 2.2 Level AA). Jangan pakai `w-8 h-8` (32px) untuk tombol interaktif
- **Aria-live**: Error messages di form wajib pakai `role="alert" aria-live="polite"` agar screen reader mengumumkan error
- **Aria-label**: Tombol icon-only wajib `aria-label` deskriptif. Contoh: `aria-label="Hapus kelas 1A"`

## Security & Accessibility

### Security (Audit 2026-03-21)
- Input length validation di semua Server Action (guru, rpp, kelas, mapel, pengaturan)
- HTML escaping di email templates (`escapeHtml()`) untuk mencegah XSS
- Email format validation di login (`auth.ts`) dan sebelum kirim email (`email.ts`)
- Ownership check di audit logs — guru hanya bisa lihat log RPP milik sendiri
- Hardcoded default password dihapus dari `resetGuruPassword` — fallback ke NIP wajib
- Numeric validation untuk alokasiwaktu, jumlahSiswa, kktp
- SMTP header injection prevention via email format validation

### Accessibility — WCAG 2.2 Level AA (Audit 2026-03-21)
- Focus trap di semua modal (RppGrid, KelasGrid, MataPelajaranGrid, ConfirmDialog) via `useFocusTrap` hook
- Touch targets 44x44px minimum pada semua tombol icon-only (delete, print)
- `role="alert" aria-live="polite"` pada semua error message div
- `aria-label` pada login password toggle button
- `aria-modal="true"`, `aria-labelledby` pada semua dialog

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pratametheus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Pratametheus)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
