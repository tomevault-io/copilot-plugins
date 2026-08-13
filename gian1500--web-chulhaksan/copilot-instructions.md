## web-chulhaksan

> - Repo layout: `backend/` (NestJS + Prisma + PostgreSQL), `frontend/` (React + Vite + TypeScript + Tailwind).

﻿# AGENTS.md

## Project snapshot
- Repo layout: `backend/` (NestJS + Prisma + PostgreSQL), `frontend/` (React + Vite + TypeScript + Tailwind).
- App domain: escuela/academia con roles `ADMIN`, `TEACHER`, `STUDENT`.
- Core flows: auth con JWT, asignacion de alumnos a profesor, cuotas mensuales, pagos Mercado Pago y pagos en efectivo.
- Extras: gimnasios + asistencias por gimnasio + etiquetas Adulto/Infantil + Formas (links) con desbloqueo manual.
- Deploy: Front en Vercel (`frontend/vercel.json` con rewrite SPA), back en Render.
- Paginacion: server-side en admin y profesor (page/limit/search); alumnos usan 10 items por pagina en UI.

## Quick commands
- Frontend dev: `cd frontend; npm install; npm run dev`
- Backend dev: `cd backend; npm install; npm run start:dev`
- Prisma: `cd backend; npx prisma generate`; `npx prisma migrate dev`
- Tests (backend): `cd backend; npm run test` / `npm run test:e2e`

## Runtime/config notes
- Backend carga `.env` desde `backend/.env` y/o raiz.
- Auth usa cookies httpOnly (`access_token`, `refresh_token`) y frontend envía `credentials: 'include'`.
- Modo testing (bloqueo por rol): `TESTING_MODE=true` y `TESTING_ALLOWED_ROLES=TEACHER,ADMIN`.
- Prisma: `backend/package.json` tiene `postinstall: prisma generate` (importante para deploys).
- MP OAuth: `MP_CLIENT_ID`, `MP_CLIENT_SECRET`, `MP_OAUTH_REDIRECT_URI`, `MP_OAUTH_STATE_SECRET`.
- MP Webhook: `MP_WEBHOOK_URL` (se usa con `teacherId` en query), `MP_WEBHOOK_SECRET`, `MP_WEBHOOK_TOLERANCE_SEC`.
- Reconciliacion MP (scheduler): `MP_RECONCILE_ENABLED` (default true), `MP_RECONCILE_BATCH_SIZE` (default 30), `MP_RECONCILE_LOOKBACK_HOURS` (default 720).
- MP Back URLs: `MP_BACK_URL_SUCCESS`, `MP_BACK_URL_FAILURE`, `MP_BACK_URL_PENDING`.
- MP Preferencias: `MP_ITEM_CATEGORY_ID`, `MP_ITEM_DESCRIPTION`, `MP_STATEMENT_DESCRIPTOR`.
- Registro público: `ALLOW_PUBLIC_REGISTER=true` habilita `/auth/register` en producción (por defecto se bloquea).
- JWT: `JWT_ACCESS_EXPIRES_IN` (default 15m), `JWT_REFRESH_SECRET`, `JWT_REFRESH_EXPIRES_IN` (default 7d).
- DB: `DATABASE_URL` (PostgreSQL).
- Admin bootstrap: no hay endpoint para crear ADMIN; se crea por SQL directo.

## Key business rules
- DNI debe ser numerico sin puntos; normalizar a solo digitos en backend.
- Alumnos solo ven nombre/apellido del profesor asignado.
- Profesores pueden crear alumnos y quedan `ACTIVE` y asignados al profesor.
- Cuotas se generan con `/fees/generate` (objetivo futuro: automatizar el dia 1) y vencen el dia 10.
- Recargo fijo de 5000 desde el dia 11 (no acumulativo).
- Alta de alumno (admin/profe): crea tambien la cuota del mes actual; si el alta es despues del dia 10, esa cuota usa `dueDate` fin de mes (sin recargo ese mes).
- Profesores solo pueden ver/alinear alumnos no asignados; no pueden reasignar alumnos de otros profesores.
- Profesores pueden marcar cuota como pagada en efectivo (genera `Payment` aprobado).
- Formas: solo `ADMIN` puede crear/editar/eliminar; `TEACHER` solo habilita/bloquea formas para sus alumnos asignados.
- Asistencia/solicitudes estan ocultas en frontend (endpoints quedan para futuro).
- Cambio de contraseña obligatorio: `User.mustChangePassword` + pantalla `/cambiar-contrasena`.
- Email y dirección son opcionales en altas/ediciones (admin/profesor).

## Data model hints (Prisma)
- `User` con `role` y `status`.
- `Gym` (carpetas/sedes).
- `Student` incluye `guardianPhone` y `gymId` obligatorio (relación a `Gym`).
- `Teacher` incluye tokens OAuth de Mercado Pago.
- `StudentTeacherAssignment` define asignacion activa.
- `MonthlyFee` y `Payment` manejan cuotas/pagos.
- `Payment.rawResponse` guarda MP raw; pagos en efectivo usan `{ manual: true, method: 'cash' }`.
- `Attendance` guarda 1 asistencia por alumno/dia y snapshot de `gymId` (presente/ausente + notas).

## Frontend touchpoints
- Auth helpers: `frontend/src/auth.ts`
- Dashboard: `frontend/src/Dashboard.tsx`
- Alumnos profesor: `frontend/src/TeacherStudents.tsx`
- Detalle alumno + cuotas: `frontend/src/StudentDetail.tsx`
- Pagos alumno: `frontend/src/Payments.tsx`, `frontend/src/PaymentQr.tsx`
- Comprobante: `frontend/src/PaymentReceiptSuccess.tsx` (descarga vía `window.print()`).
- Home: `frontend/src/Home.tsx`
- Paginación UI: `frontend/src/AdminStudents.tsx`, `frontend/src/AdminTeachers.tsx`, `frontend/src/TeacherStudents.tsx`
- Formas: `frontend/src/FormsManager.tsx` (solo admin), `frontend/src/MyForms.tsx` (alumno), desbloqueo por alumno en `frontend/src/StudentDetail.tsx` (admin+teacher)
- `/profesor/formas`: redirige a `/dashboard` (profesor sin pantalla de gestion de formas)
- Pagos alumno: polling cada 15s en `frontend/src/Payments.tsx` cuando hay cuotas pendientes y la pestana esta visible
- Asistencia por gimnasio (admin+teacher): `frontend/src/GymAttendance.tsx`
- Asistencia alumno (read-only): `frontend/src/MyAttendance.tsx`

## Backend touchpoints
- Auth: `backend/src/auth/*`
- Auth endpoints: `/auth/login` (set cookies), `/auth/refresh`, `/auth/logout`.
- Profesores: `backend/src/teachers/*`
- Alumnos: `backend/src/students/*`
- Asignaciones: `backend/src/assignments/*`
- Cuotas: `backend/src/fees/*`
- Pagos: `backend/src/payments/*`
- Prisma schema: `backend/prisma/schema.prisma`
- Paginación API: `backend/src/admin/admin.service.ts`, `backend/src/teachers/teachers.service.ts`
- Formas API: `backend/src/forms/*` (`/forms`, `/forms/me`, `/forms/student/:dni`, `/forms/student/:dni/access`)
- Formas permisos: `/forms` CRUD solo `ADMIN`; `/forms/student/:dni` y `/forms/student/:dni/access` para `ADMIN` + `TEACHER` (teacher solo sobre alumnos asignados)
- Gyms API: `GET /gyms`, `POST /admin/gyms`, `PATCH /admin/gyms/:id`, `PATCH /teachers/me/students/:dni/gym`
- Attendance API: `backend/src/attendance/*`
- Pagos reconciliacion: cron en `backend/src/payments/payments.service.ts` para conciliar `PENDING` contra Mercado Pago

## Attendance API (nuevo)
- `GET /attendance/gym/:gymId?date=YYYY-MM-DD` (ADMIN/TEACHER) lista alumnos del gimnasio + estado del dia
- `PUT /attendance/gym/:gymId?date=YYYY-MM-DD` (ADMIN/TEACHER) guarda asistencia bulk (ausente explicito para no marcados)
- `GET /attendance/me` (STUDENT) historial read-only (ultimos 120)

## Agent guidelines (project-specific)
- Mantener validaciones de DNI numerico en frontend y backend.
- No exponer datos del profesor a alumnos salvo nombre/apellido.
- Evitar cambios en endpoints de asistencia/solicitudes hasta nuevo requerimiento.
- Si agregas campos a Prisma, actualizar DTOs, servicios y UI.
- Respetar estilo UI actual (Tailwind, componentes mobile-first).
- En pagos MP, `external_reference` es `payment.id` y `metadata.paymentId` se usa en webhook.
- Scripts SQL de importación: `scripts/import-alumnos-profesor.sql`, `scripts/import-alumnos-que-son-profes.sql`, `scripts/import-profesores.sql`.
- Documentación principal: `docs/PROJECT.md`.

---
> Source: [Gian1500/web-chulhaksan](https://github.com/Gian1500/web-chulhaksan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
