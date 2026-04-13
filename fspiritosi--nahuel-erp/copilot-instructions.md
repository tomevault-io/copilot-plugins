## nahuel-erp

> Este archivo es la **guía principal** del proyecto. Las reglas detalladas están en `.claude/rules/`.

# CLAUDE.md - Guía del Proyecto NewProject

Este archivo es la **guía principal** del proyecto. Las reglas detalladas están en `.claude/rules/`.

---

## Tech Stack

```
Framework:        Next.js 16.1.3 + React 19 (App Router, Server Components)
Base de Datos:    PostgreSQL + Prisma ORM 7
Estado Global:    Zustand (global) + Jotai (atómico)
Estado Servidor:  React Query (TanStack Query v5)
UI:               shadcn/ui + Tailwind CSS v4 + Lucide Icons
Formularios:      React Hook Form + Zod (validación)
Autenticación:    Clerk
Storage:          MinIO (dev) / Cloudflare R2 (prod) - API S3
Fechas:           moment.js (NUNCA date-fns)
Notificaciones:   Sonner
```

### Versiones Clave

- Node.js: 18+ (recomendado 20+)
- Next.js: 16.1.3
- React: 19.2.3
- TypeScript: 5+
- Prisma: 7.2.0 (con adapter @prisma/adapter-pg)

---

## Comandos

```bash
# Desarrollo
npm run dev              # Servidor con Turbopack
npm run build            # Build producción
npm run start            # Servidor producción

# Calidad
npm run lint             # ESLint
npm run lint:fix         # Corregir ESLint
npm run format           # Prettier
npm run check-types      # TypeScript

# Base de Datos
npm run db:generate      # Generar Prisma
npm run db:push          # Push schema
npm run db:migrate       # Crear migración
npm run db:studio        # Prisma Studio

# Docker
docker-compose up -d db                    # Solo PostgreSQL
docker-compose --profile storage up -d     # PostgreSQL + MinIO

# Testing
npm run cy:open          # Cypress UI
npm run cy:run           # Cypress headless
```

---

## Arquitectura

```
┌─────────────────────────────────────────────────────────────────┐
│                         src/app/                                │
│   Páginas DELGADAS - Solo importan de modules/                  │
│   ├── (auth)/     → sign-in, sign-up                           │
│   ├── (core)/     → dashboard/* (protegido)                    │
│   └── api/        → storage                                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       src/modules/                              │
│   LÓGICA DE NEGOCIO - Server Actions, Componentes, Features    │
│   └── {module}/                                                 │
│       ├── features/     → list, detail, create, edit           │
│       ├── types.ts                                              │
│       └── index.ts                                              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        src/shared/                              │
│   CÓDIGO COMPARTIDO - Usado por todos los módulos              │
│   ├── components/   → ui/, layout/, common/                    │
│   ├── hooks/        → useGeography, use-mobile                 │
│   ├── actions/      → geography, storage, catalogs             │
│   ├── lib/          → prisma, logger, utils, company           │
│   └── utils/        → formatters, mappers                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## Reglas Críticas (Referencias)

Todas las reglas detalladas están en `.claude/rules/`. Se cargan automáticamente cada sesión.

| Regla | Archivo | Descripción |
|-------|---------|-------------|
| NO `:any` | `@.claude/rules/typescript-types.md` | Inferir tipos desde Prisma/Zod |
| Server Actions | `@.claude/rules/server-actions.md` | Ubicación por feature, nomenclatura |
| Logger | `@.claude/rules/logger.md` | Reemplazo automático de console.* |
| React Query | `@.claude/rules/react-query.md` | NUNCA useEffect+useState para fetch |
| Server Components | `@.claude/rules/server-components.md` | Server-first, prefijo `_` para client |
| Módulos | `@.claude/rules/module-structure.md` | Features con carpetas, NO archivos sueltos |
| Comunicación | `@.claude/rules/module-communication.md` | NO importar entre módulos, usar shared/ |
| Prisma | `@.claude/rules/prisma.md` | Queries optimizadas, select, getActiveCompanyId |
| UI/shadcn | `@.claude/rules/ui-shadcn.md` | Responsive, DataTable meta.title |
| Formularios | `@.claude/rules/forms.md` | React Hook Form + Zod |
| Storage | `@.claude/rules/storage.md` | MinIO/R2, presigned URLs |
| Nomenclatura | `@.claude/rules/nomenclature.md` | Archivos, imports, moment.js |
| Testing | `@.claude/rules/testing-cypress.md` | Cypress E2E |

---

## Reglas de Oro (Siempre Activas)

### 1. moment.js para Fechas

```typescript
// ✅ SIEMPRE
import moment from 'moment';
moment(date).format('DD/MM/YYYY');

// ❌ NUNCA
import { format } from 'date-fns';
```

### 2. Logger, NO console.*

```typescript
// ✅ SIEMPRE
import { logger } from '@/shared/lib/logger';
logger.info('mensaje');
logger.error('error', { data: { error } });

// ❌ NUNCA
console.log('mensaje');
```

### 3. useQuery para Fetching

```typescript
// ✅ SIEMPRE
const { data } = useQuery({
  queryKey: ['key'],
  queryFn: getData,
});

// ❌ NUNCA
const [data, setData] = useState([]);
useEffect(() => { getData().then(setData); }, []);
```

### 4. Client Components con Prefijo `_`

```typescript
// ✅ Server Component (por defecto)
EmployeesList.tsx

// ✅ Client Component (interactividad)
_EmployeesTable.tsx  // ← Nota el prefijo _
```

### 5. app/ = Solo Rutas, NO Components

```typescript
// ❌ NUNCA crear carpetas de components en app/
app/(auth)/invite/
├── page.tsx
└── components/        // ❌ PROHIBIDO
    └── _MyComponent.tsx

// ✅ SIEMPRE poner components en modules/ o shared/
app/(auth)/invite/
└── page.tsx           // Solo importa de modules/ o shared/

modules/auth/features/accept-invitation/
├── AcceptInvitation.tsx
├── actions.server.ts
├── components/
│   └── _AcceptForm.tsx  // ✅ Componentes van aquí
└── index.ts
```

**app/ solo puede contener:**
- `page.tsx` - Página de la ruta
- `layout.tsx` - Layout de la ruta
- `loading.tsx` - Estado de carga
- `error.tsx` - Manejo de errores
- `not-found.tsx` - 404
- Subcarpetas de rutas (ej: `[id]/`, `new/`)

---

## Estructura del Proyecto

```
newproject/
├── prisma/
│   └── schema.prisma
│
├── src/
│   ├── app/                       # Routing (páginas delgadas)
│   │   ├── (auth)/                # Login, Register
│   │   ├── (core)/dashboard/      # App principal (protegido)
│   │   └── api/                   # API Routes (storage)
│   │
│   ├── modules/                   # Dominios de negocio
│   │   ├── dashboard/
│   │   ├── employees/
│   │   ├── teams/
│   │   ├── documents/
│   │   ├── commercial/
│   │   ├── operations/
│   │   ├── maintenance/
│   │   └── company/
│   │
│   ├── shared/                    # Código compartido
│   │   ├── components/
│   │   │   ├── ui/                # shadcn components
│   │   │   ├── layout/            # Sidebar, Header
│   │   │   └── common/            # DataTable, etc.
│   │   ├── hooks/
│   │   ├── actions/
│   │   ├── lib/
│   │   ├── utils/
│   │   ├── types/
│   │   └── zodSchemas/
│   │
│   ├── providers/
│   └── generated/prisma/
│
├── cypress/                       # Tests E2E
└── CLAUDE.md
```

---

## Estructura de un Módulo

```
modules/{module-name}/
├── features/
│   ├── list/
│   │   ├── {Module}List.tsx       # Server Component
│   │   ├── actions.server.ts      # Actions de ESTA feature
│   │   ├── components/
│   │   │   └── _{Module}Table.tsx # Client Component
│   │   └── index.ts
│   ├── detail/
│   ├── create/
│   └── edit/
│
├── types.ts
├── hooks/
└── index.ts
```

---

## Checklist antes de Commit

- [ ] `npm run check-types` pasa
- [ ] `npm run lint` pasa
- [ ] No hay `console.*` (solo `logger`)
- [ ] No hay `:any` en tipos
- [ ] Server Components por defecto
- [ ] Client Components tienen `_` en nombre
- [ ] Módulos no importan de otros módulos
- [ ] Enums usan tipos de Prisma
- [ ] No hay mapeos inline (usar `shared/utils/mappers.ts`)
- [ ] Componentes < 200 líneas
- [ ] Features tienen su carpeta (no archivos sueltos)
- [ ] Queries usan `select` para campos necesarios
- [ ] Server actions usan `getActiveCompanyId()`
- [ ] Columnas de DataTable tienen `meta.title`
- [ ] Diseño responsive implementado

---

## Referencias Rápidas

### Archivos Clave

```
src/shared/lib/prisma.ts       # Cliente Prisma
src/shared/lib/logger.ts       # Logger personalizado
src/shared/lib/company.ts      # getActiveCompany(), getActiveCompanyId()
src/shared/lib/storage.ts      # S3 presigned URLs
src/shared/utils/formatters.ts # formatDate, calculateAge
src/shared/utils/mappers.ts    # genderLabels, statusBadges
```

### Imports Comunes

```typescript
import { prisma } from '@/shared/lib/prisma';
import { logger, Logger } from '@/shared/lib/logger';
import { getActiveCompanyId } from '@/shared/lib/company';
import { Button } from '@/shared/components/ui/button';
import { DataTable } from '@/shared/components/common/DataTable';
import { cn } from '@/shared/lib/utils';
import moment from 'moment';
```

---

**Las reglas detalladas están en `.claude/rules/`. Los skills de creación están en `.claude/skills/`.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fspiritosi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fspiritosi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
