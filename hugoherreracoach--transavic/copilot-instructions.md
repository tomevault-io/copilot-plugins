## transavic

> Contexto del proyecto para agentes de IA. Léeme **antes** de tocar código.

# CLAUDE.md — Transavic

Contexto del proyecto para agentes de IA. Léeme **antes** de tocar código.

> **📚 Para profundizar en cualquier área:** ver `docs/arquitectura/` (5 documentos temáticos verificados contra código). Empezar por [`docs/arquitectura/README.md`](./docs/arquitectura/README.md) que tiene un mapa "si vas a tocar X, lee Y".

---

## 1. Qué es este proyecto

**Sistema interno de gestión de pedidos** para una distribuidora avícola en Lima, Perú que opera dos marcas comerciales bajo el mismo dueño:

- **Transavic** — marca principal (pollo, gallinas, menudencia)
- **Avícola de Tony** — segunda marca (mismo flujo)

**Dueño / cliente final:** Antonio Resurrección.
**Productos:** pollo (entero, despresado, filetes), carnes (res, cerdo), huevos.
**Modelo:** venta al por mayor y menor a restaurantes, mayoristas y consumidores finales.
**Cobertura operativa:** 18 distritos de Lima Metropolitana.
**Volumen actual:** ~30 pedidos/día, 6 motorizados, 4 asesoras, 1 admin.

**No es** un e-commerce público ni un marketplace. Es un **ERP ligero interno** para la operación diaria. Los clientes finales no se loguean — los pedidos los crean las asesoras al recibirlos por WhatsApp.

---

## 2. Stack técnico

| Área | Tecnología |
|---|---|
| Framework | **Next.js 15** (App Router, Server Components + Server Actions) |
| Lenguaje | **TypeScript** (`strict: true`) |
| UI | **TailwindCSS v4** + `react-icons` (Feather) |
| Auth | **NextAuth v5 beta** + Credentials provider + `bcrypt` |
| Base de datos | **Neon Postgres** vía `@neondatabase/serverless` (HTTP, no pool) |
| Validación | **zod** (en cada API route) |
| Drag & drop | `@hello-pangea/dnd` (fork mantenido de react-beautiful-dnd) |
| Mapas | `@react-google-maps/api` + Google Maps Platform (Maps JS, Directions, Geocoding, Places) |
| Imágenes a JPEG | `html-to-image` (para compartir tickets por WhatsApp) |
| Offline | `localStorage` (NO IndexedDB) — ver `src/lib/offline-queue.ts` |
| Hosting | **Vercel** (deploy continuo desde main) |

**No usar ORM.** Las queries son SQL directo con tagged template literals de Neon (`sql\`SELECT ... \``). Hay queries dinámicas con `sql.query(query, params)` cuando hace falta.

**No usar PWA con background GPS.** iOS lo bloquea. Para el repartidor estamos planificando envolver `/dashboard/mi-ruta` con **Capacitor** (wrapper nativo) para tener GPS en background.

---

## 3. Comandos

```bash
npm run dev      # Desarrollo local en http://localhost:3000
npm run build    # Build de producción
npm run start    # Servir build
npm run lint     # ESLint (next/core-web-vitals + next/typescript)
npm run seed     # ./scripts/seed.mjs — crea tablas users + pedidos + seed inicial
```

**Migraciones:** scripts .mjs manuales en `/scripts/`. **No hay sistema automatizado.** Ejecutarlos uno a uno:

```bash
node scripts/migrate-products.mjs
node scripts/migrate-estados.mjs
node scripts/migrate-direccion-mapa.mjs
node scripts/migrate-entregado-por.mjs
node scripts/migrate-despacho-v2.mjs
node scripts/run-migration.mjs            # agrega asesor_id a clientes
# scripts/migration_add_asesor_to_clientes.sql — ejecutar manualmente en Neon
```

Cuando agregues una migración nueva, **crear nuevo archivo `migrate-<feature>.mjs`** siguiendo el patrón existente (con `CREATE EXTENSION IF NOT EXISTS`, `ADD COLUMN IF NOT EXISTS`, etc.). No modificar migraciones existentes.

---

## 4. Variables de entorno

Definidas en `.env` (no comiteado). Las críticas:

| Variable | Para qué |
|---|---|
| `DATABASE_URL` | Conexión Neon (pooled) |
| `DATABASE_URL_UNPOOLED` | Conexión Neon directa (para migraciones largas) |
| `AUTH_SECRET` | Firma JWT de NextAuth |
| `AUTH_URL` | URL base para callbacks NextAuth |
| `NEXT_PUBLIC_MAPS_API_KEY` | Google Maps JS (cliente) |
| `Maps_SERVER_KEY` | Google Directions / Geocoding (server-side) — **ojo el naming inusual** (camelCase con M mayúscula y guión bajo, NO `MAPS_SERVER_KEY` ni `GOOGLE_MAPS_SERVER_KEY`) |
| `BASE_LATITUDE`, `BASE_LONGITUDE` | Fallback de ubicación del almacén; la fuente real es la tabla `settings.base_location` |

`ADMIN_USER`/`ADMIN_PASSWORD` están en `.env` pero **no se usan en código activo** (legacy del scaffolding inicial). La auth real lee de la tabla `users`.

---

## 5. Estructura del código (alto nivel)

```
src/
├── app/
│   ├── api/                      # Backend (Route Handlers)
│   │   ├── pedidos/              # CRUD de pedidos + transiciones
│   │   ├── despacho/             # Vista admin de despacho (kanban + asignación + ruta)
│   │   ├── repartidor/mi-ruta/   # Endpoint específico del repartidor
│   │   ├── clientes/, productos/, users/, analytics/, settings/, resumen-diario/
│   │   └── version/              # BUILD_ID para VersionChecker
│   ├── dashboard/                # UI con auth obligatoria
│   │   ├── (rutas por feature)/
│   │   └── layout.tsx            # Aplica DashboardLayout con sidebar
│   ├── login/, layout.tsx, page.tsx
├── components/                   # Componentes compartidos cross-feature
├── lib/
│   ├── types.ts                  # Pedido, Cliente, User, EstadoPedido, etc.
│   ├── data.ts                   # Queries reutilizables (fetchFilteredPedidos, fetchAsesores...)
│   ├── actions.ts                # Server actions (authenticate, doLogout)
│   ├── offline-queue.ts          # Queue de acciones del repartidor (localStorage)
│   └── utils.ts
├── auth.ts                       # NextAuth setup
├── auth.config.ts                # Callbacks + redirects por rol
└── middleware.ts                 # Protege /dashboard/*

scripts/                          # Migraciones manuales + seed
public/                           # transavic.jpg, avicola.jpg (logos para tickets)
```

**Path alias:** `@/*` → `./src/*`.

---

## 6. Roles y permisos

El sistema tiene **3 roles** (próximamente **4** con el módulo de producción):

| Rol | Quién es | Qué ve | Permisos clave |
|---|---|---|---|
| `admin` | Antonio (dueño) | Todo | Gestionar usuarios, productos, despacho, base_location, ver TODOS los pedidos |
| `asesor` | Vendedoras (Leslie, Yoshelin, Sarai, Yesica) | Solo sus pedidos y sus clientes | Crear pedidos y clientes; ver lista propia. Scoping en SQL por `asesor_id = userId` |
| `repartidor` | Motorizados (Marco, Yhorner, Anghelo, etc.) | Solo `/mi-ruta` con SUS pedidos del día | Cambiar estado de SUS pedidos. Scoping por `repartidor_id = userId` |
| `produccion` *(en implementación)* | Asistente de producción (en otro distrito que la oficina) | Cola del día + filtro búsqueda + ingresar pesos | A definir |

**Login redirige por rol** (ver `auth.config.ts:authorized`):
- `repartidor` → `/dashboard/mi-ruta`
- todos los demás → `/dashboard/nuevo-pedido`

**El scoping NO está en middleware**, está en cada query SQL. Si agregas un nuevo endpoint, **NO te olvides de filtrar por rol** (ver `lib/data.ts:fetchFilteredPedidos` como referencia).

---

## 7. Modelo de datos (resumen)

### Tablas

```
users              → auth + roles (admin/asesor/repartidor)
clientes           → directorio de clientes recurrentes (con asesor_id)
pedidos            → tabla central (DENORMALIZADA del cliente — ver §8)
pedido_items       → relación pedido↔producto con cantidad/unidad
productos          → catálogo (Pollo/Carnes/Huevos)
settings           → key/value JSONB (hoy solo 'base_location')
```

### Convenciones

- **DB columnas:** `snake_case` (`fecha_pedido`, `repartidor_id`).
- **TypeScript propiedades:** mantiene `snake_case` cuando viene de DB; `camelCase` cuando son campos derivados o de UI.
- **Fechas:** `DATE` para `fecha_pedido`, `TIMESTAMP WITH TIME ZONE` para timestamps de evento.
- **Timezone en queries:** SIEMPRE `(NOW() AT TIME ZONE 'America/Lima')::date` cuando se compara "hoy". Lima está en UTC-5 sin DST.
- **IDs:** UUID v4 (`uuid_generate_v4()`).
- **Numéricos:** `NUMERIC(6,2)` para distancias km, `DECIMAL(10,2)` para cantidades de productos, `DECIMAL(10,8)` para latitude / `DECIMAL(11,8)` para longitude.

### Decisión: pedidos denormalizados

`pedidos` **copia** `cliente`, `whatsapp`, `direccion`, `lat/lng` del `cliente` al crear el pedido. Esto es deliberado:

- Preserva historial — si el cliente cambia de dirección, los pedidos pasados no se reescriben.
- `cliente_id` se inserta en `pedidos` (vínculo "vivo") pero **no está en el tipo `Pedido` de TS** todavía. Si lo necesitas, agrégalo a `lib/types.ts`.

---

## 8. Máquina de estados del pedido

```
Pendiente ──asignar──▶ Asignado ──iniciar viaje──▶ En_Camino ──entregar──▶ Entregado
                          │                            │                    │
                          │                            ├──cancelar──┐       │
                          ├──entrega directa──┐        │            │       │
                          │                   ▼        ▼            ▼       ▼
                          └──fallar──▶  Fallido    Asignado    Asignado  (revertible)
```

**Estados:** `Pendiente | Asignado | En_Camino | Entregado | Fallido` (PascalCase con underscore).

**Reglas importantes:**

1. **Saltos permitidos** (no obligatorio pasar por En_Camino): el repartidor puede ir Asignado → Entregado/Fallido directo (entrega mostrador). Ver `api/pedidos/[id]/entregar/route.ts`.
2. **Reverso completo**: PATCH `/entregar` revierte cualquier completado de vuelta a `Asignado` limpiando timestamps.
3. **Fallido REQUIERE `razon_fallo`** (≥5 caracteres). Validado con zod refine.
4. **`entregado_por` se llena con `session.user.name`** desde quien dispara la transición — útil cuando admin marca por el repartidor.
5. **`distancia_km` se congela al asignar**, NO se sobreescribe al optimizar ruta. Solo `orden_ruta` y `duracion_estimada_min` cambian.

**Pronto se agregarán dos estados** (mejoras Antonio 2026): `En_Produccion` y `Listo_Para_Despacho` antes de `Asignado`. Cuidado al ampliar el enum: actualizar también `lib/types.ts`, validaciones zod en `/api/pedidos/[id]/route.ts` y los `CASE` de orden en queries.

---

## 9. Convenciones de código

- **Idioma:** Español en variables, funciones, comentarios, mensajes de UI, errores y commits. Excepción: identificadores estándar como `useState`, `Map`, etc. Mantener consistencia.
- **Validación de input en APIs:** zod siempre, antes de tocar DB. Patrón: `Schema.safeParse(body)` → si falla, 400 con `error.flatten().fieldErrors`.
- **Errores en APIs:** `try/catch` con `console.error("Mensaje:", error)` + `NextResponse.json({ error: "..." }, { status: N })`.
- **Status codes:** 400 input inválido, 401 no autenticado, 403 sin permisos, 404 no encontrado, 409 conflicto, 500 error servidor.
- **Auth check en cada API:** `const session = await auth(); if (!session?.user) return 401`. Si requiere admin: `session.user.role !== "admin"` → 403.
- **`export const dynamic = "force-dynamic"`** en rutas que dependen de sesión o leen DB en tiempo real.
- **Cliente Neon:** instanciar dentro del handler (`const sql = neon(process.env.DATABASE_URL!)`) — el cliente HTTP de Neon no es un pool, es seguro reinstanciar.
- **Componentes cliente:** `"use client"` en la primera línea cuando usan hooks/eventos.
- **Naming de archivos:** `kebab-case.tsx` (`dashboard-content.tsx`), excepto componentes compartidos (`PedidoForm.tsx`, `DashboardLayout.tsx`).
- **No usar emojis en strings de Paragraph de reportlab** (cuando generes PDFs) — usar texto plano.

---

## 10. Integraciones externas

### Google Maps Platform
- **Maps JS** (cliente): `useJsApiLoader({ googleMapsApiKey: NEXT_PUBLIC_MAPS_API_KEY, libraries: ["places"] })`.
- **Directions** (server): en asignar pedido, iniciar viaje, optimizar ruta. Usa `Maps_SERVER_KEY`.
- **Optimización de ruta:** Directions con `waypoints=optimize:true` — Google resuelve TSP heurístico. Límite 25 waypoints (handle remaining en `optimizar-ruta/route.ts`).
- **Fallback Haversine** si no hay key o falla Google (`haversineKm()` en `asignar/route.ts`).
- **Costo actual:** ~$48/mes consumido, dentro de los $200/mes gratis de Google. Margen amplio.

### Neon Postgres
- HTTP serverless driver — no es un pool, reinstanciar por request.
- Conexión pooled (`DATABASE_URL`) para uso normal; unpooled (`DATABASE_URL_UNPOOLED`) si necesitas transacciones largas o migraciones pesadas.
- **No hay migraciones automáticas** — ver §3.

### Vercel
- Deploy continuo desde `main`.
- BUILD_ID se lee en `/api/version` para que `VersionChecker.tsx` fuerce reload cuando hay nuevo deploy (evita repartidores con bundle viejo).

### Próximas (mejoras 2026)
- **Pusher Channels** (free tier) para tracking GPS en vivo.
- **Capacitor** para wrapper Android de `/mi-ruta`.
- **Gemini API** (free tier) para módulo de IA comercial. Anonimizar nombres de clientes antes de mandar a Gemini.
- **SUNAT PSE** (proveedor de facturación electrónica autorizado) para emisión de boletas/facturas — el cliente lo contrata por separado.

---

## 11. Patrones técnicos críticos

### 11.1 Optimistic updates + Offline queue (repartidor)

En `mi-ruta-content.tsx`, cuando el repartidor toca un botón de transición (Entregar, Fallar, Iniciar viaje):

1. Cambia estado **localmente primero** (UI inmediata).
2. Intenta llamar a la API.
3. Si está offline → encola en `localStorage` (`transavic_offline_queue`).
4. Cuando vuelve la conexión, `syncQueue()` reintenta (max 3 retries) y maneja conflictos (estado ya cambió en servidor) descartando sin error.

**Cualquier endpoint que el repartidor llame debe ser idempotente.** Si por la naturaleza optimistic se llama dos veces, no debe romper.

### 11.2 Polling para actualizaciones

- `/dashboard/despacho` refresca cada **15s** (auto).
- `/dashboard/mi-ruta` refresca cada **60s** (auto).
- **No hay websockets** (todavía — viene con Pusher para el módulo de tracking en vivo).

### 11.3 GPS bajo demanda

El navegador solo pide ubicación al repartidor cuando el mapa está visible o hay pedido `En_Camino`. Es decisión explícita para ahorrar batería.

### 11.4 VersionChecker

`/api/version` devuelve `BUILD_ID` de Vercel. `VersionChecker.tsx` lo lee cada cierto tiempo y fuerza `window.location.reload()` si cambió — evita repartidores con bundle viejo.

---

## 12. Gotchas (cosas que NO son obvias)

1. **Doble fuente de verdad estado/entregado**: la columna legacy `entregado BOOLEAN` se mantiene **sincronizada con `estado VARCHAR`** en cada PATCH. Si modificas el estado, también sincroniza `entregado`. Ver lógica en `/api/pedidos/[id]/route.ts:80-114`. Eventualmente eliminar `entregado`, `entregado_por`, `entregado_at` cuando ya no haya queries legacy que lo lean. Por ahora **NO eliminar**.
2. **`detalle` (texto del pedido) vs `detalle_final` (peso real entregado)** son campos distintos. El primero es lo que pidió el cliente; el segundo lo registra el repartidor/producción al pesar realmente.
3. **`Maps_SERVER_KEY`** está con mayúscula M y guión bajo bajo. No es typo, así está en `.env`.
4. **`cliente_id`** se inserta en `pedidos` (`api/pedidos/route.ts`) pero **no está en el tipo `Pedido` de TypeScript**. Si lo agregas, también actualiza `fetchFilteredPedidos` en `lib/data.ts` para que lo seleccione.
5. **`direccion_mapa`** es una columna agregada después (ver `migrate-direccion-mapa.mjs`) pero no en todos los lugares se usa. Es texto libre para notas de ubicación.
6. **El sidebar (`DashboardLayout.tsx`) filtra navegación por rol** vía `roles[]` y `adminOnly`. Si agregas una sección nueva, decide en qué roles aparece.
7. **Empresa**: el campo `empresa` en `pedidos` puede ser `"Transavic"` o `"Avícola de Tony"`. La UI muestra logos distintos según valor. **No agregar otras empresas sin coordinar con Antonio.**
8. **`fecha_pedido` es `DATE` (sin hora)** — comparaciones por día usan timezone Lima. `created_at` es `TIMESTAMP WITH TIME ZONE` para timing exacto.
9. **Offline queue usa `localStorage`** (no IndexedDB) — capacidad ~5-10MB, suficiente para una jornada del repartidor pero no para histórico largo.
10. **El precio (`precio_unitario` en pedido_items y `precio_compra`/`precio_venta` en productos) NO existe todavía.** Es parte de la Mejora 5 de las próximas mejoras. Si necesitas trabajar con margen, agregar migración primero.

---

## 13. Estado del proyecto (mayo 2026)

### En producción
- Sistema base v1 (pedidos, despacho, mi-ruta, productos, clientes, analytics, resumen).
- Deploy en `transavic.app` (Vercel).
- 6 motorizados activos, 4 asesoras, 1 admin.

### En implementación (Fase actual)
Las **8 mejoras** acordadas con Antonio (mayo 2026, S/ 4 000, 17 días, 50% pagado):

1. Pesos digitales + flujo completo de pedidos (nuevos estados: `En_Produccion`, `Listo_Para_Despacho`).
2. Guía de remisión digital con foto firmada.
3. Seguimiento del motorizado en vivo (Capacitor + Pusher).
4. Avisos automáticos entre áreas + metas diarias.
5. Dashboard comercial + metas + panel gerencial (objetivo basado en mes anterior, +15%).
6. Gestión de cobranzas (facturas pendientes 7/15 días, alertas de deuda vencida).
7. Integración SUNAT (facturación electrónica, 2 RUCs).
8. Seguimiento comercial con IA (Gemini): clientes inactivos/frecuentes, registro de actividad, recomendaciones.

Ver `propuesta-mejoras-transavic.pdf` para el detalle entregable al cliente.

### Próximas fases (no cotizadas aún)
- CRM con WhatsApp Business API (Antonio lo postpuso explícitamente).
- App iOS del repartidor (solo Android por ahora — todos los motorizados usan Android).

---

## 14. Para el próximo agente (tú, IA futura)

Antes de empezar cualquier tarea:

0. **Lee primero [`docs/arquitectura/README.md`](./docs/arquitectura/README.md)** — tiene un mapa "si vas a tocar X, lee Y" que te ahorra tiempo. Los 5 documentos temáticos tienen verificación contra código real.
1. **Si vas a modificar el flujo de estados del pedido**, lee `§8` de este archivo + `docs/arquitectura/04-flujos-de-negocio.md` § 3 (máquina de estados completa con diagrama Mermaid).
2. **Si vas a agregar una nueva tabla o columna**, crea un nuevo `scripts/migrate-<feature>.mjs` siguiendo el patrón. NO modifiques migraciones existentes ni el `seed.mjs`.
3. **Si vas a agregar una nueva API**, valida con zod, chequea sesión, scopea por rol, devuelve errores con status correcto. Usa `lib/data.ts:fetchFilteredPedidos` como referencia de cómo se filtra por rol.
4. **Si vas a tocar la pantalla del repartidor (`mi-ruta-content.tsx`)**, recuerda que toda acción debe pasar por `offline-queue` para que funcione sin internet. No llames `fetch` directo desde un botón.
5. **Si vas a integrar un servicio externo nuevo**, usa env vars (no hardcodes), y prefiere planes gratuitos para no generar costos a Antonio (ver propuesta: "se mantienen costos al mínimo").
6. **Cuando completes algo**, ofrece actualizar este CLAUDE.md con cualquier decisión nueva o gotcha que haya surgido. Mantenerlo vivo es lo que lo hace útil.

**Idioma**: responde en español al usuario, escribe código y comentarios en español. El dueño Antonio NO es técnico — si necesitas explicarle algo, usa lenguaje sencillo y enfoque en beneficios (no detalles técnicos).

---

## 15. Contactos y propiedad

- **Desarrollador:** Hugo Herrera (`eventonegocioslegendarios@gmail.com`)
- **Cliente / dueño del negocio:** Antonio Resurrección
- **Repo:** local en `/Users/hugoherrera/Programación/proyectos/transavic`
- **Deploy:** Vercel (cuenta de Hugo)
- **DB:** Neon (cuenta de Hugo)
- **Google Cloud (Maps API):** cuenta `hugoherreradeveloper@gmail.com`

---
> Source: [HugoHerreraCoach/transavic](https://github.com/HugoHerreraCoach/transavic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
