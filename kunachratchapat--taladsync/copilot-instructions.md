## taladsync

> ระบบจัดการแผงตลาดสดรายวัน แก้ pain point: การแย่งจองแผงขาจร (concurrency) + การเช็คชื่อหน้างาน (roll-call) ว่าใครมา/ไม่มาตรงกับระบบ

# AGENTS.md — TaladSync

ระบบจัดการแผงตลาดสดรายวัน แก้ pain point: การแย่งจองแผงขาจร (concurrency) + การเช็คชื่อหน้างาน (roll-call) ว่าใครมา/ไม่มาตรงกับระบบ

> **เก็บเงิน 20 บาท:** เจ้าหน้าที่เก็บสดหน้างานเอง — **ไม่ track ในระบบ MVP** (ย้ายไป Phase 2 ถ้าต้องการ financial summary)

## เอกสารอ้างอิง (อ่านก่อนเริ่ม)

- [docs/SYSTEM_DESIGN.md](docs/SYSTEM_DESIGN.md) — ภาพรวม, stack, schema (attendance roll-call MVP)
- [docs/BOOKING_FLOW.md](docs/BOOKING_FLOW.md) — state machine, transaction strategy, concurrent test

---

## Implementation Checkpoint (อัปเดตล่าสุด)

**สถานะรวม:** Backend MVP **Phase 0–8 DONE**. Frontend **F1–F4 DONE** (merged → `main`).
**กำลังทำ: UI redesign บน branch `bookingvendor`** (push แล้ว)

- `ensureWalkin` → refactor เป็น `ensureRole(role)` (auth store เก็บ role คู่ token กัน 403 ข้าม role)
- **DevNav ถูกลบแล้ว** (commit `9698a12`) — เข้าแต่ละหน้าด้วย URL ตรงๆ (`/`, `/staff`, `/regular`, `/directory`)

### 🎨 UI Redesign — branch `bookingvendor` (กำลังทำ)

เพื่อนทำ UX/UI ออกแบบให้ · ทำ**ทีละ component** แล้วให้ user review ก่อน commit

**เสร็จแล้ว** (commit `5cce091`):

| ไฟล์ | ทำอะไร |
|------|--------|
| `components/PageHeader.vue` 🆕 | header ตามดีไซน์: ปุ่ม `<` (absolute ซ้าย) + title กลางจอ · emit `back` |
| `views/WalkinView.vue` | list → **กริด 3 คอลัมน์ แบ่งตามโซน** (`computed stallsByZone`) + พื้นหลัง `bg-brand-50` |
| `components/StallCard.vue` | list row → **การ์ดกริด** 3 สถานะ (ว่าง / selected / disabled) |
| `database/seed.go` | **โซน A (A-01..A-15 มีเจ้าของ) + โซน B (B-01..B-15 ว่าง)** แทน Zone A + Pool เดิม |

**⚠️ สำคัญ:** `AppHeader.vue` **ยังใช้อยู่** ในอีก 3 หน้า (staff/regular/directory) — **ห้ามลบ** จนกว่าจะ redesign ครบทุกหน้า

**🔴 ติดรอดีไซน์จากเพื่อน (ทำต่อไม่ได้):**
1. **Flow การจอง** — กดแผงแล้วเกิดอะไร? (เลือกก่อนค่อยกดปุ่ม vs เด้ง modal ทันที) · ตอนนี้คงของเดิม = กดแล้วเด้ง `BookingModal`
2. **สถานะ "เลือกแล้ว"** หน้าตายังไง (ตอนนี้เดาไว้: `bg-brand-50` + `ring-2 ring-brand-500`)
3. **ปุ่ม "จองแผง"** ข้างล่าง (ขึ้นกับข้อ 1)
4. ดีไซน์อีก 3 หน้า + `BookingModal`

### Next session — ทำได้เลย (ไม่ต้องรอดีไซน์)

1. **Vitest test ตัวแรก** — frontend ยังไม่มี test (ปิดจุดอ่อนตอนสัมภาษณ์ + อยู่ใน roadmap)
2. ~~**BOOK audit**~~ — **DONE**: จองสำเร็จเขียน `audit_logs` action `BOOK` แล้ว (ครบทั้ง BOOK/LEAVE/CHECKIN/NOSHOW)
3. **CI/CD** — GitHub Actions รัน `go test` + `npm run type-check` ทุก push
4. **Deploy จริง** — Railway/Render → มี URL ใส่ resume
5. ใช้ design guidance ใน `frontend/SKILL.md` ตอนสร้าง UI ใหม่

### Backend เพิ่มระหว่าง F1 (commit แล้ว `59eced9`)

| งาน | ไฟล์ | หมายเหตุ |
|------|------|----------|
| `GET /api/bookings/mine?date=` | `handler/booking.go`, `usecase/booking.go`, `repository/booking.go`, `main.go` | WALKIN เช็กจองวันนี้แล้วหรือยัง → `{ booking: null \| DailyBooking }` |
| CORS | `cmd/api/main.go` | `AllowOrigins` จาก `CORS_ORIGINS` (default `http://localhost:5173`) |
| Seed today | `database/seed.go` | `SeedDailyBookingsForToday` รันทุกครั้งที่ API start |

### Phase 8 — DONE (commit แล้ว)

| ไฟล์ | งาน |
|------|-----|
| [`docker-compose.yml`](docker-compose.yml) | `postgres:17-alpine` + `api`, secrets จาก `.env` |
| [`backend/Dockerfile`](backend/Dockerfile) | multi-stage Go 1.24 → alpine, `TZ=Asia/Bangkok` |
| [`backend/.dockerignore`](backend/.dockerignore) | กัน `.env`, binaries |
| [`.env.example`](.env.example) | template secrets (commit ได้) — copy → `.env` |

**Secrets:** compose ใช้ `${POSTGRES_PASSWORD}`, `${JWT_SECRET}` จาก `.env` (root) — **ไม่ hardcode ใน repo**

**สองไฟล์ env:**

| ไฟล์ | ใช้เมื่อ |
|------|---------|
| `.env` (root) | `docker compose up` |
| `backend/.env` | `go run ./cmd/api` |

**Pitfalls Docker:** เปลี่ยน `POSTGRES_PASSWORD` หลัง volume สร้างแล้ว → `docker compose down -v` แล้ว up ใหม่; port 5432/8080 ชน local ได้

### Incremental roadmap

| Phase | สถานะ | เป้าหมาย |
|-------|--------|----------|
| 0 | **DONE** | Health API — `cmd/api/main.go` + `GET /health` |
| 1 | **DONE** | Domain models (4 tables) + Postgres connect + migrate |
| 2 | **DONE** | Seed — **1 แม่ค้าประจำ : 1 แผง** (12 REGULAR เจ้าของ A-01..A-12 คนละแผง) + 1 WALKIN + 1 STAFF + pool P-01..P-08; mock-login REGULAR = คนแรก (เจ้าของ A-01) |
| 3 | **DONE** | Auth mock-login + JWT middleware + `GET /api/auth/me` |
| 4 | **DONE** | Booking — list/book, 409, `date?` body, partial unique index (1 vendor/วัน), unit tests |
| 5 | **DONE** | Vendor leave + staff checklist/attendance/summary + audit (LEAVE/CHECKIN/NOSHOW) |
| 6 | **DONE** | Directory — `GET /api/directory?q=` public, ไม่มี PII |
| 7 | **DONE** | Concurrent test — 50 goroutines, 1 winner (`-tags=integration`) |
| 8 | **DONE** | DevOps — compose + Dockerfile + `.env.example` |

### Frontend roadmap

| Phase | สถานะ | เป้าหมาย |
|-------|--------|----------|
| F1 | **DONE** | WALKIN — auto mock-login, รายการแผงว่าง, ฟอร์มจอง, 409 ไทย, จองแล้วแสดงการ์ด |
| F2 | **DONE** | STAFF — checklist roll-call (progress + toggle มา/ไม่มา), summary นับ client-side |
| F3 | **DONE** | REGULAR — `GET /api/vendor/mine` แผงตัวเอง + แจ้งหยุด (modal ยืนยัน, optimistic) — branch `regular-frontend` |
| F4 | **DONE** | Public — directory ค้นหาร้าน (search debounce, ไม่มี login) — branch `directory-frontend`; แผนที่ posX/posY ไว้ Phase 2 |

**Stack:** Vue 3 + Vite + TypeScript + Tailwind + Pinia + Vue Router + `@jamescoyle/vue-icon` + `@mdi/js`

**โฟลเดอร์ `frontend/` (มีแล้ว):**

```
frontend/
├── src/
│   ├── api/
│   │   ├── client.ts          # fetch + Bearer token + map 409
│   │   ├── auth.ts            # mock-login
│   │   ├── booking.ts         # stalls/available, bookings, bookings/mine
│   │   └── errors.ts
│   ├── stores/auth.ts         # Pinia: ensureWalkin + localStorage token
│   ├── composables/            # useAvailableStalls, useBooking
│   ├── components/            # AppHeader, StallCard, BookingModal, BookedTodayCard, ...
│   ├── views/WalkinView.vue
│   ├── utils/date.ts, errors.ts
│   └── style.css              # .market-bg พื้นหลังตลาด
├── .env.example               # VITE_API_URL=http://localhost:8080
└── package.json, vite.config.ts, tailwind.config.js, package-lock.json
```

**F1 user flow (ต่อ API จริงแล้ว):**

```
1. เปิดหน้า → ensureWalkin (token ใน localStorage หรือ POST mock-login WALKIN)
2. GET /api/bookings/mine?date=   → ถ้ามี booking แสดง BookedTodayCard (1 แผง/วัน)
3. ถ้ายังไม่จอง → GET /api/stalls/available?date=  → รายการแผง pool + badge จำนวน
4. กดแผง → BookingModal → POST /api/bookings → 201 success หรือ 409 ไทย
```

**F1 UI decisions (ตกลงกับ user แล้ว):**

- วันที่ = **วันนี้เท่านั้น** (ไม่มี date picker)
- พื้นหลัง `.market-bg` — รูปตลาด Trang + overlay จาง
- หัวข้อกลางจอ + badge จำนวนแผงว่าง
- กฎ 1 ขาจร 1 แผง/วัน — จองแล้วแสดง `BookedTodayCard` แทนรายการแผง
- **Out of scope F1:** แผนที่, STAFF/REGULAR, LINE LIFF, role picker, PWA/LIFF prep

**Backend สำหรับ F1:**

- [x] **CORS** — `fiber/middleware/cors` ใน `main.go`
- [x] **Seed today** — `SeedDailyBookingsForToday`
- [x] **GET /api/bookings/mine** — เช็กการจองของ vendor วันนี้

### Backend optional polish (ไม่ block F2)

1. ~~Seed daily bookings for today~~ — **DONE**
2. ~~**BOOK audit**~~ — **DONE**: `BookStall` เขียน `audit_logs` action `BOOK` หลังจองสำเร็จ

### DONE — Backend (มีไฟล์แล้ว)

| Layer | ไฟล์ | หมายเหตุ |
|---|---|---|
| **entrypoint** | `backend/cmd/api/main.go` | Fiber routes — **middleware ต่อ route** (ดูด้านล่าง) |
| **devops** | `docker-compose.yml`, `backend/Dockerfile`, `backend/.dockerignore`, `.env.example` | Postgres 17 + API; secrets ใน `.env` |
| **config** | `backend/internal/config/config.go` | env loader (APP_PORT, DB_*, JWT_SECRET) |
| **domain** | `backend/internal/domain/models.go`, `errors.go` | entities, audit constants, inputs, `DirectoryEntry` DTO |
| **database** | `backend/internal/database/database.go`, `seed.go` | Connect, Migrate (+ partial unique index), SeedIfEmpty |
| **auth** | `backend/internal/auth/jwt.go` | TokenService Sign/Parse (HS256) |
| **repository** | `user.go`, `booking.go`, `audit.go` | user; booking CRUD+leave+attendance+SearchDirectory; audit Create |
| **usecase** | `auth.go`, `booking.go`, `vendor.go`, `staff.go`, `directory.go`, `*_test.go`, `booking_concurrent_test.go` | MockLogin; BookStall; Leave; MarkAttendance; Search + unit/integration tests |
| **handler** | `auth.go`, `booking.go`, `vendor.go`, `staff.go`, `directory.go`, `date.go`, `errors.go` | all MVP endpoints + error mapping |
| **middleware** | `auth.go`, `role.go` | JWT Auth, RequireRoles |
| **timeutil** | `backend/internal/pkg/timeutil/market_date.go` | `Today()`, `Now()` Asia/Bangkok |
| **env** | `backend/.env.example`, `.env.example` (root) | local dev vs docker compose |
| **deps** | `backend/go.mod`, `go.sum` | Fiber, godotenv, GORM, postgres, jwt/v5, testcontainers-go |

### TODO

- **merge `directory-frontend` → `main`** (F4 พร้อม) + push
- ~~Backend polish: BOOK audit~~ — **DONE**
- **Product Phase 2** — LINE LIFF, map (posX/posY), financial, PDPA (ดู section ท้ายไฟล์)

---

## Route wiring (`cmd/api/main.go`) — สำคัญ

**อย่าใช้ `api.Group("")` หลายครั้งพร้อม `RequireRoles` คนละ role** — Fiber รวม middleware ของ group prefix ว่าง ทำให้ staff route โดน `RequireRoles(WALKIN)` → **403 forbidden**

รูปแบบที่ใช้ตอนนี้:

```go
authMW := middleware.Auth(tokenSvc)

api.Get("/directory", directoryHandler.Search)  // public — ไม่มี authMW

api.Get("/stalls/available", authMW, middleware.RequireRoles(domain.RoleWalkin), ...)
api.Get("/bookings/mine", authMW, middleware.RequireRoles(domain.RoleWalkin), ...)  // F1: เช็กจองวันนี้
api.Post("/bookings", authMW, middleware.RequireRoles(domain.RoleWalkin), ...)
api.Get("/vendor/mine", authMW, middleware.RequireRoles(domain.RoleRegular), ...)  // F3: แผงของ vendor
api.Post("/vendor/leave", authMW, middleware.RequireRoles(domain.RoleRegular), ...)
api.Get("/staff/checklist", authMW, middleware.RequireRoles(domain.RoleStaff), ...)
api.Get("/staff/summary", authMW, middleware.RequireRoles(domain.RoleStaff), ...)
api.Post("/bookings/:id/attendance", authMW, middleware.RequireRoles(domain.RoleStaff), ...)
```

`/api/auth/me` ใช้ group แยก `api.Group("/auth", authMW)` — ไม่มี RequireRoles

## Tech Stack

- **Backend:** Go 1.24 + Fiber v2 + GORM + PostgreSQL 17
- **Frontend:** Vue 3 + Vite + TypeScript + Tailwind + Pinia + Vue Router — โฟลเดอร์ `frontend/` **F1 DONE**
- **Auth:** JWT mock login (MVP) → production LINE LIFF
- **Project root:** `D:\taladsync`

## Architecture: Modular Monolith + Clean Architecture

**Dependency rule:** ลูกศรชี้เข้าด้านใน `handler → usecase → domain` ; `repository` implement interface ของ `usecase`

```
backend/                     ← DONE (Phase 0–8) + bookings/mine
frontend/                    ← F1 DONE (WALKIN) · F2 TODO (STAFF)
```

**Backend** (`cmd/api/main.go` …) — DONE: health, auth, booking, vendor, staff, directory, bookings/mine

**Frontend** — F1 WALKIN done (`WalkinView`); ถัดไป F2 STAFF


## Core Concurrency Rule (สำคัญที่สุด)

จองแผงใช้ **conditional atomic UPDATE** ไม่ใช่ lock ใน app:

```sql
UPDATE daily_bookings
SET status = 'BOOKED', vendor_id = ?, shop_name = ?, product_type = ?, booked_at = now()
WHERE stall_id = ? AND market_date = ? AND status = 'AVAILABLE';
```

- `RowsAffected = 1` → สำเร็จ
- `RowsAffected = 0` → `ErrStallNotAvailable` (HTTP 409)
- **ห้าม** ทำ I/O ภายนอก (LINE push) ใน transaction ที่ถือ row lock
- Double-booking กันสองชั้น: conditional `WHERE` + `UNIQUE(stall_id, market_date)`
- 1 ขาจร 1 แผง/วัน: partial unique index `idx_vendor_one_booked_per_day` บน `(vendor_id, market_date) WHERE status='BOOKED'`

## State Machine

```
Stall status (daily_bookings.status):
  EXPECTED  → AVAILABLE  (แม่ค้าแจ้งหยุด / cron)
  AVAILABLE → BOOKED     (ขาจรจอง — conditional UPDATE)
  EXPECTED  → ABSENT     (Force Absent — Phase 2)

Attendance (daily_bookings.attendance) — แยก lifecycle:
  PENDING → PRESENT  (staff เช็คว่ามา)
  PENDING → NOSHOW   (staff เช็คว่าไม่มา — บันทึก+audit เท่านั้น ไม่เปลี่ยน status)
```

## Schema สำคัญ (`DailyBooking`)

```go
Status           BookingStatus    // EXPECTED | AVAILABLE | BOOKED | ABSENT
Attendance       AttendanceStatus // PENDING | PRESENT | NOSHOW
CheckedByStaffID *uint
CheckedAt        *time.Time
// ลบแล้ว: PaymentStatus, PaidByStaffID, PaidAt
```

## Conventions

- เวลาใช้ `Asia/Bangkok` เสมอ; `market_date` เป็น type `date` (`internal/pkg/timeutil`)
- Business error อยู่ใน `internal/domain/errors.go`; **handler** เป็นที่เดียวที่ map → HTTP status
- `national_id` = PII: ห้าม return ออก public API (`json:"-"`); production ต้อง encrypt
- Idempotent: การเช็คชื่อ (`MarkAttendance`) และการจอง (middleware `Idempotency-Key`) ต้องกดซ้ำได้ไม่พัง
- Audit: `BOOK`, `CHECKIN`, `NOSHOW`, `LEAVE` → `audit_logs` ครบทุก state transition แล้ว
  - **Known limitation (MVP):** audit อยู่บน critical path — ถ้า `audit.Create` fail หลังจองสำเร็จ request จะได้ error ทั้งที่แผงถูกจองแล้ว (consistent กับ `vendor.Leave`/`staff` ทั้งระบบ ไม่มี tx ครอบ)
  - `BookStall` ยิง `FindVendorBookedToday` เพิ่ม 1 query เพื่อดึง detail ของ audit — optimize ได้ถ้าให้ `TryBookStall` คืน booking กลับมา
- `parseMarketDate` ใน `handler/date.go` — query/body `date` ว่าง = วันนี้ Bangkok; ใช้ร่วม list/book/leave/checklist

## API (MVP — implemented vs TODO)

| Method | Endpoint | Role | สถานะ | คำอธิบาย |
|--------|----------|------|--------|----------|
| GET | `/health` | public | **DONE** | health check |
| POST | `/api/auth/mock-login` | public | **DONE** | `{role}` → `{token, user}` |
| GET | `/api/auth/me` | any JWT | **DONE** | profile จาก token |
| GET | `/api/stalls/available?date=` | WALKIN | **DONE** | แผงว่างใน pool |
| GET | `/api/bookings/mine?date=` | WALKIN | **DONE** | การจองของ vendor วันนี้ (`{ booking: null \| ... }`) |
| POST | `/api/bookings` | WALKIN | **DONE** | `{stallId, shopName, productType, date?}` → 409 ถ้าเต็ม |
| GET | `/api/vendor/mine?date=` | REGULAR | **DONE** | แผงที่ vendor เป็นเจ้าของ + สถานะวันนี้ (query by `stalls.owner_vendor_id`) → `{ stalls: [...] }` |
| POST | `/api/vendor/leave` | REGULAR | **DONE** | `{stallId, date?}` แจ้งหยุด (EXPECTED→AVAILABLE) |
| GET | `/api/staff/checklist?date=` | STAFF | **DONE** | roll-call ทุกแผงที่มีคน |
| POST | `/api/bookings/:id/attendance` | STAFF | **DONE** | `{present: true/false}` |
| GET | `/api/staff/summary?date=` | STAFF | **DONE** | `{present, noShow, pending, total}` |
| GET | `/api/directory?q=` | public | **DONE** | ค้นหาร้าน (ไม่มี PII) — `?date=` optional |

## การรัน (dev)

### Docker Compose (full stack)

```powershell
copy .env.example .env
# แก้ POSTGRES_PASSWORD, JWT_SECRET ใน .env (ไม่ commit .env)

docker compose up --build
# GET http://localhost:8080/health → {"status":"ok"}
```

หยุด local Postgres/API ก่อนถ้า port `5432` / `8080` ชน — reset volume: `docker compose down -v`

### Local (go run)

```powershell
copy backend\.env.example backend\.env

# สร้าง database (ครั้งแรก)
createdb -U postgres taladsync

# รัน API (seed อัตโนมัติถ้า users ว่าง)
cd backend
go run ./cmd/api
# GET http://localhost:8080/health → {"status":"ok"}
```

**หมายเหตุ dev:** `SeedDailyBookingsForToday` สร้าง `daily_bookings` วันนี้อัตโนมัติทุกครั้งที่ API start

### Frontend (Vite dev)

```powershell
cd frontend
copy .env.example .env   # VITE_API_URL=http://localhost:8080
npm install
npm run dev
# เปิด http://localhost:5173/
```

ต้องรัน backend ที่ `:8080` พร้อมกัน — **restart API** หลัง pull โค้ดใหม่เพื่อให้ `GET /api/bookings/mine` ทำงาน

**ทดสอบจองใหม่ (dev):** ลบ booking ของ WALKIN ใน DB หรือ `docker compose down -v` แล้ว up ใหม่ · ลบ `localStorage` key `taladsync_token`

## Mock Login (ทดสอบ)

```json
POST /api/auth/mock-login
{ "role": "WALKIN" }
```

Role ที่ใช้ได้: `REGULAR` | `WALKIN` | `STAFF`  
Header: `Authorization: Bearer <token>`

## Booking (ทดสอบ — WALKIN)

```
1. POST /api/auth/mock-login  { "role": "WALKIN" }
2. GET  /api/stalls/available?date=YYYY-MM-DD   (Bearer token)
3. POST /api/bookings  { "stallId": 13, "shopName": "ร้านทด", "productType": "ผลไม้", "date": "YYYY-MM-DD" }
```

Unit test: `go test ./internal/usecase/... -v`

## Phase 5 smoke test

ใช้ `YYYY-MM-DD` ตรงกับ `market_date` ใน DB (ดูด้วย `SELECT DISTINCT market_date FROM daily_bookings`)

```
# REGULAR แจ้งหยุด (stallId = แผง A-01..A-12, ไม่ใช่ daily_bookings.id)
POST /api/auth/mock-login  { "role": "REGULAR" }
POST /api/vendor/leave     { "stallId": 1, "date": "YYYY-MM-DD" }

# WALKIN ดู pool หลัง leave
POST /api/auth/mock-login  { "role": "WALKIN" }
GET  /api/stalls/available?date=YYYY-MM-DD

# STAFF roll-call (:id = daily_bookings.id จาก checklist ไม่ใช่ stallId)
POST /api/auth/mock-login  { "role": "STAFF" }
GET  /api/staff/checklist?date=YYYY-MM-DD
POST /api/bookings/:id/attendance  { "present": true }
GET  /api/staff/summary?date=YYYY-MM-DD
```

## Phase 6 smoke test

ใช้ `YYYY-MM-DD` ตรงกับ `market_date` ใน DB (ดูด้วย `SELECT DISTINCT market_date FROM daily_bookings`) — **ไม่ต้อง JWT**

```
GET /api/directory?q=ของสด&date=YYYY-MM-DD
GET /api/directory?date=YYYY-MM-DD          # q ว่าง = list ทุกร้านวันนั้น
GET /api/directory?q=ร้านA-01&date=YYYY-MM-DD
```

Response: `{"results":[{"stallCode","zone","shopName","productType","posX","posY"}]}`

Unit test: `go test ./internal/usecase/... -v -run Search_`

## Phase 7 concurrent test (integration)

พิสูจน์ conditional UPDATE กัน double-booking — **ต้อง Docker Desktop รันอยู่**

```powershell
cd backend

# unit tests ปกติ (ไม่ต้อง Docker)
go test ./internal/usecase/... -v

# killer demo — 50 goroutines จองแผงเดียว, assert 1 winner
go test -tags=integration ./internal/usecase/... -v -run Concurrent -count=1
```

ถ้า Docker ไม่พร้อม test จะ **skip** (ไม่ fail) พร้อมข้อความให้เปิด Docker Desktop

## Phase 8 Docker Compose

```powershell
# จาก repo root
copy .env.example .env
# แก้ POSTGRES_PASSWORD, JWT_SECRET

docker compose up --build
# GET http://localhost:8080/health → {"status":"ok"}
```

```powershell
docker compose down          # หยุด
docker compose down -v       # หยุด + ลบ volume (reset DB / แก้รหัสผ่าน PG)
docker compose up --build -d # รัน background
docker compose logs -f api   # ดู log
```

**Verify แล้ว:** build ผ่าน, seed อัตโนมัติ, health OK

## Phase 2 Product (ยังไม่ทำ — หลัง Frontend MVP)

- LINE LIFF / Push จริง (Flash Drop)
- โควตาวันหยุด, แจ้งหยุดฉุกเฉิน 2 ครั้ง/เดือน, ลาพักยาว+อนุมัติ
- Force Absent + ปล่อยแผง no-show อัตโนมัติ
- แจ้งมาช้า (late arrival lock)
- **Financial tracking** (เก็บเงิน 20 บ. ในระบบ + สรุปยอดส่งคลัง) — ตัดออกจาก MVP แล้ว
- national_id encryption เต็มรูปแบบ (PDPA)

---
> Source: [KunachRatchapat/TaladSync](https://github.com/KunachRatchapat/TaladSync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
