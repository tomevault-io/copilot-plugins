## smart-pole

> Monorepo: `backend/` `frontend/` `e2e/` `docs/`

# Project Convention

Monorepo: `backend/` `frontend/` `e2e/` `docs/`
Runtime: Bun — ใช้ `bun`/`bunx` เท่านั้น ห้ามใช้ `node`/`npm`/`npx`

> รายละเอียดเฉพาะแต่ละ sub-project ดูที่ `CLAUDE.md` ในแต่ละโฟลเดอร์

---

## Repository Structure

```
backend/src/
├── modules/<name>/     controller, service, repository, schema, index.ts
│   └── <concern>/      sub-folder สำหรับ large module (ดู backend/CLAUDE.md)
├── common/             errors, middleware, schemas, utils
├── plugins/            prisma, auth, logger, websocket, storage
└── config/             env validation

frontend/
├── app/(auth)/         login
├── app/(dashboard)/    protected pages
├── components/layout/  AppForm, DataTable, AppSider, AppHeader, TableActions, ...
├── components/shared/  reusable components ที่ใช้ข้าม module
├── components/ui/      shadcn — ห้ามแก้ไข
├── lib/api/            client.ts + module API functions
├── hooks/api/          TanStack Query hooks per module
├── hooks/              shared hooks (useAppAlertDialog, useDebounce)
├── stores/             Zustand stores (auth-store)
└── config/             env validation

e2e/                    1 module = 1 folder, 1 action = 1 spec
docs/                   test-scenarios (.md→.xlsx), delivery (.xlsx)
planning/               PLANNING.md + epics/<E##>/tasks/<T##>.md

infra/
├── mosquitto/config/   mosquitto.conf + aclfile + passwordfile
├── srs/srs.conf        SRS 5 — HLS-only mode (DVR ใน Pi side)
├── pole-firmware/      Pi scripts (main.py, stream-rtmp.sh, record-mp4.sh,
│                       sync-recordings.sh, cleanup-recordings.sh,
│                       *.service systemd units, provision-pi.sh, README)
└── host-scripts/       Scripts ที่ run บน DO host (cleanup-camera-clips.sh)
```

---

## Naming

| สิ่งที่ตั้งชื่อ | รูปแบบ |
|---|---|
| variable, function | camelCase |
| Component, Type, Interface, Enum | PascalCase |
| Enum value, Constant | UPPER_SNAKE_CASE |
| File, Folder | kebab-case |

- boolean ขึ้นต้นด้วย `is` `has` `can` `should`, event handler ขึ้นต้นด้วย `handle`
- interface สำหรับ object shape — type สำหรับ union/intersection/utility

---

## Git

Branch: `main` (production) ← `develop` ← `feature/*` `fix/*` `chore/*`
Commit: `<type>(<scope>): <description>` — present tense, lowercase, ≤72 chars
Types: `feat` `fix` `chore` `refactor` `test` `docs` `style`

- ห้าม push ตรงไปที่ `main`, ห้าม commit `.env`, ห้าม AI-generated tag (`Co-Authored-By: Claude` ฯลฯ)

---

## Absolute Prohibitions (ทั้งโปรเจค)

### Atomic Code (กฎข้อสำคัญที่สุด — สำคัญกว่า clean code)

**ทุกฟังก์ชันต้องทำ "1 อย่าง" เท่านั้น — ทั้ง frontend และ backend**

- ฟังก์ชัน 1 ตัว = **1 step / 1 action / 1 side-effect** — ห้ามรวม multi-step ในฟังก์ชันเดียว
- ถ้าฟังก์ชันมี > 1 step → **ต้อง** แยก step ละ atom function แล้วให้ orchestrator เรียกประกอบ
- ระดับการตัดสิน: ถ้าตั้งชื่อฟังก์ชันต้องใช้คำว่า "และ" หรือ description มี "+ " อยู่ → ต้องแยก
- Atom = **testable แยกได้, แก้ไขเฉพาะจุดได้, mock ได้** โดยไม่ต้องรู้ทั้ง flow
- Orchestrator เรียก atoms เรียงลำดับ + จัดการ error/result — ไม่มี business logic ของตัวเอง

**Why atomic > clean code:**
- Test failure ระบุได้ตรงจุด — atom ไหนพัง รู้ทันที
- แก้ bug ที่ atom เดียวไม่กระทบ atom อื่น — control surface เล็ก
- Reuse atoms ใน flow อื่นได้ — DRY ตามมาเอง

**Pattern ครบ + Performance:** `backend/CLAUDE.md` § "Atomic Refactor Pattern" + "Performance Optimization Patterns"

### TypeScript
- ห้าม `any` / `as` / `!` non-null assertion / `@ts-ignore` / `eslint-disable` ไม่มี comment
- **ข้อยกเว้น:** `as const` อนุญาต, `any` กรณี library บังคับ (ต้องมี eslint-disable + comment)

### DRY & SRP
- โค้ดใช้ซ้ำ ≥ 2 ที่ ต้อง extract ทันที — logic เหมือนแต่ต่าง parameter → ใช้ parameter/config
- 1 function 1 ความรับผิดชอบ — ถ้าต้องใช้คำว่า "และ" แสดงว่าควรแยก (ข้อยกเว้น: function ≤ 10 บรรทัด)

### File/Function Granularity (ย่อยที่สุดเพื่อ test ได้)

- **ห้ามเพิ่ม flag/branch ใน generic component/service** เพื่อรองรับ variant ใหม่ — ให้แยกไฟล์ใหม่ (ระดับ duplicate ยอมรับได้: < 30% LoC)
- **Module ขนาด > 500 LoC หรือ ≥ 3 concerns:** แยก sub-folder per concern (เช่น `flow/forward.ts`, `flow/backward.ts`)
- **1 function 1 concern → 1 test file**
- **Sub-folder ห้าม import กันเอง** — ป้องกันวงกลม + ให้ orchestrator (parent service) เป็นจุดเดียวที่ compose
- **ทุก function ใน sub-folder ต้องรับ `tx?: PrismaTx`** + return explicit type

### Hardcode & Raw Query
- ห้าม hardcode ค่าจาก DB (status key, icon, color, label, role, threshold), ห้าม hardcode config map ใน frontend
- ห้าม raw query ทุกรูปแบบ — ใช้ Prisma query API เท่านั้น

### Audit Log
- ทุก mutation ต้องมี `auditService.log()` — fire-and-forget (ห้าม await)

### Minimal Data Exposure (OWASP API3)

- **ทุก query ต้องมี `select`** — แยก `FOO_LIST_SELECT` (ตาราง) / `FOO_DETAIL_SELECT` (form) — ห้าม reuse detail ใน list
- ห้าม return `passwordHash`, `token`, `secret` — relation ต้องมี `select` (ห้าม `include: { user: true }`)
- ห้าม return nested array ที่ client ไม่ใช้ (ใช้ `_count` แทน), ห้าม expose stack trace/SQL ใน production
- **frontend type มีเฉพาะ field ที่ UI ใช้จริง**
- timestamp (`createdAt`/`updatedAt`) → return เฉพาะเมื่อ frontend แสดงจริง

### List vs Lookup vs Detail

| ระดับ | Endpoint | จังหวะโหลด | วิธีทำ |
|---|---|---|---|
| List (ตาราง) | `GET /` | ทันที | backend batch count/flags พร้อม list |
| Lookup (combobox) | `GET /lookup` | dialog เปิด / filter mount | **id+label เท่านั้น — ไม่มีข้อยกเว้น** |
| Detail (form) | `GET /:id` | คลิกเปิด | `useQuery({ enabled: isOpen })` lazy-load |

- **Combobox / Dropdown / Filter ทุกตัวต้องใช้ `GET /lookup` เท่านั้น**
- Backend: ทุก module ต้องมี `/lookup` endpoint — return เฉพาะ field ที่ combobox ใช้
- Frontend: combobox hook ต้องลงท้าย `Lookup` เสมอ (เช่น `useBranchLookup()`)

### Lazy-load — ห้ามโหลดข้อมูลที่ยังไม่แสดง

**ทุก `useQuery` ที่ยังไม่ปรากฏต้องมี `enabled` gate** — Dialog/Panel/Tab/Lookup ที่ยังไม่เปิด

### List Response — ข้อมูลพร้อมใช้

- aggregate (sum, count, %) → backend คำนวณส่งมาเลย — ห้าม frontend reduce/filter+count

### Concurrency / Optimistic Locking

- Detail endpoint return `updatedAt` → client ส่งกลับตอน update → backend เช็คก่อน save → ถ้าไม่ตรง throw `ConflictError`
- **ข้อยกเว้น:** reorder, workflow action, bulk operations, status enum change

### Decimal / Money Handling (มาตรฐานบัญชี)

**ห้ามใช้ `Float` กับเงิน — ใช้ `Decimal` เท่านั้น**

| ประเภท | Prisma Type |
|---|---|
| จำนวนเงิน | `Decimal @db.Decimal(18,2)` |
| อัตรา/เปอร์เซ็นต์ | `Decimal @db.Decimal(10,4)` |
| ปริมาณชิ้น | `Int` |
| ปริมาณทศนิยม | `Decimal @db.Decimal(12,2)` |

- คำนวณด้วย `Prisma.Decimal` — ห้าม JS native `+` `*` `/` กับเงิน
- **ห้ามปัดเศษทุกตัวเลขเด็ดขาด** — ใช้ `Math.trunc(value * 100) / 100` ตัด 2 ทศนิยมเท่านั้น (ห้าม `Math.round`, ห้าม `toFixed`)
- **API serialize:** `Decimal.prototype.toJSON` override ใน `src/plugins/prisma.ts` แปลง Decimal → `number`
- frontend type ประกาศ Decimal fields เป็น `number` — แสดง 2 ทศนิยมเสมอ
- **ห้าม frontend คำนวณยอดรวม/VAT สำหรับ save** — preview ได้แต่ label "ยอดประมาณ"

### Date / Timezone

**DB เก็บ UTC — Frontend แปลง `Asia/Bangkok` ตอนแสดงผล**

| ชั้น | Format |
|---|---|
| DB / API | UTC ISO 8601 |
| Date-only field | `YYYY-MM-DD` string |
| Frontend display | Thai locale UTC+7 |

- ใช้ `formatDate()`/`formatDateTime()` จาก `@/lib/format` — ห้าม `toLocaleDateString`
- business logic เปรียบเทียบวันที่ → ทำที่ backend

### Dependency Management

- `bun.lock` ต้อง commit, pin exact version (ห้าม `^` `~`), `bun audit` ก่อน deploy
- เพิ่ม dep ใหม่ → ตรวจ license (GPL ❌), TypeScript types, last publish < 2 ปี

---

## Work Strategy

### Parallel Agents
- งาน > 1 ที่ไม่กระทบกัน → แยก agent ทำขนาน — แบ่งตาม module/concern, ให้ context ครบ

### Post-task Verification
- หลังเสร็จ → `bunx tsc --noEmit` ทั้ง backend + frontend ต้อง 0 errors + รัน test ที่เกี่ยวข้อง

---
> Source: [teerawat-k/smart-pole](https://github.com/teerawat-k/smart-pole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
