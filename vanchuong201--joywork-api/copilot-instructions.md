## joywork-api

> This repo is a Fastify + Prisma backend organized by feature modules.

# JoyWork API Agent Guide

This repo is a Fastify + Prisma backend organized by feature modules.

Follow `.cursor/rules/*.mdc` first. Use this file as the short default guide:

- Match existing backend formatting: `single quotes`, semicolons, and `@/` alias imports inside `src`.
- Keep module structure consistent with `controller`, `service`, `routes`, and `schema` files under `src/modules/*`.
- Keep backend filenames aligned with the current style such as `jobs.routes.ts`, `auth.controller.ts`, or `user-profile.service.ts`.
- Return success responses in the current envelope shape `{ data: ... }` and error responses through `AppError` plus the shared error handler.
- Validate external input before use, keep Zod parsing and Swagger route schemas accurate, and enforce auth with the existing middleware pattern.
- Prefer named exports for new backend files, especially routes; do not add test or demo routes to production modules.
- Do not leave `console.*`, localhost instrumentation, or temporary debug hooks in `src/**`; use Fastify logging patterns when logging is necessary.
- Reuse shared helpers in `src/shared` and `src/config` instead of duplicating Prisma, auth, storage, or email clients.
- Make database changes through Prisma schema plus migrations, never by manual production schema edits.
- Before finishing meaningful backend work, run the most relevant checks available, typically `npm run lint`, `npm run type-check`, and targeted tests when they exist.

## Admin / vận hành (`/api/system`)

- Toàn bộ endpoint vận hành nền tảng (overview, users, companies, reports, xác minh công ty, khóa tài khoản) nằm trong module **`src/modules/system`**, prefix **`/api/system`**, bảo vệ bằng `verifyToken` + `requireAdmin`.
- `User.accountStatus`: `ACTIVE` | `SUSPENDED` — đăng nhập và mọi request sau JWT gọi `assertUserActive` (trừ khi chỉ verify token mà không cần; optionalAuth cũng bỏ qua user nếu assert lỗi).
- **joywork-admin** (repo riêng) dùng JWT ADMIN qua BFF; CORS tới admin origin đã có `ADMIN_CP_ORIGIN` / tương đương khi cần.

---
> Source: [vanchuong201/joywork-api](https://github.com/vanchuong201/joywork-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
