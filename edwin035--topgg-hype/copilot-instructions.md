## topgg-hype

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> Idioma: comentarios, mensajes de error y texto de UI en español (ver CLAUDE.md global).

## Qué es esto

SPA de e-commerce (**TopGG / TopLevel Shop**): venta de pines/gift cards de gaming.
Stack: **React 18 + Vite + TypeScript + TailwindCSS + shadcn/ui**, data fetching con
`@tanstack/react-query` + `axios`/`fetch`, formularios con `react-hook-form` + `zod`.

Es solo el **frontend**. Consume el backend NestJS del repo hermano
`../hype-integration-2026` (working directory adicional), desplegado en
`https://hypes.up.railway.app/api`. Ver [Relación con el backend](#relación-con-el-backend).

## Comandos

```bash
npm run dev        # dev server en http://localhost:5173  (el script fuerza --port 5173)
npm run build      # build de producción (Vite + SWC; NO hace typecheck)
npm run build:dev  # build en modo development
npm run preview    # previsualiza el build
npm run lint       # ESLint (flat config, eslint.config.js)
npm test           # vitest run (una pasada)
npm run test:watch # vitest en watch
```

- **No hay script de typecheck** y `build` no verifica tipos (usa SWC). Para chequear tipos:
  `npx tsc --noEmit`.
- **Un solo test:** `npx vitest run src/ruta/al/archivo.test.ts` o por nombre
  `npx vitest run -t "nombre del test"`.
- Tests: patrón `src/**/*.{test,spec}.{ts,tsx}`, entorno `jsdom`, setup en
  `src/test/setup.ts`.
- Alias de imports: **`@` → `src/`** (definido en `vite.config.ts`, `vitest.config.ts`,
  `tsconfig.*.json` y `components.json`).

## Arquitectura

### Capa de red (dos módulos, un solo cliente HTTP)

Todo pasa por **`src/lib/api/http.ts`** — es el único cliente fetch:
- `apiRequest<T>()` con aliases `http()` y `authClient()` (mismo cuerpo, nombres legacy).
- Base URL: `import.meta.env.VITE_API_URL` (fallback `https://hypes.up.railway.app/api`).
- Token Bearer leído de `localStorage["toplevel_access_token"]` (`STORE_TOKEN_KEY`).
- Errores normalizados en la clase `ApiError` (status + data + mensaje en español).

Sobre ese cliente hay **dos familias de endpoints** que apuntan al mismo backend pero a
dominios distintos:

- **`src/lib/api/*`** → dominio propio del backend: `auth` (login / me / changePassword),
  `users`, `sales`. Barril: `src/lib/api/index.ts`.
- **`src/lib/providers/*`** → passthrough a las rutas del proveedor **Pin Hype**
  (`/pin-hype/...`): `catalog` (collections/products/stock), `redeem` (pre-redeem),
  `reversal`. Barril: `src/lib/providers/index.ts`. Inyecta `PIN_HYPE_DEFAULTS`
  (country `CO`, currency **`USD`**, language `es`) definidos en `http.ts`.

Los hooks de negocio viven en `src/hooks/providers/*` (`useCatalogSections`, `useBuyPin`,
`useStock`, `useReversal`, ...) y envuelven esas funciones de endpoints.

### Autenticación (OJO: dos contextos distintos, no confundir)

- **`src/components/Auth/AuthProvider.tsx`** → estado de **sesión/usuario**. Hidrata desde
  el token en localStorage: `getMe()` y luego `getUserById()`. Expone `useAuth()`
  (`user`, `token`, `initializing`, `setToken`, `logout`, `refresh`). Escucha el evento
  `storage` para sincronizar entre pestañas.
- **`src/contexts/AuthContext.tsx`** → estado del **modal de login** (abierto/cerrado y
  vista: login/register/recover/verify/reset). Expone `useAuthDialog()`. Emite el evento
  `app:close-modals` al abrir.
- **`src/RequireAuth.tsx`** protege rutas: si no hay `user`, dispara `openLogin()` y
  redirige a `/`.

Orden de providers en `src/App.tsx`:
`QueryClientProvider > TooltipProvider > AuthProvider > AuthDialogProvider`.

### Rutas (`src/App.tsx`)

- **Públicas:** `/` (Index), `/catalogo`, `/aliados`.
- **Protegidas (`RequireAuth`):** `/producto/:id`, `/checkout/:id`, `/factura/:id`,
  `/perfil`, `/cambiar-contrasena`, `/historial`, `/pago/binance/success`.

### Flujo de compra (pago con Binance Pay)

**Regla de oro:** el pin de Hype (`pre-redeem`) NO se canjea hasta que el pago está
verificado por webhook. Antes de pagar, la venta vive `PENDIENTE` sin pines.

**Dos orígenes de venta (`Sale.origen`):** `HYPE` (id de producto < 1.000.000 → se entrega con
`pre-redeem` contra el proveedor) y `PROPIO` (id >= 1.000.000 → se entrega asignando un
**código del inventario local**, sin tocar Hype). El backend ramifica por origen en
`purchase()` (precio de la BD, moneda USD, stock = códigos libres) y en `fulfill()` (asigna N
códigos con lock por producto — `OwnFulfillmentService`, a prueba de doble venta). Ver la
sección de productos propios más abajo.

1. `CheckoutPage` → `checkout()` (`src/lib/api/checkout.ts` → `POST /checkout`). El backend
   valida el precio real contra el catálogo de Hype (el cliente **no** envía precios),
   **pre-chequea stock**, convierte la moneda de venta a USDT (USD va **1:1**, sin tasa
   manual; COP usaría `BINANCE_USDT_COP_RATE`, camino legacy), crea la venta
   `PENDIENTE` y una **orden hosted-checkout de Binance**. Devuelve
   `{ sale, checkoutUrl, amountUsdt, usdtCopRate, ... }`. **No canja.**
2. El front guarda `tg_pending_binance_sale` en localStorage, muestra la aclaración de
   cobro en USDT y redirige a `checkoutUrl` (Binance).
3. Binance paga → **webhook** (`POST /api/binancepay/webhook`, firma RSA-SHA256 verificada)
   → marca `paidAt` + `EN_PROCESO` → `fulfill()`: claim atómico, `pre-redeem` por unidad
   **all-or-nothing con reversa**, adjunta pines, `COMPLETADA`.
4. Binance devuelve al usuario a `/pago/binance/success` (`BinancePaySuccess.tsx`), que hace
   **polling** de `GET /sales/me/:id` hasta `COMPLETADA` → factura (`/factura/:id`).

**Estados de venta** (`SaleStatus`): `PENDIENTE` → `EN_PROCESO` (pagada, entregando) →
`COMPLETADA`. Fallos: `CANCELADA` (no pagó/expiró) y **`REQUIERE_ATENCION`** (pagó pero no
se pudo entregar tras ~15 reintentos → **resolución manual, NO hay refund automático**; el
cliente se contacta a mano).

**Resiliencia (backend):**
- **Claim atómico** (`fulfillmentStartedAt`) evita doble canje entre webhook y job.
- **Job cada minuto** (`CheckoutFulfillmentJob`): reintenta entregas `EN_PROCESO` pagadas
  sin pines (contador `fulfillmentAttempts`; al máximo → `REQUIERE_ATENCION`) y **expira**
  las `PENDIENTE` cuya orden de Binance venció (→ `CANCELADA` + `closeOrder`).
- **Admin:** `GET /sales/admin/attention` lista las ventas `REQUIERE_ATENCION` (con datos del
  cliente y motivo). `GET /sales` y ese endpoint exigen rol **ADMIN** (`AdminGuard`).

- **OJO:** el flujo viejo de 2 pasos (`useBuyPin` + `createSale` a `/sales`) se eliminó,
  igual que el `POST /sales` público. Los hooks `useBuyPin`/`useReversal` y `createSale` ya
  no existen. Único método de pago: `BINANCE`.
- **Deploy:** requiere migración Prisma, env de Binance (`BINANCEPAY_*`), tasa
  `BINANCE_USDT_COP_RATE`, `BINANCEPAY_RETURN_URL` → `.../pago/binance/success`, y registrar
  el webhook en el panel de Binance. Vars opcionales: `BINANCE_FULFILL_STALE_MS`,
  `BINANCE_PENDING_EXPIRY_MS`.

### Panel de administración (dashboard del gestor)

**Vive en su propio repo/app: `../topgg-admin`** (Vite + React + shadcn, mismo tema).
Se separó de este frontend por decisión de negocio: deploy y acceso independientes de la
tienda. Tiene login propio (solo email+contraseña), guard `RequireAdmin` y rutas `/`
(Overview), `/ventas` y `/atencion`. **En este repo NO hay código del panel** — no
reintroducir rutas `/admin` aquí.

Backend (repo `../hype-integration-2026`): módulo **`src/admin/`** (`AdminModule`), todo
bajo `@UseGuards(JwtAuthGuard, AdminGuard)`:
- `GET /api/admin/stats?range=today|7d|30d|all` → KPIs agregados (por estado, ingresos
  COP+USDT de `COMPLETADA`, ticket, conversión, cola `REQUIERE_ATENCION`) + serie diaria
  (vía `$queryRaw` con `date_trunc`).
- `GET /api/admin/sales?status=&from=&to=&q=&page=&pageSize=` → listado paginado y
  filtrable (reemplaza el uso del `findAll` sin paginar). Decisión de negocio: el gestor ve
  datos **completos** (pines y contacto del cliente).
- **Acciones (fase 2, escritura):** `POST /admin/sales/:id/resolve` (body `{outcome, note}`
  sobre `REQUIERE_ATENCION` → COMPLETADA/CANCELADA), `POST /admin/sales/:id/retry-fulfill`
  (reabre y llama `CheckoutService.fulfill`), `POST /admin/sales/:id/cancel`. Auditoría en
  `Sale`: `resolvedByUserId`, `resolvedAt`, `adminNote`. `AdminModule` importa
  `CheckoutModule` (que ahora **exporta** `CheckoutService`).

UI del panel (en `topgg-admin`): Overview con KPIs + gráfico recharts, tabla de ventas
paginada/filtrable con detalle, cola de atención con acciones (Reintentar / Resolver /
Cancelar via `SaleActions`).

**Seed de desarrollo (backend):** `npm run db:seed` (o `npx prisma db seed`) corre
`prisma/seed.ts` — idempotente: upsert de un admin + ventas de ejemplo (COMPLETADA,
REQUIERE_ATENCION, PENDIENTE) para probar el panel. Credenciales por env
`SEED_ADMIN_EMAIL`/`SEED_ADMIN_PASSWORD` (defaults de dev `admin@topgg.local`/`admin1234`;
sin secretos en el repo). Wireado en `prisma.config.ts` (`migrations.seed`). **Nunca contra
producción.**

## Seguridad (modelo de accesos — no reintroducir huecos)

Backend con `JwtAuthGuard` **global** (`APP_GUARD`); lo público lleva `@Public()`. Reglas:
- **Público = solo tienda:** catálogo, registro (`POST /users`, que **siempre** fuerza rol
  `USER`), login y webhook de Binance. Nada más.
- **No hay canje de pin por HTTP:** `POST /pin-hype/pre-redeem` se eliminó. El pin se genera
  server-side solo tras pago confirmado por webhook. No reintroducir ese endpoint.
- **Endpoints sensibles de Hype** (`reversal`, `report`, `pin-hype/auth`, `echo`) y todo
  `/admin/*`, `GET/DELETE /users`, `GET /sales` → **solo rol ADMIN**.
- `GET/PATCH /users/:id` → dueño o admin; un cliente no puede cambiar su `role`/`isActive`.
- **Primer admin:** `npm run create:admin` en el backend (nunca por HTTP).

## Gotchas (importante)

- **La tienda opera en USD.** `PIN_HYPE_DEFAULTS.currency` = `USD` y el backend pide USD a
  Hype. USDT va 1:1 con el dólar, así que **no hay tasa manual** en el camino normal.
- **Fail-closed de moneda (no lo debilites):** solo se muestran/venden productos cuyo precio
  venga etiquetado **exactamente** en la moneda pedida.
  - Front: `useCatalogSections` filtra por `salesCurrencyCode === PIN_HYPE_DEFAULTS.currency`.
  - Backend: `purchase()` exige `salesCurrencyCode ===` la moneda pedida y `usdtRateFor()`
    revienta si no conoce la tasa. Antes de cobrar en la moneda equivocada, no se cobra.
- **Catálogo USD ≠ catálogo COP (verificado contra el API real):** en USD Hype solo publica
  los **6 productos de Free Fire**; las **7 gift cards de PlayStation existen solo en COP** y
  por eso NO aparecen en la tienda. No es un bug: es el fail-closed haciendo su trabajo.
- **Quirk de Hype — moneda:** `/catalog/products/:id` IGNORA la moneda pedida y devuelve el
  precio en USD sin etiquetar. Los endpoints de catálogo/colecciones sí respetan `currency`
  e incluyen `salesCurrencyCode`. Para PRECIOS usar siempre el árbol: front
  `getProductForDisplay`/`findProductInCatalog` (`lib/providers/endpoints/catalog.ts`);
  backend `CatalogService.findProductInCatalog`.
- **Quirk de Hype — stock:** `{hasStock: true, amount: -1}` significa stock
  ILIMITADO/desconocido. Un amount negativo NO es "insuficiente".
- **Binance geo-bloqueado en dev:** el API de Binance Pay responde **451 (restricted
  location)** desde esta red local — cualquier llamada (certificados, crear orden) falla.
  En Railway/producción sí funciona. O sea: el flujo de pago completo solo se puede probar
  desplegado; en local se prueba hasta "crear orden falla → venta CANCELADA/CREATE_FAILED".

- **Fetch de catálogo:** el hook vivo es `useCatalogSections` (`src/hooks/providers/`), que
  lee el **árbol** (`getCatalogTree` → `GET /catalog`) y lo aplana: cada colección con
  productos propios es una sección, a cualquier profundidad. No hay un `useCatalog` genérico
  (existía uno roto y se eliminó).
  **OJO:** `getCatalogTree` apunta a **`/catalog`** (backend, `StorefrontModule`), que agrega
  el árbol de Hype **+ los productos propios agrupados por su `Category`** (una sección por
  categoría, ordenadas por `order` y luego alfabético). NO vuelvas a `/pin-hype/catalog` o
  desaparecen los productos propios.
- **Categorías (productos propios):** cada `OwnProduct` tiene una `Category` **obligatoria**
  (tabla propia, nombre único case-insensitive). Se gestionan en el panel (`/categories`, solo
  ADMIN: crear/renombrar/borrar-si-vacía). En la home cada categoría es una sección. El costo
  de cada código de inventario (`ProductCode.cost`) también es **obligatorio** al cargar.
  **OJO:** antes componía `getCollections` + `getCollection`, que solo miran el primer nivel;
  las colecciones cuyos productos viven en sub-colecciones (p. ej. Console → PlayStation)
  quedaban con `products: []` y se descartaban, así que **nunca se mostraban**. No vuelvas a
  ese patrón.
- **Sistema de componentes: `shadcn/ui`** (Radix, en `src/components/ui/`). Mantine se
  eliminó por completo (código y dependencias `@mantine/*` + `@emotion/react`); no lo
  reintroduzcas. Para animaciones usa Tailwind (`tailwindcss-animate`) o los primitivos de
  shadcn.
- **Gestor de paquetes: npm.** El lockfile canónico es `package-lock.json` (se eliminaron
  `pnpm-lock.yaml` y `bun.lockb`). No reintroduzcas otros lockfiles.
- **Puertos:** `vite.config.ts` declara `host: 0.0.0.0`, `port: 8080`, `allowedHosts: true`,
  pero `npm run dev` fuerza `--port 5173` (gana la CLI). La config de test está **duplicada**
  en `vite.config.ts` y `vitest.config.ts` (ambos importan de `vitest/config`).
- **`.env`** solo tiene `VITE_API_URL`. Otras vars opcionales que lee el código:
  `VITE_PIN_HYPE_COUNTRY|CURRENCY|LANGUAGE`, `VITE_FREE_FIRE_COLLECTION_ID`. No editar `.env`
  salvo petición explícita.

## Relación con el backend

`../hype-integration-2026` — API **NestJS + Prisma + PostgreSQL** que expone `/api`:
- Dominio propio: `auth` (JWT), `users`, `sales`.
- Wrapper del proveedor externo **Pin Hype** bajo `/pin-hype/*` (catalog, pre-redeem,
  reversal, report, echo) con token store y cliente HTTP propios.

Este frontend es un cliente de esa API; los tipos en `src/lib/api/*` y `src/lib/providers/*`
reflejan (a mano) los contratos del backend. Si cambia un DTO en el backend, hay que
actualizarlos aquí.

---
> Source: [Edwin035/topgg-hype](https://github.com/Edwin035/topgg-hype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
