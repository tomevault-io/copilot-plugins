## aprendizaje-inventario-carpas

> **Sistema de Gestion de Inventario y Alquileres de Carpas** — Full-stack web application for managing inventory, quotations, rentals, and logistics operations for an event equipment rental business (tents, structures, accessories).

# CLAUDE.md

## Project Overview

**Sistema de Gestion de Inventario y Alquileres de Carpas** — Full-stack web application for managing inventory, quotations, rentals, and logistics operations for an event equipment rental business (tents, structures, accessories).

- **Backend**: Node.js + Express 5.x + MySQL (raw SQL via mysql2/promise)
- **Frontend**: React 19 + Vite + TailwindCSS + React Query + Zustand
- **Test Frameworks**: Jest (backend), Vitest (frontend)
- **Architecture**: Modular MVC (backend), Feature-module (frontend)

## Quick Reference

```bash
# Backend
cd backend
npm install
npm run dev          # nodemon server.js (port 3000)
npm test             # jest --verbose (812 tests)
npm run test:watch   # jest --watch

# Frontend
cd inventario-frontend
npm install
npm run dev          # vite dev server (port 5173)
npm run build        # production build
npm test             # vitest (watch mode, 70 tests)
npm run test:run     # vitest run (single pass)
```

## Repository Structure

```
/
├── backend/                    # Express API server
│   ├── config/                 # database.js, constants.js
│   ├── middleware/             # errorHandler, validator, upload, httpLogger
│   ├── modules/               # 7 feature modules (MVC each)
│   │   ├── auth/              # Login, JWT, roles, permissions
│   │   ├── inventario/        # Categories, elements, series, lotes, materials, units, locations
│   │   ├── alquileres/        # Quotations, rentals, transport, discounts, events
│   │   ├── productos/         # Composite products (bundles of elements)
│   │   ├── clientes/          # Clients, cities, departments
│   │   ├── operaciones/       # Work orders, employees, vehicles, alerts
│   │   └── configuracion/     # System settings, rental alerts
│   ├── utils/                 # AppError, logger, pagination, validators
│   ├── migrations/            # SQL migration scripts
│   ├── scripts/               # Admin setup, data migration scripts
│   ├── uploads/               # File storage (logos, images)
│   ├── server.js              # App entry point
│   └── jest.config.js
│
├── inventario-frontend/       # React SPA
│   ├── src/
│   │   ├── modules/           # 8 feature modules
│   │   │   ├── auth/          # Login, registration, ProtectedRoute
│   │   │   ├── inventario/    # Inventory CRUD pages
│   │   │   ├── productos/     # Product catalog
│   │   │   ├── alquileres/    # Quotation/rental management
│   │   │   ├── clientes/      # Client management
│   │   │   ├── operaciones/   # Work orders, field ops
│   │   │   ├── calendario/    # Calendar views (FullCalendar)
│   │   │   └── configuracion/ # Settings, alerts
│   │   ├── shared/            # Cross-module resources
│   │   │   ├── api/           # Axios config + interceptors
│   │   │   ├── components/    # 40+ reusable UI components
│   │   │   ├── hooks/         # Shared hooks
│   │   │   ├── stores/        # Zustand auth store
│   │   │   └── utils/         # helpers, validation, constants
│   │   ├── pages/             # Top-level pages (Dashboard, ModulosDashboard)
│   │   ├── App.jsx            # Router setup
│   │   └── main.jsx           # Entry point with providers
│   ├── vite.config.js
│   └── vitest.config.js
│
├── sql/                       # Full schema (00_SCHEMA_COMPLETO.sql)
└── docs/                      # Project planning and documentation
```

## Environment Setup

Copy `.env.example` to `.env` in the `backend/` directory:

```bash
# Required
DB_HOST=127.0.0.1
DB_USER=tu_usuario
DB_PASSWORD=tu_password          # REQUIRED - server won't start without it
DB_NAME=aprendizaje_inventario
JWT_SECRET=<64-char-hex-string>  # Generate: node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"

# Optional
PORT=3000
NODE_ENV=development             # development | production
FRONTEND_URL=http://localhost:5173
JWT_ACCESS_EXPIRES=15m
JWT_REFRESH_EXPIRES=7d
```

Frontend `.env` (in `inventario-frontend/`):
```bash
VITE_API_URL=http://localhost:3000/api
```

## Backend Conventions

### Module Structure

Each backend module follows:
```
modules/<name>/
├── controllers/       # Request handlers (exports.functionName)
│   └── __tests__/     # Jest tests per controller
├── models/            # Static class with SQL queries
├── routes/            # Express routers
├── services/          # Business logic (optional)
└── index.js           # Mounts all subroutes
```

### Model Pattern (Static classes, raw SQL, parameterized queries)

```javascript
class CategoriaModel {
  static async obtenerTodas() {
    const [rows] = await pool.query('SELECT * FROM categorias ORDER BY nombre');
    return rows;
  }
  static async obtenerPorId(id) {
    const [rows] = await pool.query('SELECT * FROM categorias WHERE id = ?', [id]);
    return rows[0];
  }
  static async crear(datos) {
    const [result] = await pool.query('INSERT INTO categorias SET ?', [datos]);
    return { id: result.insertId, ...datos };
  }
}
```

- Always use `?` placeholders — never interpolate values into SQL
- Return `rows[0]` for single-record queries, `rows` for lists
- Models are stateless static classes — no instances

### Controller Pattern

```javascript
exports.obtenerTodas = async (req, res, next) => {
  try {
    const data = await Model.obtenerTodas();
    res.json({ success: true, data, total: data.length });
  } catch (error) {
    next(error);  // Always delegate to global error handler
  }
};
```

- Every controller wraps logic in try/catch and calls `next(error)`
- Use `throw new AppError('message', statusCode)` for known errors
- Pagination: use `getPaginationParams(req.query)` and `getPaginatedResponse()`

### Response Format

```json
// Success
{ "success": true, "data": {...}, "message": "Operacion exitosa" }

// Success with pagination
{ "success": true, "data": [...], "pagination": { "page": 1, "limit": 20, "total": 100, "totalPages": 5 } }

// Error
{ "success": false, "status": "fail", "message": "Descriptive error message" }
```

### Route Pattern

```javascript
const router = express.Router();
router.use(verificarToken);  // All routes require auth

router.get('/', controller.obtenerTodas);
router.get('/:id', validateId(), controller.obtenerPorId);
router.post('/', controller.crear);
router.put('/:id', validateId(), controller.actualizar);
router.delete('/:id', validateId(), controller.eliminar);
```

- All routes require `verificarToken` unless explicitly public (login, registro)
- Use `validateId()` middleware for `:id` params
- Route files are mounted in the module's `index.js`, then in `server.js`

### Naming Conventions (Backend)

| Type | Convention | Example |
|------|-----------|---------|
| Model | PascalCase + `Model.js` | `CategoriaModel.js` |
| Controller | camelCase + `Controller.js` | `categoriaController.js` |
| Route | camelCase `.js` | `categorias.js` |
| Service | PascalCase + `Service.js` | `TokenService.js` |
| Test | `__tests__/<name>.test.js` | `categoriaController.test.js` |
| DB tables | snake_case plural | `cotizacion_productos` |
| DB columns | snake_case | `fecha_creacion`, `stock_minimo` |
| FK columns | `{singular}_id` | `categoria_id`, `cliente_id` |
| Estado fields | `estado` (not `status`) | `estado: 'activo'` |

### Method Naming (Spanish)

| Prefix | Purpose | Example |
|--------|---------|---------|
| `obtener*` | Read/GET | `obtenerTodas()`, `obtenerPorId()` |
| `crear*` | Create/POST | `crear()`, `crearMultiple()` |
| `actualizar*` | Update/PUT | `actualizar()`, `actualizarEstado()` |
| `eliminar*` | Delete | `eliminar()` |
| `verificar*` | Auth/validate | `verificarToken()`, `verificarPermiso()` |
| `generar*` | Generate | `generarAccessToken()`, `generarPDF()` |
| `buscar*` | Search | `buscarTarifa()` |
| `contar*` | Count | `contarTodas()` |

### Auth System

- JWT access tokens (15min) + refresh tokens (7d, stored in DB)
- Middleware chain: `verificarToken` -> `verificarRol(['admin'])` -> `verificarPermiso('modulo', 'accion')`
- Admin role bypasses all permission checks
- Account lockout: 5 failed attempts -> 15 min block
- Passwords: bcryptjs hashing

### Error Handling

- `AppError(message, statusCode)` for operational errors
- Global error handler in `middleware/errorHandler.js`
- Development: full stack traces sent to client
- Production: sanitized messages only, generic 500 for programming errors
- MySQL errors mapped: `ER_DUP_ENTRY` -> 409, `ER_NO_REFERENCED_ROW_2` -> 400

### Test Pattern (Jest, mock-based)

```javascript
jest.mock('../../../../config/database', () => ({ pool: { query: jest.fn() } }));
jest.mock('../../models/CategoriaModel');

const mockReq = (overrides = {}) => ({ body: {}, params: {}, query: {}, ...overrides });
const mockRes = () => { const r = {}; r.status = jest.fn().mockReturnValue(r); r.json = jest.fn().mockReturnValue(r); return r; };
const mockNext = () => jest.fn();

describe('obtenerTodas', () => {
  test('retorna categorias', async () => {
    CategoriaModel.obtenerTodas.mockResolvedValue([{ id: 1 }]);
    const res = mockRes();
    await controller.obtenerTodas(mockReq(), res, mockNext());
    expect(res.json).toHaveBeenCalledWith(expect.objectContaining({ success: true }));
  });
});
```

- All DB calls and models are mocked (no real DB in tests)
- Tests live in `__tests__/` folders next to controllers
- Each test file follows the same `mockReq/mockRes/mockNext` pattern

## Frontend Conventions

### Module Structure

```
modules/<name>/
├── api/           # Axios service functions (apiElementos.js)
├── components/    # Module-specific components
│   ├── cards/
│   ├── forms/
│   └── modals/
├── hooks/         # React Query hooks (useElementos.js)
├── pages/         # Routable page components
└── constants/     # Module constants (optional)
```

### State Management

- **Server state**: React Query (TanStack) for all API data
  - `staleTime: 5min`, `cacheTime: 10min`, `retry: 1`
  - Query keys follow array hierarchy: `['elementos', 'subcategoria', id]`
- **Client state**: Zustand for auth only (`shared/stores/authStore.js`)
  - Persisted to localStorage
  - Holds: `usuario`, `accessToken`, `refreshToken`, `isAuthenticated`

### Hook Patterns

```javascript
// Read hook
export const useGetElementos = (subcategoriaId) => {
  const { data, isLoading, error } = useQuery({
    queryKey: ['elementos', 'subcategoria', subcategoriaId],
    queryFn: () => elementosAPI.obtenerPorSubcategoria(subcategoriaId),
    enabled: !!subcategoriaId,
  });
  return { elementos: data?.data || [], isLoading, error };
};

// Mutation hook
export const useCreateElemento = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data) => elementosAPI.crear(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['elementos'] });
    },
  });
};
```

### API Layer

- Axios instance at `shared/api/Axios.config.js` with base URL `VITE_API_URL`
- Request interceptor: auto-attaches `Authorization: Bearer <token>`
- Response interceptor: auto-refreshes on 401 and retries original request
- API files export plain async functions: `export const obtenerTodas = () => api.get('/elementos')`

### Import Aliases (vite.config.js)

```javascript
'@shared'        -> 'src/shared'
'@auth'          -> 'src/modules/auth'
'@inventario'    -> 'src/modules/inventario'
'@productos'     -> 'src/modules/productos'
'@alquileres'    -> 'src/modules/alquileres'
'@clientes'      -> 'src/modules/clientes'
'@operaciones'   -> 'src/modules/operaciones'
'@calendario'    -> 'src/modules/calendario'
'@configuracion' -> 'src/modules/configuracion'
```

### Styling

- **TailwindCSS 4.x** — utility-first, no CSS modules
- Mobile/tablet-first design with `lg:` breakpoint for desktop
- Touch-optimized: 44px minimum touch targets, `active:scale-[0.97]` feedback
- Color tokens: primary (blue), success (green), danger (red), warning (yellow)

### Component Patterns

- Shared components: `Button`, `Card`, `Modal`, `ConfirmModal`, `Badge`, `PageHeader`, `Spinner`, `EmptyState`, `StatCard`
- Props: `isOpen`/`onClose` for modals, `value`/`onChange` for controlled inputs, `loading` for async state
- Toast notifications via Sonner
- Forms use React Hook Form
- Icons via Lucide React

### Frontend Naming

| Type | Convention | Example |
|------|-----------|---------|
| Component | PascalCase `.jsx` | `ElementoCard.jsx` |
| Hook | `use` prefix, camelCase | `useElementos.js` |
| API service | `api` prefix, camelCase | `apiElementos.js` |
| Page | PascalCase + `Page.jsx` | `ElementosPage.jsx` |
| Constants | UPPER_SNAKE_CASE | `ESTADOS`, `ESTADO_COLORS` |

## Database

- **MySQL** with 31 tables
- Key relationships:
  - `categorias` (self-referencing hierarchy) -> `elementos` -> `series` (individual) / `lotes` (batch)
  - `elementos_compuestos` -> `compuesto_componentes` -> `elementos`
  - `cotizaciones` -> `cotizacion_productos` -> `elementos_compuestos`
  - `cotizaciones` -> `alquileres` -> `ordenes_trabajo` -> `orden_elementos`
  - `clientes` -> `cotizaciones`
  - `empleados` / `vehiculos` -> `ordenes_trabajo`
- Full schema in `sql/00_SCHEMA_COMPLETO.sql`
- Indexes in `backend/migrations/add_indexes.sql`

## Key Technical Decisions

- **No ORM** — raw SQL with `mysql2/promise` parameterized queries throughout
- **Dual tracking**: elements use `series` (individual serial numbers) or `lotes` (batch quantities) based on `requiere_series` flag
- **Spanish naming**: all code uses Spanish for domain terms (obtener, crear, actualizar, eliminar, cotizacion, alquiler, etc.)
- **Modular monolith**: backend modules are self-contained but share DB pool and middleware
- **No CI/CD or Docker**: development-only project without automated deployment

## Known TODOs

- `backend/modules/alquileres/controllers/alquilerController.js:186` — Change series/lotes state to 'alquilado' on rental creation
- `backend/modules/alquileres/controllers/alquilerController.js:595` — Restore element state to 'disponible' on cancellation
- `backend/modules/operaciones/models/EmpleadoModel.js:325` — Filter employees by availability when date is provided
- `inventario-frontend/src/modules/alquileres/pages/AlquilerDetallePage.jsx:333` — Implement print functionality

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ander9606)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ander9606)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
