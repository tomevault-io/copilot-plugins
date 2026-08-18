## scout-deportivo

> Plataforma de scouting de futbolistas venezolanos en Argentina. Los scouts cargan

# Futvenar — Sistema de Scouting

Plataforma de scouting de futbolistas venezolanos en Argentina. Los scouts cargan
informes técnicos sobre jugadores; los jugadores gestionan su ficha y ven su
progreso; los agentes/consultores acceden al banco de talentos; los admin aprueban
solicitudes de acceso y gestionan pagos, campamentos y roadmaps.

**La app está íntegramente en español.** Código, rutas, nombres de tablas, columnas,
variables, comentarios y UI: todo en español. Mantené esa convención sin excepción.

## Comandos

```bash
npm run dev      # vite dev
npm run build    # vite build (Cloudflare Workers via nitro)
npm run lint     # eslint
npm run format   # prettier --write
```

No hay tests en el proyecto. El chequeo de tipos no está en un script — corré
`npx tsc --noEmit` a mano; es la red de seguridad principal antes de commitear.

## Stack

- **TanStack Start + Router** con file-based routing (`src/routes/`)
- **React 19**, TypeScript estricto
- **Supabase** — auth, Postgres con RLS, Storage, Edge Functions (Deno)
- **Tailwind v4** (config en CSS, no en JS) + shadcn/ui en `src/components/ui/`
- **Cloudflare Workers** para deploy (`wrangler.jsonc`)
- `@/` → `src/`

## Routing

Rutas planas separadas por punto. `_authenticated` es el layout con guard de sesión;
`app` agrupa la aplicación interna.

```
_authenticated.app.scout.informe.$jugadorId.tsx  →  /app/scout/informe/:jugadorId
```

`src/routeTree.gen.ts` es **generado** — nunca lo edites a mano; se regenera al
correr el dev server o el build.

Patrón estándar de una ruta:

```tsx
export const Route = createFileRoute("/_authenticated/app/scout/dashboard")({
  head: () => ({ meta: [{ title: "Mi dashboard · Scout" }] }),
  component: ScoutDashboard,
});
```

El guard vive en [`_authenticated.tsx`](src/routes/_authenticated.tsx): usa
`ssr: false` + `beforeLoad` que verifica la sesión y redirige a `/login`. Los
usuarios con `estado !== "activo"` ven una pantalla de bloqueo en vez de la app.

## Auth, roles y planes

`useAuth()` de [`src/lib/auth.tsx`](src/lib/auth.tsx) expone `session`, `user`,
`perfil`, `loading`. **Siempre esperá a que `loading` sea `false`** antes de
decidir qué renderizar — hubo bugs de "flash" por no hacerlo.

En el `AuthProvider`, el listener `onAuthStateChange` se monta **antes** de
`getSession()`, y las consultas a Supabase dentro del callback van diferidas con
`setTimeout(…, 0)`. Es intencional: invertir el orden o consultar sincrónicamente
dentro del callback rompe la sesión. No lo reordenes.

**Roles:** `admin` · `scout` · `jugador` · `consultor` · `tutor`
**Planes:** `libre` · `radar` · `elite` · `seleccion` (los consultores no tienen plan
pago por ahora — solo requieren aprobación de admin para registrarse)

La navegación y el gating de funcionalidad se derivan de `rol` + `plan_actual`
(ver `NavHeader` en `_authenticated.tsx`). El gating de UI es cosmético: **la
autorización real vive en las políticas RLS de Postgres**, y ahí es donde hay que
aplicarla.

## Base de datos

Cliente: `import { supabase } from "@/integrations/supabase/client";`

Dos archivos **generados** que nunca se editan a mano:
- `src/integrations/supabase/client.ts`
- `src/integrations/supabase/types.ts`

Al agregar o cambiar columnas, el flujo es:

1. Escribir una migración nueva en `supabase/migrations/` (formato
   `AAAAMMDD######_nombre_descriptivo.sql`)
2. Aplicarla a la BD
3. Regenerar los tipos:
   ```bash
   npx supabase gen types typescript --project-id <project-id> > src/integrations/supabase/types.ts
   ```

Saltarse el paso 3 produce una cascada de errores de tipo que parecen bugs de
código pero no lo son. Si ves errores del tipo `Argument of type '"columna"' is
not assignable to parameter of type …`, los tipos están desactualizados.

`schema_completo.sql` en la raíz es un dump de referencia del esquema completo —
útil para consultar, pero **no es la fuente de verdad**; las migraciones sí.

### RLS

Todas las tablas tienen RLS. Las políticas usan dos helpers de Postgres:
`public.mi_rol()` y `public.mi_estado()`. Al agregar una tabla, agregá también sus
políticas en la misma migración.

### Edge Functions

En `supabase/functions/` (Deno). Actualmente `send-invitacion` y
`aceptar-invitacion`. Los emails salen por `supabaseAdmin.auth.admin.inviteUserByEmail`
(se migró desde Resend). Se despliegan por separado del frontend.

## Diseño

El sistema de diseño está documentado en el header de
[`src/styles.css`](src/styles.css). Identidad de la Federación Venezolana de Fútbol:
vinotinto primario, dorado de acento.

**Usá siempre tokens semánticos** (`bg-primary`, `text-fvf-gold`,
`text-muted-foreground`). Nunca `text-white`, `bg-black` ni colores hardcodeados
sueltos en componentes. Tailwind v4 define el tema en CSS con `@theme inline` —
no hay `tailwind.config.js`.

Tipografía: Montserrat (`font-sans` y `font-display`).

## Convenciones de código

- Componentes y estado en español: `cargarPerfil`, `guardando`, `jugadores`
- Data fetching: la mayoría de las rutas usan `useEffect` + `useState` con llamadas
  directas a Supabase; algunas usan TanStack Query. Seguí el patrón del archivo que
  estés tocando en vez de unificar por tu cuenta.
- Consultas paralelas con `Promise.all` y `{ count: "exact", head: true }` para conteos
- `toast` de `sonner` para feedback al usuario
- Iconos de `lucide-react`
- Constantes de dominio (posiciones, criterios de evaluación, países, planes) viven
  en `src/lib/*-constants.ts`, no inline en los componentes

### Criterios de evaluación

Los criterios del informe técnico están en
[`src/lib/jugador-constants.ts`](src/lib/jugador-constants.ts), agrupados en
`CRITERIOS_TECNICOS`, `CRITERIOS_TACTICOS`, `CRITERIOS_ACTITUDINALES` y unificados
en `TODOS_CRITERIOS`. Cada `key` corresponde a una columna `int` (1-10) en la tabla
`informes`, y `puntaje_global` es una columna **generada** que promedia las no nulas.

Cambiar un `key` de criterio implica una migración de columnas. Cambiar los valores
de `RECOMENDACIONES` implica migrar los datos ya guardados.

## Estructura

```
src/
  routes/                    rutas (file-based, planas con puntos)
  components/
    ui/                      shadcn — no editar salvo necesidad real
    admin|busqueda|comparador|jugador-nuevo|notificaciones|seguimiento/
  lib/                       auth, constantes de dominio, lógica compartida
  integrations/supabase/     client.ts y types.ts (ambos generados)
  styles.css                 sistema de diseño completo
supabase/
  migrations/                fuente de verdad del esquema
  functions/                 edge functions (Deno)
```

---
> Source: [futvenar-source/scout-deportivo](https://github.com/futvenar-source/scout-deportivo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
