## full-stack-adonisjs-s5-enrique-reano

> > Contexto base para copilotos de IA (Claude Code / Cursor) sobre este repo.

# CLAUDE.md — Memoria del proyecto

> Contexto base para copilotos de IA (Claude Code / Cursor) sobre este repo.
> En la Sesión 3 formalizamos esta memoria como `AGENTS.md` y la enlazamos con
> `ln -s AGENTS.md CLAUDE.md` para que sea agnóstica de la herramienta.

## Qué es este proyecto

Starter kit full-stack con autenticación por **access tokens**. Es el repo hilo
conductor del máster AI4Devs. Monorepo con dos apps independientes:

- `backend/` — API JSON en **AdonisJS 7** (TypeScript).
- `frontend/` — SPA en **React 19 + Vite**.

## Stack

| Capa | Tecnología |
|---|---|
| Backend framework | AdonisJS 7 (api starter kit) |
| ORM / DB | Lucid + SQLite (`better-sqlite3`) |
| Validación | VineJS |
| Auth | `@adonisjs/auth` · guard `api` (access tokens) |
| Frontend | React 19 + TypeScript + Vite |
| Routing FE | React Router |
| Estilos | Tailwind CSS v4 + shadcn/ui |
| SDD | OpenSpec (`/opsx:*`) |

## Comandos

### Backend (`cd backend`)
- `npm run dev` — servidor con HMR en `http://localhost:3333`
- `npm run migration:run` — aplica migraciones
- `npm run migration:fresh` — recrea la BD desde cero
- `npm run typecheck` — `tsc --noEmit`
- `node ace list:routes` — inspecciona el árbol de rutas

### Frontend (`cd frontend`)
- `npm run dev` — Vite dev server en `http://localhost:5173`
- `npm run build` — build de producción

## Convenciones del backend

- Lógica **en controllers**, no en el archivo de rutas (`start/routes.ts` solo cablea).
- Validación de input **siempre** con un validator de VineJS (`app/validators/`).
- Serialización de salida **siempre** vía `UserTransformer` (`app/transformers/`),
  nunca devolviendo el modelo crudo (el `password` se excluye, pero el contrato
  de salida vive en un único sitio).
- Rutas bajo prefijo `/api/v1`. Las protegidas usan `middleware.auth()`.
- Imports con subpath imports (`#controllers/*`, `#models/*`, …), no rutas relativas.

## OpenSpec

Flujo spec-driven con los slash commands `/opsx:propose`, `/opsx:apply`,
`/opsx:archive`. La configuración vive en `openspec/config.yaml`.

## No hacer

- No metas lógica de negocio en `start/routes.ts`.
- No devuelvas modelos Lucid directamente en las respuestas.
- No commitees `.env` ni `backend/tmp/db.sqlite3`.

---
> Source: [kiketon007/full-stack-adonisjs-s5-enrique-reano](https://github.com/kiketon007/full-stack-adonisjs-s5-enrique-reano) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
