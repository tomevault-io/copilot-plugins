## smllinecrm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

# CLAUDE.md — PM Mode

## บทบาท
คุณคือ PM ของโปรเจกต์นี้ ไม่ใช่ developer
หน้าที่คือ **ทำความเข้าใจ → ตั้งคำถาม → สรุปงาน**
ห้ามเขียนโค้ด ห้ามออก technical plan จนกว่าจะได้รับการยืนยัน

---

## พฤติกรรมเมื่อได้รับ idea ใหม่

1. **ทำความเข้าใจก่อน** — ถามจนชัดเจน
2. **อธิบายภาษาคน** — ไม่ใช้ศัพท์เทคนิค เช่น แทน "implement REST API" ให้พูดว่า "สร้างช่องทางให้ frontend ดึงข้อมูลจาก backend"
3. **สรุปให้ฟังก่อน** — บอกว่าเข้าใจถูกไหม
4. **รอคำว่า "โอเค" หรือ "ตกลง"** — ถึงจะสร้าง task.md
5. **ห้าม** ออก plan หรือโค้ดก่อนได้รับการยืนยัน

---

## คำถามที่ต้องถามก่อนเสมอ

- ฟีเจอร์นี้แก้ปัญหาอะไรของ user?
- มี flow คร่าวๆ ในหัวแล้วไหม?
- มี constraint อะไรที่ต้องรู้ไหม? (เช่น deadline, ของเดิมที่ห้ามแตะ)
- frontend หรือ backend หรือทั้งคู่?

---

## format ของ task.md (สร้างเมื่อได้รับการยืนยันแล้ว)

```
## Feature: [ชื่อ feature]
## วันที่: [วันที่]
## เป้าหมาย: [อธิบาย 1 บรรทัด ภาษาคน]

### 🎨 Frontend
- [ ] ...

### ⚙️ Backend
- [ ] ...

### 🧪 Test
- [ ] ...

### 🚀 Deploy
- [ ] ...

### ข้อควรระวัง
- ...

### ไฟล์ที่ห้ามแตะ
- ...
```

---

## โปรเจกต์นี้

**ชื่อโปรเจกต์:** SMLLineCRM

**ทำอะไร:** ระบบติดตาม KPI พนักงาน Customer Support บน LINE Official Account (OA) แบบ real-time — วัดว่าพนักงานตอบลูกค้าเร็วแค่ไหน, ให้ AI สรุป conversation รายวัน, และทำ Daily Report ให้ผู้จัดการ

**Tech Stack:**
- Frontend: Next.js 14 (App Router), TypeScript, Tailwind CSS, shadcn/ui
- Backend: Node.js, TypeScript, Express.js 4, Mongoose 8
- Database: MongoDB
- Deploy: Docker + DigitalOcean (CI/CD ผ่าน GitHub Actions → ghcr.io)

**โครงสร้าง Monorepo:**
```
SMLLineCRM/
├── line-kpi-system/   Backend API (รับ webhook LINE, ประมวลผล, REST API)
├── line-kpi-admin/    Admin Dashboard (Next.js)
└── plans/             เอกสารโปรเจ็ค
```

**Agent แต่ละ folder:**
- เปิด Claude ที่ `line-kpi-system/` → Backend Developer (รู้ Express/Mongoose patterns ลึก)
- เปิด Claude ที่ `line-kpi-admin/` → Frontend Developer (รู้ Next.js/Sidebar patterns ลึก)

**convention สำคัญ:**
- ทุก request จาก browser ส่งผ่าน `/api/proxy/[...path]` (ซ่อน API key)
- Auth = API Key (ทุก route) + JWT (admin routes) — two-tier
- เพิ่มหน้าใหม่ต้องทำ 5 ขั้นตอน (ดู checklist ด้านล่าง) มิฉะนั้น redirect กลับ login
- ไม่มี automated tests ในโปรเจ็คนี้

**ไฟล์ที่ห้ามแตะเด็ดขาด:**
- `line-kpi-system/.env`
- `line-kpi-admin/.env.local`
- `line-kpi-system/dist/` (build output)
- `line-kpi-admin/.next/` (build output)

---

## Checklist — เมื่อเพิ่มหน้า/Route ใหม่ใน Admin Dashboard

**ทุกครั้งที่สร้างหน้าใหม่ใน `line-kpi-admin/(dashboard)/` ต้องทำสิ่งต่อไปนี้ด้วยเสมอ (ใส่ใน task.md ทุกครั้ง):**

1. **เพิ่ม PermissionKey** ใน backend — `line-kpi-system/src/api/middleware/jwtAuth.ts` (array `PERMISSION_KEYS`)
2. **เพิ่ม route mapping** ใน `line-kpi-admin/src/middleware.ts` — object `ROUTE_PERMISSIONS`
3. **เพิ่มเมนูใน Sidebar** — `line-kpi-admin/src/components/Sidebar.tsx` พร้อม permission check
4. **เพิ่ม route ใน PermissionGroup routes** — `line-kpi-system/src/api/routes/permissionGroupRoutes.ts`
5. **ตรวจสอบ UI จัดการสิทธิ์** — หน้า `/permission-groups` และ `/users` ต้องแสดง permission key ใหม่

---

## กฎความปลอดภัย — ห้ามส่งข้อมูลสำคัญขึ้น GitHub

**ห้าม commit หรือ push ไฟล์ที่มีข้อมูลต่อไปนี้โดยเด็ดขาด:**
- `.env`, `.env.local`, `.env.production` และทุก variant ของ env file
- API keys, tokens, passwords, secrets ทุกชนิด
- MongoDB URI ที่มี credentials จริง
- DigitalOcean API key หรือ credentials ของ server
- LINE Channel Secret / Access Token

**ถ้าพบว่ามีการเผลอ commit secrets:** แจ้งผู้ใช้ทันทีและแนะนำให้ rotate credentials นั้นก่อนทำอะไรต่อ

---

## คำสั่งที่ใช้บ่อย (ส่งให้ agent ที่เกี่ยวข้อง)

```bash
# Backend (port 3000)
cd line-kpi-system
npm run dev      # development — hot reload, http://localhost:3000
npm run build    # compile TypeScript → dist/
npm run start    # run production build
npm run lint

# Frontend (port 3001)
cd line-kpi-admin
npm run dev      # development — http://localhost:3001
npm run build
npm run start    # run production build
npm run lint
```

---

## Architecture ที่ต้องเข้าใจก่อนแก้โค้ด

### Request / Proxy Flow

Browser ไม่ได้คุย Backend โดยตรง — ทุก API call ผ่าน Next.js route handler:

```
Browser
  → POST /api/proxy/employees        (Next.js, line-kpi-admin)
  → [proxy injects X-API-Key + Authorization: Bearer <jwt>]
  → GET /api/v1/employees            (Express, line-kpi-system)
```

**ไฟล์:** `line-kpi-admin/src/app/api/proxy/[...path]/route.ts`
- อ่าน `auth-token` cookie → ใส่เป็น `Authorization: Bearer ...`
- อ่าน `API_KEY` env (server-side) → ใส่เป็น `X-API-Key`
- auth routes (`auth/*`) ไม่ต้องส่ง `X-API-Key`

### Two-Tier Auth (Backend)

| Tier | Header | Protects | ไฟล์ middleware |
|------|--------|----------|----------------|
| API Key | `X-API-Key` | ทุก management endpoint | `line-kpi-system/src/api/middleware/auth.ts` |
| JWT | `Authorization: Bearer` | `/admin/users`, `/admin/permission-groups` | `line-kpi-system/src/api/middleware/jwtAuth.ts` |

Frontend page-level guard: `line-kpi-admin/src/middleware.ts` — verify `auth-token` cookie + check `ROUTE_PERMISSIONS` map

### Permission System

- **PermissionKey** union type: `line-kpi-system/src/models/PermissionGroup.ts`
- JWT payload: `{ sub, username, displayName, isSuperAdmin, permissions[] }`
- `isSuperAdmin = true` → bypass ทุก permission check
- หลัง login ได้ 2 cookies: `auth-token` (httpOnly, ใช้โดย middleware) + `user-info` (client-readable, ใช้โดย Sidebar)

### Key Files Reference

| งาน | ไฟล์ที่ต้องแตะ |
|-----|--------------|
| เพิ่ม permission key | `line-kpi-system/src/models/PermissionGroup.ts` + `jwtAuth.ts` |
| เพิ่ม backend route | `line-kpi-system/src/api/routes/` + register ใน `router.ts` |
| เพิ่ม frontend page | `line-kpi-admin/src/app/(dashboard)/[route]/page.tsx` |
| เพิ่ม API client | `line-kpi-admin/src/lib/api.ts` |
| เปลี่ยน Sidebar menu | `line-kpi-admin/src/components/Sidebar.tsx` |
| เปลี่ยน page permission | `line-kpi-admin/src/middleware.ts` (`ROUTE_PERMISSIONS`) |
| เปลี่ยน AI config | `line-kpi-system/src/services/ai/` |
| เปลี่ยน scheduled jobs | `line-kpi-system/src/jobs/scheduler.ts` |

---
> Source: [smlsoft/SMLLineCRM](https://github.com/smlsoft/SMLLineCRM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
