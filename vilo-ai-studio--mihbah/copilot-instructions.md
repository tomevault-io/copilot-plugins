## mihbah

> Sistema Integral de Gestion multi-empresa para el Universo Jade. Migracion desde un prototipo de Lovable a un proyecto de produccion con stack autohospedado.

# MIHBAH — SIG Universo Jade

Sistema Integral de Gestion multi-empresa para el Universo Jade. Migracion desde un prototipo de Lovable a un proyecto de produccion con stack autohospedado.

## Empresas en alcance Fase 1

- MIHBAH — empresa de obras. Foco en avance de obra medido por gasto.
- BM CORP — empresa de ventas y comisiones. Datos vienen de Monday.com.
- YCDI (You Can Do It) — empresa de levantamiento de capital.

## Stack

- React 18 + TypeScript + Vite
- Tailwind + shadcn/ui (ya migrado de Lovable)
- Postgres 16 + pgvector (Docker local, despues Easypanel)
- Drizzle ORM (NO Prisma, NO Supabase)
- Lucia Auth + Arctic (NO Supabase Auth, NO NextAuth)
- TanStack Query para data fetching
- Zustand para estado global
- Anthropic SDK directo (NO Lovable AI Gateway)
- Voyage AI para embeddings del RAG semantico

## Convenciones de codigo

### Estricto

- TypeScript strict mode siempre activado
- No usar `any` jamas, usar `unknown` y narrow types
- No usar `console.log` en codigo de produccion, usar el logger de la app
- Validar todo input externo con Zod
- Toda mutacion en DB pasa por una funcion de servicio en `src/server/`

### Naming

- Archivos en kebab-case: `dashboard-mihbah.tsx`
- Componentes React en PascalCase: `DashboardMihbah`
- Hooks empiezan con `use`: `useEmpresaActiva`
- Funciones de servicio en camelCase: `getMovimientosByEmpresa`

### Estructura

- `src/components/` — componentes React puros (presentacionales)
- `src/pages/` — paginas con su lógica
- `src/db/` — schema y cliente de Drizzle
- `src/lib/` — utilidades sin estado
- `src/server/api/` — endpoints (handlers)
- `src/hooks/` — custom hooks
- `src/store/` — Zustand stores

## Reglas de la base de datos

- Usar siempre `uuid` como primary key
- Toda tabla con timestamps `createdAt` y `updatedAt`
- Foreign keys con `onDelete: "cascade"` cuando aplique
- Soft delete con campo `activo: boolean` en lugar de borrar
- Embeddings se guardan en columna `vector(1024)` para Voyage AI

## Reglas de auth

- Lucia maneja sesiones, NO JWT en cliente
- Sesiones se guardan en tabla `sessions`
- 2FA obligatorio para roles ADMIN y SUPER_ADMIN
- Recuperacion de contrasena con tokens de 1 hora

## Roles

- SUPER_ADMIN_DEV — todo + config LLM (solo desarrollo)
- SUPER_ADMIN — todo + gestion de usuarios
- ADMIN — cargar Excel, registrar movimientos
- VIEWER — solo lectura

## Modulos por empresa

| Modulo | Todas | MIHBAH | YCDI | BM CORP |
|---|---|---|---|---|
| Dashboard | Si | Si | Si | Si |
| Flujo de caja | Si | Si | Si | Si (semanal) |
| Proyectos | Si | Si | Si+ | Si (vendidos) |
| Cuentas | Si | Si | Si | Si (comisiones) |
| Reportes | Si | Si | Si | Si |
| Cargas Excel | No | Si | Si | No |
| Sincronizacion Monday | No | No | No | Si |

## Comandos importantes

- `npm run db:up` — levanta Postgres en Docker
- `npm run db:push` — aplica el schema a la DB sin generar migracion (solo dev)
- `npm run db:generate` — genera archivo SQL de migracion
- `npm run db:migrate` — aplica migraciones pendientes
- `npm run db:studio` — abre Drizzle Studio en el navegador
- `npm run db:seed` — corre el seed con datos de prueba
- `npm run dev` — arranca Vite en :5173

## Plan de migracion (orden)

1. Setup: Postgres en Docker, Drizzle, schema base
2. Auth con Lucia (login, registro, 2FA, recuperacion)
3. Layout base: sidebar dinamico, selector de empresa, panel IA siempre visible
4. Dashboards: 4 dashboards (Todas, MIHBAH, BM CORP, YCDI) con datos vacios
5. Modulos: Flujo, Proyectos, Cuentas, Reportes
6. Cargas Excel + Sincronizacion Monday
7. Asistente IA con tool use + RAG semantico
8. Admin: gestion de usuarios y roles

## Lo que NO debe hacer Claude Code

- NO instalar `@supabase/supabase-js` ni nada de Supabase
- NO usar `localStorage` para sesiones de auth (eso lo maneja Lucia con cookies)
- NO crear Edge Functions (estamos migrando a endpoints normales)
- NO escribir codigo SQL crudo, todo va por Drizzle
- NO crear migraciones manualmente, usar `drizzle-kit generate`
- NO hacer `console.log` en commits finales
- NO instalar paquetes sin justificarlos en el commit

---
> Source: [vilo-ai-studio/mihbah](https://github.com/vilo-ai-studio/mihbah) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
