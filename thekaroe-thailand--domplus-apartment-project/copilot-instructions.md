## domplus-apartment-project

> ไฟล์นี้ Claude Code อ่านอัตโนมัติทุกครั้งที่เริ่ม session ในโปรเจกต์นี้

# CLAUDE.md — DormPlus

ไฟล์นี้ Claude Code อ่านอัตโนมัติทุกครั้งที่เริ่ม session ในโปรเจกต์นี้

## โปรเจกต์

DormPlus — เว็บแอปจัดการหอพัก UI ภาษาไทย (ปี พ.ศ.)
Stack: Next.js (App Router) · TypeScript strict · Tailwind + shadcn/ui · Recharts · Prisma + PostgreSQL · Vitest · Playwright · pnpm
งานปัจจุบัน: Phase 1 — หน้า Dashboard ให้ตรงกับ `assets/reference-dashboard.png`

## เอกสารหลัก (import อัตโนมัติ)

@RULES.md
@SPEC.md
@DESIGN.md
@AGENT.md

## คำสั่ง

- `pnpm dev` — รัน local
- `pnpm lint && pnpm typecheck && pnpm test` — ต้องผ่านก่อนบอกว่างานเสร็จ
- `pnpm test:e2e` — Playwright
- `pnpm db:migrate` / `pnpm db:seed` — Prisma

## วิธีทำงานกับโปรเจกต์นี้

- ก่อนแก้ UI ให้เปิดดู `assets/reference-dashboard.png` ด้วย tool อ่านภาพทุกครั้ง อย่าทำจากความจำ
- งานที่มีหลายขั้นตอน ให้วางแผนเป็นรายการสั้น ๆ ก่อน แล้วทำทีละขั้นตาม AGENT.md "ลำดับการทำงานที่แนะนำ"
- เขียน test ของ `lib/format.ts` และ `lib/kpi.ts` ก่อนเขียนคอมโพเนนต์ที่ใช้มัน
- หลังแก้ UI ถ้ามี Playwright ให้ถ่าย screenshot ที่ 1536px และ 390px แล้วเทียบกับภาพอ้างอิง
- ทำตาม RULES.md §8 — หยุดถามก่อนเพิ่ม dependency, เปลี่ยน schema แบบลบข้อมูล หรือรันคำสั่งทำลายข้อมูล
- แจ้งผู้ใช้ทุกครั้งที่ตัดสินใจเรื่องที่ spec ไม่ได้ระบุ หรือเจอข้อขัดแย้งใน SPEC.md §11

## สิ่งที่มักพลาด

- บวก 543 เอง → ใช้ `Intl.DateTimeFormat("th-TH")`
- เก็บเงินเป็น float → เก็บเป็นสตางค์ (`Int`), API ส่งเป็นบาท
- ใส่ตัวเลขจากภาพลงในคอมโพเนนต์ → ตัวเลขต้องมาจาก seed/DB
- ลืมกรองด้วย `dormitoryId` ที่ผู้ใช้มีสิทธิ์
- สีรายจ่าย: ขึ้น = แดง (กลับทิศกับรายรับ)
- ใช้ hex ตรง ๆ → ใช้ token จาก DESIGN.md

---
> Source: [thekaroe-thailand/domplus-apartment-project](https://github.com/thekaroe-thailand/domplus-apartment-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
