## educore

> Panduan utama untuk AI coding agent yang bekerja di repository `EduCore`.

# AGENTS.md

Panduan utama untuk AI coding agent yang bekerja di repository `EduCore`.
File ini harus dianggap sebagai **instruksi operasional utama** untuk chat baru agar agent memahami arsitektur, batasan runtime, standar kualitas, dan urutan eksekusi kerja.

---

## 1. Project Identity

**EduCore** adalah aplikasi manajemen sekolah hybrid:
- **Desktop**: Tauri v2
- **Web**: Next.js 16 App Router + React 19
- **Bahasa**: TypeScript strict
- **Package manager**: Bun
- **ORM**: Drizzle ORM
- **Desktop local DB**: SQLite
- **Cloud shared DB**: Turso / libSQL

Target produk:
- Bisa dipakai di **PC/laptop** via desktop app
- Bisa dipakai di **browser** tanpa install
- Bisa diakses dari **HP** via web responsif / PWA
- Mendukung **offline-first di desktop**
- Mendukung **multi-user dengan data sekolah yang sama**

---

## 2. Current Architecture Truth

Instruksi di file ini harus mengikuti kondisi aktual repo, bukan asumsi lama.

### Shared model
- **Cloud DB (Turso)** = shared collaboration layer antar user/device
- **Local SQLite desktop** = operational source of truth saat desktop offline
- **Web runtime** = online-first, akses data melalui backend web
- **Desktop runtime** = local-first, akses data melalui local runtime bridge / service lokal
- **Sync engine** = push/pull delta antara desktop local DB dan cloud

### Source of truth policy
- **Global shared truth**: Turso
- **Local operational truth**: SQLite per desktop device

Jangan memodelkan sistem sebagai “semua device memakai satu DB lokal yang sama”.
Sistem yang benar adalah **replicated local-first system**.

---

## 3. Runtime Boundary Policy

### Desktop runtime
Desktop harus:
- usable tanpa internet
- membaca/menulis ke SQLite lokal
- tidak bergantung pada `/api/*` web untuk flow inti
- tidak bergantung pada session provider web untuk flow inti
- hanya sync ke cloud saat online

Desktop tidak boleh:
- memerlukan Next server untuk CRUD inti
- menarik dependency web/server-only ke client bundle desktop
- mengandalkan secret cloud di frontend

### Web runtime
Web harus:
- memakai route handler / backend layer untuk akses cloud
- memakai auth web
- usable di browser desktop dan HP

Web tidak boleh:
- mengakses Tauri API
- mengakses SQLite desktop
- membawa secret Turso ke browser bundle

### Client component policy
Client component **dilarang**:
- import service DB secara langsung
- import module native-only / node-only yang berisiko terbawa ke browser
- mencampur business rule penting dengan presentational state

---

## 4. Build and Distribution Policy

### Web distribution
Web adalah jalur distribusi universal:
- untuk browser desktop
- untuk HP
- untuk user yang belum install aplikasi

### Desktop distribution
Desktop adalah jalur distribusi offline-first:
- untuk admin / staff / guru / operator
- harus memakai local DB + sync

### Important build rule
Desktop production build **tidak boleh** dibuka jika masih ada flow inti yang bergantung ke runtime web.
Lebih baik build desktop **fail-secure** daripada menghasilkan installer yang terlihat valid tapi rusak.

Status aktual saat ini:
- jalur Windows desktop yang sudah disignoff = `MSI`
- `NSIS` belum otomatis ikut dianggap siap
- signoff desktop harus disebut per-channel installer, bukan digeneralisasi

---

## 5. Technology Baseline

Gunakan stack ini, jangan mengganti tanpa instruksi eksplisit user:

- **Desktop**: Tauri v2
- **Web**: Next.js 16 App Router
- **UI**: React 19
- **Language**: TypeScript strict
- **Styling**: Tailwind CSS v4
- **Components**: shadcn/ui + Radix UI
- **Icons**: Lucide React
- **Toasts**: Sonner
- **State**: Zustand + Nuqs
- **ORM**: Drizzle ORM
- **Cloud DB**: Turso / libSQL
- **Desktop DB**: SQLite
- **Lint/format**: Biome
- **Testing**: Vitest + React Testing Library + Playwright

---

## 6. Database and Schema Policy

### Schema authority
Single source of truth schema ada di:
- [src/core/db/schema.ts](src/core/db/schema.ts)

Kalau ada perubahan schema, agent wajib cek:
- connection layer
- query terkait
- relasi
- migration
- validasi Zod
- sync contract
- data adapter web/desktop

### Minimum entity rules
Untuk entity yang disync, standar field minimal:
- `id`
- `version`
- `updatedAt`
- `deletedAt`
- `syncStatus` jika dipakai local queue
- `hlc` jika dipakai conflict resolution

### Delete policy
- gunakan **soft delete** untuk entity bisnis penting
- jangan hard delete tanpa alasan kuat
- query harus sadar `deletedAt`

---

## 7. Sync Policy

Sync bukan fitur tambahan. Sync adalah bagian inti arsitektur.

### Contract
Desktop:
- write ke local DB dulu
- tandai perubahan untuk sync
- push ke cloud saat online
- pull perubahan cloud ke local

### Conflict policy
Default:
- **LWW dengan HLC-aware policy** jika tersedia

Fallback:
- bandingkan `updatedAt`
- gunakan deterministic tie-breaker jika perlu

### Security
- cloud tidak boleh percaya payload mentah dari client
- semua payload sync harus tervalidasi
- secret sync tidak boleh masuk browser bundle

---

## 8. Auth Policy

### Web auth
- auth web dikelola lewat flow web
- session web hanya untuk runtime web

### Desktop auth
- desktop harus punya flow auth/session lokal sendiri
- desktop tidak boleh mewajibkan `SessionProvider` web untuk berfungsi
- logout / refresh / change-password desktop harus aman di runtime lokal
- packaged desktop auth harus dianggap berbeda dari `tauri dev`
- route desktop yang dijaga middleware harus punya proof runtime server-side yang stabil

### Shared auth rules
Harus konsisten lintas runtime:
- role
- permission
- account status
- password validation
- access boundary

---

## 9. Code Quality Rules

### General
- TypeScript strict
- no `any` kecuali benar-benar unavoidable dan dibatasi
- no `@ts-ignore`
- no silent catch
- no destructive revert terhadap perubahan user yang tidak diminta

### Validation
Semua input eksternal wajib divalidasi:
- frontend form
- route/API handler
- local runtime bridge
- sync payload

Gunakan Zod / schema validation yang konsisten.

### Business logic
Business logic harus hidup di:
- domain/service layer
- adapter layer

Bukan di:
- page component
- dialog component
- random hook presentasional

### Error handling
Semua error penting harus:
- punya feedback jelas ke UI
- punya status/code yang konsisten
- tidak jatuh ke generic error jika bisa dibedakan

### Performance
- hindari duplicate request
- hindari client state yang drift dari backend/local DB source of truth
- hindari bundling module server/native ke browser

---

## 10. React Rules

- Gunakan React 19 patterns yang simpel dan defensible
- Jangan menambah `useMemo` / `useCallback` tanpa alasan kuat
- Utamakan state flow yang jelas
- Loading state, empty state, error state wajib eksplisit
- Form lifecycle harus lengkap: default value, submit, success, error, reset, pending

---

## 11. Security Rules

Wajib menjaga:
- role/access boundary
- runtime boundary web vs desktop
- tidak ada secret cloud di browser bundle
- tidak ada desktop-only capability dipanggil di web
- tidak ada web-only dependency dipakai untuk flow desktop inti

Untuk data sensitif, prioritaskan:
- audit log
- clear failure mode
- deny-by-default permission

---

## 12. AI Working Rules

Jika AI agent bekerja di repo ini:

### Wajib dilakukan sebelum implementasi
1. audit teknis dulu
2. jangan langsung asumsi
3. baca file terkait dulu
4. pahami boundary runtime yang terlibat
5. cek apakah perubahan menyentuh schema, sync, auth, atau build path
6. jika bug hanya muncul di deploy/installer, anggap itu sebagai **runtime boundary issue** sampai terbukti sebaliknya

### Saat mengubah kode
AI harus:
- menyebut file yang akan diubah sebelum edit
- meminimalkan radius perubahan
- menjaga modul stabil yang tidak terkait
- tidak merusak auth, settings, attendance, students, dashboard, kecuali memang bagian task

### Setelah implementasi
AI harus validasi bertahap:
1. `bunx biome check .`
2. `bunx tsc --noEmit`
3. `bun run build`
4. jika relevan desktop production path: `bun run build:desktop`
5. jika menyentuh packaged desktop path: `bun tauri build`

Kalau ada kegagalan, AI harus jelaskan:
- akar masalah
- apakah `web-only`, `desktop-only`, atau `keduanya`

---

## 13. Default Workflow for New Tasks

Untuk task baru, AI harus memakai urutan ini:

1. **Audit**
   - cari file terkait
   - pahami flow eksisting
   - cek apakah ada dead path / risky path / boundary leak

2. **Diagnose**
   - jelaskan akar masalah singkat
   - bedakan issue runtime: web / desktop / keduanya

3. **Implement**
   - edit sekecil mungkin tapi tuntas
   - jaga source of truth tetap jelas

4. **Stabilize**
   - cek apakah ada komponen/backend/util yang mubazir atau regress-prone
   - rapikan jika langsung terkait

5. **Validate**
   - biome
   - tsc
   - build

6. **Report**
   - file yang diubah
   - akar masalah
   - solusi
   - hasil validasi
   - langkah retest
   - residual risk

Untuk incident deploy/packaged yang terasa sulit, baca juga:
- [docs/runtime-boundary-incident-postmortem.md](/e:/Freelance/Project/educore/docs/runtime-boundary-incident-postmortem.md)

---

## 14. Definition of Done

Sebuah perubahan dianggap selesai jika:
- source of truth jelas
- boundary runtime aman
- validasi input lengkap
- feedback sukses/gagal jelas
- loading/empty/error state ada
- tidak ada request duplication yang tidak perlu
- tidak ada client direct DB access
- tidak ada jalur mubazir/regress-prone yang dibiarkan kalau langsung terkait task
- `bunx biome check .` lulus
- `bunx tsc --noEmit` lulus
- `bun run build` lulus

Tambahan untuk desktop production-safe area:
- tidak bergantung ke route web untuk flow inti
- tidak menarik dependency server/native ke browser bundle
- jika signoff disebut “final”, sebutkan channel installer yang benar-benar lolos, misalnya `MSI`

---

## 15. EduCore Phase Plan (Updated Operational Interpretation)

Master plan besar tetap:
- Fase 0: setup
- Fase 1: fondasi data + auth
- Fase 2: akademik inti
- Fase 3+: modul lanjut

Tetapi interpretasi implementasi harus mengikuti arsitektur hybrid aktual repo ini.

### Fase 2 target operasional

#### 2.1 Master Data
Entitas target:
- tahun ajaran
- semester
- kelas
- mata pelajaran
- guru_mapel

Target:
- desktop local-first CRUD
- web CRUD
- sync-enabled
- role-safe

#### 2.2 Jadwal
Target:
- local conflict detection
- local editor
- web compatibility
- sync-enabled

#### 2.3 Absensi
Target:
- desktop production-safe offline capture
- sync queue
- reporting yang tidak merusak runtime boundary

#### 2.4 Keuangan Dasar
Target:
- transaction-safe
- audit-friendly
- sync-safe

### Fase 2 belum dianggap selesai jika:
- baru jalan di `bun tauri dev` tapi belum desktop-safe secara arsitektur
- masih bergantung ke route web untuk flow inti desktop
- state UI tidak sinkron dengan data nyata

---

## 16. Distribution Vision

Target final EduCore:

### Web
- di-deploy untuk browser desktop dan HP
- dipakai oleh user yang belum install app

### Desktop
- installer Tauri
- local SQLite + sync
- dipakai operator utama sekolah

### Multi-user model
- banyak user / banyak device
- data sekolah yang sama
- cloud sebagai shared collaboration layer
- desktop sebagai offline operational layer

---

## 17. Instructions for Future Chat

Jika chat baru dimulai dan user menunjuk file ini, AI harus langsung menganggap:
- repo ini **hybrid desktop + web**
- target arsitektur adalah **desktop-native local runtime + cloud sync**
- **Turso** adalah shared cloud DB
- **SQLite desktop** adalah local operational DB
- **web dan desktop tidak boleh dicampur boundary-nya**
- semua task harus dieksekusi dengan audit dulu, bukan asumsi

Jika ada konflik antara instruksi lama di file lain dengan file ini:
- prioritaskan **kondisi aktual repo** dan **runtime safety**

---

## 18. Quick Commands

```bash
bun run dev
bun tauri dev
bunx biome check .
bunx tsc --noEmit
bun run build
bun run build:desktop
```

Catatan:
- desktop production path saat ini sudah sehat untuk `MSI`.
- `NSIS` belum otomatis ikut sehat hanya karena `MSI` lulus.
- jangan menyebut “desktop final” tanpa menyebut channel installer yang benar-benar lolos.

---
> Source: [ciola1999/educore](https://github.com/ciola1999/educore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
