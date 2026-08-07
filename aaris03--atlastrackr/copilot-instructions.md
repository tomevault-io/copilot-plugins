## atlastrackr

> App para repartidores: registra jornadas de trabajo, ingresos, gastos, vehículos y mantenimientos por kilometraje.

# AtlasTrackr — Contexto para Claude Code

## Qué es este proyecto
App para repartidores: registra jornadas de trabajo, ingresos, gastos, vehículos y mantenimientos por kilometraje.

---

## Stack

### Frontend (frontend/)
- React 19 + Vite 6
- React Router DOM 7 (SPA, sin SSR)
- Tailwind CSS 4
- Axios 1.x con interceptor de token en `src/api/axios.js`
- Estado global con Context API (`src/context/AuthContext.jsx`)

### Backend (backend/)
- Node.js 24 + Express 4
- Prisma 6 + PostgreSQL 18
- JWT (jsonwebtoken 9) — tokens de 7 días
- bcrypt 5 (10 rounds)
- Zod 3 para validación de schemas en todos los endpoints
- Jest + Supertest para tests de integración

---

## Estructura clave

```
AtlasTrackr/
├── frontend/src/
│   ├── api/axios.js              ← instancia axios con interceptor JWT
│   ├── context/AuthContext.jsx   ← estado global: user + vehículos
│   ├── components/               ← modales y Layout
│   ├── pages/                    ← Login, Register, Dashboard, Sessions, Expenses, Vehicles, Maintenance, Profile
│   └── utils/
│       ├── validators.js         ← validaciones de formularios
│       └── formatters.js         ← formatDate, formatCurrency
└── backend/
    ├── app.js                    ← Express + rutas + middlewares
    ├── server.js                 ← app.listen()
    ├── controllers/              ← lógica de negocio
    ├── middleware/auth.js        ← verifyToken (protege rutas privadas)
    ├── models/prisma.js          ← instancia PrismaClient (singleton)
    ├── routes/                   ← definición de rutas por módulo
    ├── utils/                    ← schemas Zod por módulo
    ├── test/                     ← Jest + Supertest (177 tests, todos pasando)
    └── prisma/
        ├── schema.prisma
        └── migrations/
```

---

## Convenciones del backend

### Estructura de un controlador
```js
// Importa desde models/prisma.js (nunca instanciar PrismaClient directo)
import prisma from '../models/prisma.js'

// Valida con Zod antes de tocar la DB
const parsed = schema.safeParse(req.body)
if (!parsed.success) return res.status(400).json({ error: parsed.error.issues })

// Respuestas: 200 OK, 201 Created, 400 bad input, 401 unauth, 403 forbidden, 404 not found, 409 conflict
```

### Rutas protegidas
Todas las rutas excepto `/api/auth/*` usan el middleware `verifyToken` de `middleware/auth.js`. El usuario autenticado queda en `req.user`.

### Soft delete vs Hard delete
- **Soft delete**: vehicles, maintenance_types, maintenance_records → setear `deleted_at = NOW()`, `is_active = false`
- **Hard delete**: expenses → DELETE directo. Al eliminar un expense → soft delete en cascade de sus `maintenance_records`

### Transacciones Prisma
Usar `prisma.$transaction()` en:
- Registro de usuario (crea 14 tipos de mantenimiento por defecto)
- Finalizar jornada
- Crear expense de tipo `maintenance` (crea `maintenance_record` automáticamente)

### Zona horaria
El frontend genera "hoy" con `getLocalDateString()` (`frontend/src/utils/formatters.js`), basado en hora local del navegador, no UTC. El backend valida fechas "no futuras" con tolerancia de ±1 día (`getDateStringWithOffset()` en `backend/utils/dateHelpers.js`) para absorber el desfase entre la fecha local del usuario (Chile, UTC-4) y la fecha UTC del servidor. `formatDate()` ya no usa el hack `setUTCHours(12)`.

---

## Convenciones del frontend

### Llamadas al backend
Siempre usar la instancia de `src/api/axios.js`, nunca `fetch` ni una instancia nueva de axios. El interceptor agrega el JWT automáticamente.

### Estado global
`AuthContext` maneja usuario autenticado y lista de vehículos. Consumir con `useContext(AuthContext)`. No duplicar este estado en componentes locales.

### Formularios
Validar con las funciones de `utils/validators.js` antes de llamar al backend.

### Modales
Pattern estándar: `show` (boolean) + `onClose` (función) como props. El estado de apertura vive en el padre (lifting state up).

---

## Modelos de datos clave

### work_sessions — estados
- `started` → jornada activa
- `finished` → jornada finalizada normalmente
- `voided` → jornada anulada (no cuenta en métricas)

### Cálculos de negocio
```
base_earnings  = orders × order_value
total_earnings = base_earnings + tips
km_work        = SUM(km_end - km_start) de jornadas finished con reserve_km = true
km_total       = km_actual - km_base
km_personal    = km_total - km_work
net_income     = total_earnings + base_salary - SUM(expenses.cost)
```

### vehicle_km_adjustments — reglas
- Diferencia ≤ 300 km → ajuste normal
- Diferencia > 300 km → requiere `force: true` (confirmación explícita del usuario)

---

## Comandos útiles

```bash
# Frontend
cd frontend && npm run dev         # Vite en localhost:5173

# Backend
cd backend && npm run dev          # nodemon en localhost:3000
cd backend && npm test             # Jest (177 tests de integración)

# Base de datos
cd backend && npx prisma studio    # GUI para explorar la DB
cd backend && npx prisma migrate dev --name <nombre>  # nueva migración
cd backend && npx prisma db seed   # seed inicial
```

---

## Estado actual del MVP

### Pendiente prioritario
Ninguno — el MVP está funcionalmente completo (dashboard, feedback, gastos, mantenimientos, vehículos, sesiones). Lo único diferido a propósito es pricing/effort para `feature_request` en feedback (ver sección de feedback más abajo).

### Completado recientemente
- ✅ Tests sobre DB separada (`atlas_tracker_test` con `jest.config.js` + `.env.test`)
- ✅ Métricas en Expenses (tarjetas superiores)
- ✅ Gráfico donut de gastos por categoría con filtros móviles
- ✅ Gráfico de línea de evolución de gastos con filtros móviles
- ✅ Bloqueo de edición de km_base cuando hay datos vinculados (backend rechaza con 409 + frontend deshabilita input con mensaje explicativo)
- ✅ Ajuste automático de odómetro al crear/editar mantenimientos y gastos con km superior al actual (≤300 km silencioso, >300 km banner ámbar inline + force:true)
- ✅ Dashboard completo (resumen mensual/anual/global, kilometraje, promedios, rentabilidad, mantenciones próximas/vencidas) + sueldo base mensual (`monthly_salaries`)
- ✅ Sistema de feedback (bug/comentario/feature_request) con mensajería bidireccional usuario↔admin
- ✅ Cobertura de tests ampliada a dashboard, sueldos base, perfil, tipos de mantenimiento, configuración de mantenimiento por vehículo y rango de km de sesiones — 177 tests pasando

### Fuera del MVP (no implementar)
- Recuperación de contraseña
- Roles Admin/User
- App móvil (React Native)
- OAuth (Google/Facebook)
- Refactor Register con Zod

---

## Reglas generales
- No usar `any` en TypeScript (si se agrega TS al frontend en el futuro)
- No instanciar PrismaClient fuera de `models/prisma.js`
- No crear instancias de axios fuera de `api/axios.js`
- Antes de crear un endpoint nuevo, verificar si ya existe en la lista de rutas del MVP
- Los tests en `backend/test/` deben pasar siempre — no romper los 177 existentes
- Ante duda sobre una regla de negocio, preguntar antes de asumir

---

## Decisiones de diseño tomadas

### Gráficos de Expenses — ✅ Implementado

#### Donut (gastos por categoría)
- Componente `ExpensesDonutChart` independiente con `refreshKey` prop
- Selector **Mes / Año / Todo** con navegación ◀ ▶ limitada al primer gasto
- Desglose por los 7 tipos reales (`fuel`, `maintenance`, `clothing`, `food`, `toll`, `tools`, `other`) — solo muestra los que tienen monto > 0
- SVG puro sin librerías; ángulos calculados desde montos exactos para evitar huecos por redondeo
- Endpoint `/expenses/metrics` extendido con `period`, `byType` y `firstExpenseDate`; `breakdown` y `current` intactos para las tarjetas métricas

#### Línea (evolución de gastos)
- Componente `ExpensesLineChart` independiente con `refreshKey` prop
- Toggle **Acumulado / Por día|Por mes** para cambiar el modo de visualización
- Selector **Mes / Año** con navegación ◀ ▶ limitada al primer gasto
- SVG puro con tooltip hover, área de gradiente y eje Y dinámico
- Nuevo endpoint `GET /expenses/evolution?period=month|year&month=YYYY-MM&year=YYYY`
  - Modo `month`: un punto por día, devuelve `{label, date, total, accumulated}`
  - Modo `year`: un punto por mes, agrupa días correctamente en el backend
- Ambos gráficos se refrescan automáticamente al crear/editar/eliminar un gasto via `donutRefresh` en `Expenses.jsx`

### Soft delete de vehículos — ✅ Implementado
- `DELETE /api/vehicles/:id` → soft delete: `is_active = false`, `deleted_at = NOW()`
- Validar que el usuario tenga al menos 2 vehículos activos antes de eliminar (para que quede 1)
- En cascade: `vehicle_maintenance_config` también se desactiva (`is_active = false`)
- No tocar: `work_sessions`, `expenses`, `maintenance_records` — son histórico inmutable
- `maintenance_types` no se tocan — pertenecen al usuario, no al vehículo
- `PATCH /api/vehicles/:id/reactivate` → reactiva el vehículo y también sus `vehicle_maintenance_config`
- `GET /api/vehicles` admite `?include_inactive=true` para incluir archivados (por defecto solo activos)
- Frontend: vehículos inactivos desaparecen de todos los selectores
- Frontend: en la vista Vehicles aparecen como archivados con botón Reactivar

---
> Source: [Aaris03/AtlasTrackr](https://github.com/Aaris03/AtlasTrackr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
