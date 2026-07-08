## sistema-activa-t

> Aplicación interna de FONDOEMPLEO para gestión y monitoreo de proyectos

# Sistema Activa-T — FONDOEMPLEO

Aplicación interna de FONDOEMPLEO para gestión y monitoreo de proyectos
de inserción laboral, mejora de empleabilidad y aumento de ingresos.

## Stack

- **Next.js 16** (App Router, Server Components, Server Actions)
- **React 19**
- **Supabase** (Postgres + Auth + Storage)
- **Tailwind CSS 3**
- **Recharts** para gráficos, **Leaflet** para mapa de Perú
- **TypeScript 5**

## Cómo correrlo

```bash
npm install
npm run dev   # http://localhost:3000
```

Requiere un archivo `.env.local` (NO trackeado) con:

```
NEXT_PUBLIC_SUPABASE_URL=...
NEXT_PUBLIC_SUPABASE_ANON_KEY=...
SUPABASE_SERVICE_ROLE_KEY=...
```

## Estructura

```
src/
  app/
    auth/                       # login, signout
    dashboard/                  # módulo principal protegido por middleware
      page.tsx                  # Proyectos: KPIs + mapa + gráficos
      actions.ts                # server actions del dashboard de proyectos
      servicios/                # módulo Servicios
      campo/                    # módulo Supervisión (monitoreo en campo)
      evaluacion/               # config de PDFs de bases/formato para evaluación
      gestion-proyectos/        # bandeja administrativa de proyectos
      gestion-servicios/        # bandeja administrativa de servicios
      gestion-aportantes/       # gestión de aportantes corporativos
      gestion-monitores/        # gestión de monitores de campo
      inf-gerencial/            # informe gerencial
      (corporativo)/documentos/ # documentos corporativos
    presentation/               # vista pública de presentación
  components/                   # componentes compartidos (charts, modals, tablas)
  config/
    permissions.ts              # matriz de permisos por email
  utils/supabase/               # clientes de Supabase (client.ts, server.ts)
  modules/
    gestion-proyectos/campo-test/  # vista de campo (bandeja monitores)
middleware.ts                   # gate de rutas según permissions.ts
scripts/                        # scripts one-off de migración e importación
```

## Autorización

- Acceso por correo electrónico hardcodeado en `src/config/permissions.ts`.
- `middleware.ts` filtra rutas `/dashboard/*` según el módulo permitido.
- Super admin: `jduran@fondoempleo.com.pe` (acceso total).
- **Importante:** los server actions usan `SUPABASE_SERVICE_ROLE_KEY` (bypasea RLS).
  No revalidan permisos por acción — el gate es solo el middleware. Es una deuda
  conocida; ver TODO abajo.

## Convenciones

- Server Actions con `"use server"` viven en `actions.ts` de cada módulo.
- Componentes con estado de cliente usan `"use client"`.
- Tablas Supabase en snake_case y español: `proyectos`, `instituciones_ejecutoras`,
  `etapas`, `lineas`, `ejes`, `especialistas`, `avance_proyecto`, etc.
- Columnas en español (incluyendo `año` con ñ — sí, funciona).
- `dynamic = 'force-dynamic'` en páginas que dependen de filtros frescos.

## Deuda técnica conocida

- [ ] Permisos hardcodeados → migrar a tabla `usuarios_modulos`.
- [ ] Server actions no validan usuario individualmente (defensa en profundidad).
- [ ] RLS deshabilitado (ver `scripts/disable_rls.sql`).
- [ ] Múltiples versiones de schema (`database_schema_v{1..4}.sql`) sin sistema de
      migraciones. Migrar a `supabase/migrations/`.
- [ ] Raíz del repo con ~100 scripts one-off (import/check/verify) — mover a
      `scripts/oneoff/` o ignorar.
- [ ] ~200 usos de `any` en `src/`. El archivo `supabase_types.ts` existe pero
      no está conectado a los clientes — conectarlo elimina la mayoría.
- [ ] Catálogos del dashboard (líneas, ejes, regiones, etapas) se recargan en
      cada navegación. Usar `unstable_cache` con revalidación bajo demanda.
- [ ] Sin tests ni CI.

## Módulo de Evaluación (estado actual)

`/dashboard/evaluacion/configuracion` permite subir PDFs de "bases" y "formato"
a Supabase Storage (bucket `documentos_evaluacion`). **Hoy es solo gestión de
archivos** — no hay lógica de evaluación automática de proyectos con IA en este
repo. La skill `fondoempleo-evaluacion` que ejecuta esa lógica vive en el
workspace de Claude (`~/.claude/skills/`), no en este código.

---
> Source: [Jaimelll/sistema-activa-t](https://github.com/Jaimelll/sistema-activa-t) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
