## caligua

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Reglas

1. Pensar antes de actuar. Leer archivos existentes antes de escribir.
2. Salida concisa, razonamiento exhaustivo.
3. Editar antes que reescribir.
4. No releer archivos ya leídos salvo que hayan cambiado.
5. Probar antes de declarar listo.
6. Sin aperturas aduladoras ni cierres innecesarios.
7. Soluciones simples y directas.
8. Las instrucciones del usuario priman sobre este archivo.
9. Responder siempre en español neutro.

## Visión general

**Fénix 2.0** — POS full-stack para **Linux Mint** (servidor 24/7 con nginx + systemd). Monorepo con npm workspaces (`apps/*`, `packages/*`).

En producción: backend Node como servicio `systemd`, nginx sirve el frontend pre-compilado y proxea `/api` al backend en `:3000`. Acceso vía navegador en la red local (`http://IP_DEL_SERVIDOR`).

## Comandos

### Raíz
```bash
npm run dev          # backend + pos-tablet + pos-mobile en paralelo
npm run dev:tablet   # backend + pos-tablet
npm run dev:mobile   # backend + pos-mobile
npm run build        # build pos-tablet y pos-mobile
npm run start:backend
```

### Backend (`apps/backend`)
```bash
npm run dev              # nodemon
npm start                # producción
npm run build            # node --check sobre cada .js (no transpila)
npm run init-db          # crea esquema (src/scripts/init-db.js)
npm run create-admin     # src/scripts/createAdmin.js
node src/scripts/seed-coffee-shop.js       # data demo
node src/scripts/import-menu-from-text.js  # carga de menú
```

### pos-tablet / pos-mobile (`apps/<app>`)
```bash
npm run dev      # Vite
npm run build
npm run preview
```

### Despliegue Windows → USB → Linux Mint

Empaquetar en Windows (PowerShell):
```powershell
./pack-for-linux.ps1   # genera pos-fenix-YYYYMMDD-HHMM.tar.gz limpio en el Escritorio
```

`pack-for-linux.ps1` excluye `node_modules`, `dist`, `.git`, `.env` y artefactos Windows que rompen el despliegue.

En Linux Mint, tras extraer el `.tar.gz`:
```bash
chmod +x install-linux-24x7-autostart.sh deploy.sh

# Primera instalación (30–40 min, hace TODO)
./install-linux-24x7-autostart.sh

# Actualizaciones posteriores (1–3 min)
./deploy.sh
```

`deploy.sh` detecta y limpia `node_modules` con binarios Windows, convierte CRLF→LF, hace `npm install`, recompila los frontends, los republica en `/var/www/pos-fenix/{tablet,mobile}` y reinicia el backend.

Variables opcionales del install script: `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `ADMIN_NAME`, `ADMIN_EMAIL`, `ADMIN_PASSWORD`, `PUBLIC_PORT`.

### Portabilidad Windows ↔ Linux

- `.gitattributes` fuerza LF en todos los archivos de texto.
- `.editorconfig` mantiene LF + UTF-8 + 2-spaces en cualquier editor.
- Los `.sh` están marcados `+x` en git (`100755`).
- **Nunca** copies `node_modules` o `dist` entre sistemas — el `.gitignore` y `pack-for-linux.ps1` ya los excluyen.

## Layout

```
apps/backend     Express API (Node ESM, PostgreSQL via pg)
apps/pos-tablet  React/TypeScript (núcleo del sistema)
apps/pos-mobile  React/TypeScript (interfaz móvil)
packages/ui-kit  Componentes React compartidos
```

## Backend (Express, puerto 3000)

Patrón en capas: `routes/ → controllers/ → services/ → PostgreSQL` (pool en `src/config/db.js`). Auth con JWT (`utils/jwt.js`, `middlewares/auth.middleware.js`); autorización por rol (`authorize.middleware.js`, `role.middleware.js`).

**Gotcha:** `src/routes/index.js` no existe en este proyecto (a diferencia de lo que documenta `pos/CLAUDE.md`). Todas las rutas se montan a mano en `src/app.js`. Al agregar una ruta nueva hay que registrarla ahí.

Rutas montadas en `app.js`: `/api/auth`, `/api/categories`, `/api/products`, `/api/reports`, `/api/sales`, `/api/orders`, `/api/print`, `/api/users`, `/api/roles`, `/api/settings`, `/api/reservations`, `/api/bodega`, `/api/caja` (más `GET /api/stock-events` y `GET /api/health` sueltas).

`db.js` traduce placeholders `?` a `$1, $2…` de Postgres (`convertParams`), así que los services pueden escribir SQL con `?` como en SQLite. `withTransaction(fn)` envuelve BEGIN/COMMIT/ROLLBACK con un client dedicado del pool.

Subsistemas notables:
- **Bodega/BOM**: `insumos`, `recetas`, `movimientos_insumo` y `products.tipo_stock` (`'directo'` vs `'receta'`) en `db.init.js`; lógica en `bodega.service.js` / `bodega.controller.js` / `/api/bodega`.
- **Caja**: `cash-register.service.js` / `/api/caja` — apertura/cierre de caja, propio de este proyecto (no existe en fenix/tecpan).
- **Reservas**: `reservations.service.js` / `/api/reservations` — propio de este proyecto.
- **Reportes**: `services/reports.service.js` + `utils/pdf/` (`index.js` reúne datos y llama a builders puros en `utils/pdf/reports/*`; `theme.js`, `logo.js`, `charts.js`, `layout-letter.js`, `layout-mobile.js`) y `utils/exportExcel.js`.
- **Impresión**: `utils/escpos-builder.js`, `escpos-logo.js` (tickets ESC/POS).
- **Alertas de stock**: `services/stock-alert.service.js` — evalúa transiciones de nivel de stock (bajo/crítico) y notifica por Telegram (`telegram.service.js`). Config en `settings.key = 'stock_alert_notify_config'`, umbrales en `stock_alert_thresholds`, estado de transición en la tabla `stock_alert_state`.
- **Telegram**: `services/telegram.service.js` (config cifrada en `settings`, envío vía `fetch` nativo, sin dependencias — texto, foto y documento) + `services/telegram-listener.service.js` (long-polling `getUpdates`; responde a la palabra clave "informe" con la imagen del resumen del día). Es el único canal de envío de reportes (diarios/semanales/mensuales, cierres de caja) e inventario — no hay envío por email/SMTP en este proyecto. Arranca via `setImmediate` después de `server.listen` en `server.js`. Solo puede correr un poller por bot token a la vez — Telegram devuelve 409 si hay dos procesos activos (p. ej. `npm run dev` + una instancia ya corriendo).

`db.init.js` corre migraciones idempotentes (`CREATE TABLE/COLUMN IF NOT EXISTS`) marcadas por clave en `settings`; `npm run init-db` es seguro de re-ejecutar sobre una BD existente.

## Frontends

`apps/pos-mobile` usa `@pos/pos-core` (hooks compartidos en `packages/pos-core/src/hooks/`) para catálogo/checkout. `apps/pos-tablet` **no** lo usa — mantiene su propia lógica de venta en `features/pos/PosScreen.tsx`, con alias `@pos/pos-core` ausente de su `vite.config.ts`/`tsconfig.app.json`. Antes de tocar lógica de venta, confirmar en cuál de los dos apps se está trabajando y no asumir que un cambio en `pos-core` afecta a `pos-tablet`.

## Base de datos

PostgreSQL del sistema. Esquema completo en `apps/backend/src/config/db.init.js` (lo ejecuta `npm run init-db`). Conexión vía `apps/backend/.env`: `PG_HOST`, `PG_PORT`, `PG_USER`, `PG_PASSWORD`, `PG_DATABASE`, más `JWT_SECRET`, `PORT`, `HOST`.

## Otros scripts en la raíz

- `start.sh [tablet|mobile|all]` — arranque local: verifica Node/PostgreSQL, crea la BD si falta, corre `init-db` y lanza `npm run dev*`.
- `git-update.sh` — en el servidor: `git fetch/pull --ff-only` + `deploy.sh`. Aborta si el pull falla o si falta `apps/backend/.env`. Nunca toca PostgreSQL ni el `.env`.

---
> Source: [djmml12/caligua](https://github.com/djmml12/caligua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
