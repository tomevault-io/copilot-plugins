## rbsoftware

> Lee este archivo completo antes de tocar cualquier código.

# RBSoftware — AGENTS.md
# Reglas para agentes de IA

Lee este archivo completo antes de tocar cualquier código.

---

## Stack fijo

```
Backend:      FastAPI (Python) + SQLModel + Alembic
DB:           MySQL 8 (driver: PyMySQL)
Frontend:     Next.js + React + TypeScript
UI:           Tailwind CSS + shadcn/ui
Formularios:  React Hook Form
Estado:       Context + Zustand
Storage:      MinIO (S3 compatible)
Infra:        Docker + Docker Compose + Nginx
API:          REST + OpenAPI
Auth:         JWT access + refresh tokens en httpOnly cookies
```

No cambies el stack. No propongas SQLite, no propongas microservicios, no propongas ORMs alternativos.

---

## Layout del monorepo

```
/backend        FastAPI app
/frontend       Next.js app
/infra          nginx configs
compose.dev.yml
compose.prod.yml
db.sql          modelo de datos completo (DBML) — fuente de verdad
```

---

## Arquitectura: monolito modular por dominio

```
backend/app/
  core/           config, database, security, permissions, storage
  api/            router principal + health
  domains/
    auth/
    rbac/
    audit/
    catalog/
    commercial/
    inventory/
    production/
    fulfillment/
    integrations/
    academic/
```

Cada dominio tiene exactamente estas capas:
```
{domain}/
  models/       SQLModel table definitions
  schemas/      Pydantic schemas (Create / Read / Update)
  repositories/ persistencia — CRUD y queries
  services/     lógica de negocio
  routes/       FastAPI routers — delgados, sin lógica
```

---

## Estado actual del código

| Dominio | Estado |
|---|---|
| auth | modelos ✅ schemas ✅ repos ✅ servicios ✅ rutas ✅ |
| rbac | modelos ✅ schemas ✅ repos ✅ servicios ✅ rutas ✅ |
| audit | modelos ✅ schemas ✅ repos ✅ servicios ✅ rutas ✅ |
| catalog | modelos ✅ schemas ✅ repos ✅ servicios ✅ rutas ✅ |
| commercial | modelos ✅ schemas ✅ repos ✅ servicios ✅ rutas ✅ |
| inventory | modelos ✅ schemas ✅ repos ✅ servicios ✅ rutas ✅ |
| production | modelos ✅ schemas ✅ repos ✅ servicios ✅ rutas ✅ |
| fulfillment | modelos ✅ schemas ✅ repos ✅ servicios ✅ rutas ✅ |
| integrations | modelos ✅ schemas ✅ repos ✅ servicios ✅ rutas ✅ |
| academic | modelos ✅ schemas ✅ repos ✅ servicios ✅ rutas ✅ |
| frontend | páginas funcionales conectadas al backend |

Endpoints de auth:
```
POST   /auth/login
POST   /auth/logout
POST   /auth/refresh
GET    /auth/me
GET    /auth/users
POST   /auth/users
PATCH  /auth/users/{user_id}
PATCH  /auth/users/{user_id}/password
```

Endpoints de RBAC:
```
GET    /rbac/roles
POST   /rbac/roles
DELETE /rbac/roles/{public_id}
GET    /rbac/roles/{role_id}/permissions
POST   /rbac/roles/{role_id}/permissions/{permission_id}
DELETE /rbac/roles/{role_id}/permissions/{permission_id}
GET    /rbac/permissions
POST   /rbac/permissions
GET    /rbac/users/{user_id}/roles
POST   /rbac/users/{user_id}/roles/{role_id}
DELETE /rbac/users/{user_id}/roles/{role_id}
GET    /rbac/users/{user_id}/permissions
```

Frontend — interceptor de auto-refresh activo en `frontend/lib/api.ts`:
si una llamada retorna 401, intenta `POST /auth/refresh` automáticamente
y reintenta la request original. Si el refresh falla, redirige a `/login`.

---

## Estado del sistema

Todos los dominios están implementados. El sistema está en fase de
mantenimiento, mejoras y extensión de features.

---

## Modelo de negocio que debes entender

### El producto central: kits

Un kit es una caja con componentes para armar robots. Tiene BOM (lista de materiales) compuesta por:
- Componentes electrónicos (picking individual)
- Chasis MDF cortado con láser (tratado como componente en BOM, pero tiene biblioteca de archivos de corte en catalog)
- Tornillería en bolsitas (se arma en lotes anticipados)

### Catálogo
- `Product` con tipos `KIT` y `COMPONENT`
- Un kit tiene `kit_bom_items` que listan sus componentes
- El chasis es un componente en BOM + tiene archivos de corte asociados (no serializar a nivel de unidad)

### Comercial
- `SalesOrder` es la fuente comercial interna después de aprobación
- Orígenes: `WOO`, `POS`, `MANUAL`
- Mientras está `PENDING` o `UNPAID`, el origen externo puede modificarla
- Al aprobar (`APPROVED`):
  - Se congela el snapshot de la orden
  - Se crean registros de fulfillment
  - Se evalúa stock disponible
  - Se reserva lo disponible
  - Se genera necesidad de producción solo para lo faltante

### Inventario
- Inventario **agregado** por producto + ubicación + estado
- No hay serialización por unidad individual
- Reservar = mover unidades de `FREE` → `RESERVED_WEB` o `RESERVED_FAIR`
- Tablas separadas para kits terminados y componentes

Estados de inventario de kits:
`FREE`, `RESERVED_WEB`, `RESERVED_FAIR`, `PACKED`, `SHIPPED`, `DELIVERED`, `TO_DISASSEMBLE`, `LOST`, `DAMAGED`, `BLOCKED`

Estados de inventario de componentes:
`AVAILABLE`, `BLOCKED`, `DAMAGED`, `RESERVED`

### Producción
- Unidad operativa central: `ProductionBatch`
- Tipos: `SALES`, `STOCK`, `FAIR`, `MANUAL`
- Se produce **solo lo faltante**, no el total
- Hora de corte diaria: el sistema agrupa órdenes pendientes y genera batch automáticamente
- El administrador puede intervenir: agregar órdenes urgentes, retener, reordenar
- `production_batch_items` incluye: `required_qty_total`, `available_stock_qty`, `to_produce_qty`

### Fulfillment
- Packing flow por orden de venta
- QR de orden: token opaco, abre contexto operativo
- QR de kit: confirma que el kit correcto entra en el packing
- Componentes sueltos: confirmación manual por cantidad
- Cierre de packing: cambia `fulfillment_status` a `PACKED`

### Integrations
- WooCommerce: webhooks + sync incremental + cron fallback
- Wompi: confirmación de pagos
- `integration_sync_state`: estado de última sincronización por integración

---

## Reglas de implementación

### Rutas
```python
# ✅ Correcto — ruta delgada
@router.post("/login")
async def login(data: LoginRequest, service: AuthService = Depends(get_auth_service)):
    return await service.authenticate(data)

# ❌ Incorrecto — lógica en la ruta
@router.post("/login")
async def login(data: LoginRequest, session: Session = Depends(get_session)):
    user = session.exec(select(User).where(User.email == data.email)).first()
    # ... lógica directamente aquí
```

### Repositorios
```python
# ✅ Correcto — persistencia en repositorio
class UserRepository:
    async def get_by_email(self, email: str) -> User | None:
        ...

# ❌ Incorrecto — SQL en servicio
class UserService:
    async def authenticate(self, email: str):
        user = self.session.exec(select(User).where(...))  # MAL
```

### Modelos
- Siempre usar `public_id` (UUID) como identificador expuesto en APIs
- Nunca exponer el `id` interno en respuestas de API
- Siempre incluir `created_at` y `updated_at`

### Migraciones
- Siempre crear migración con `alembic revision --autogenerate -m "descripcion"`
- Siempre revisar el archivo generado antes de aplicar
- Nunca modificar el schema directamente en la DB
- Pensar en MySQL 8: usar tipos compatibles, no asumir SQLite

### QR y seguridad
- QR tokens son opacos — generados aleatoriamente, sin datos sensibles embebidos
- `qr_token` en `sales_orders` es un token de lookup, no un payload
- Nunca exponer PII en tokens o QR codes

---

## Lo que NO debes hacer

- ❌ Inventar tablas o campos no definidos en `db.sql` sin justificación
- ❌ Mezclar lógica de un dominio dentro de otro
- ❌ Poner lógica de negocio en rutas
- ❌ Escribir SQL directo en servicios (usa repositorios)
- ❌ Crear microservicios
- ❌ Usar SQLite como solución real
- ❌ Cambiar el stack tecnológico
- ❌ Serializar inventario por unidad individual (es agregado)
- ❌ Generar producción por el total de la orden (solo el faltante)
- ❌ Exponer `id` interno en APIs (usa `public_id`)
- ❌ Guardar refresh tokens en texto plano (siempre SHA256 hash)

---

## Antes de modificar código

1. Lee `db.sql` para entender el modelo de datos actual
2. Lee el estado de Alembic (`alembic history`)
3. Lee los archivos del dominio específico que vas a tocar
4. Entiende qué tablas ya tienen migración y cuáles no
5. Trabaja en este orden: diseño → implementación → migración → tests

---

## Contexto de negocio completo

Ver `PROJECT_CONTEXT.md` para entender el negocio, los flujos y las fases de construcción.
Ver `db.sql` para el modelo de datos completo y actualizado.

---

## Dominios implementados — notas

### academic (implementado)
Dominio LMS para colegios. Jerarquía:
Colegio → Grado (Director) → Curso (Teacher) → Estudiante

Tablas: schools, lms_grades, lms_grade_directors,
lms_courses, lms_course_students, lms_units, lms_materials,
lms_assignments, lms_submissions

Roles LMS: ADMIN / DIRECTOR / TEACHER / STUDENT
- DIRECTOR: personal del colegio, gestiona grados y cursos,
  no da clases, ligado a un solo colegio
- TEACHER: personal del colegio, gestiona contenido de sus
  cursos, puede estar en varios cursos, no dirige grados
- STUDENT: login propio, ve material publicado, entrega tareas

Storage: MinIO para PDFs y archivos de entregas
Keys: academic/{course_id}/materials/{uuid}.pdf
      academic/{course_id}/submissions/{assignment_id}/
      {student_id}/{uuid}.{ext}

Endpoints de academic:
```
GET    /academic/schools
POST   /academic/schools
GET    /academic/schools/{school_id}
PATCH  /academic/schools/{school_id}
GET    /academic/schools/{school_id}/grades
POST   /academic/schools/{school_id}/grades
GET    /academic/my-grades
GET    /academic/grades/{grade_id}
PATCH  /academic/grades/{grade_id}
POST   /academic/grades/{grade_id}/director
DELETE /academic/grades/{grade_id}/director
GET    /academic/grades/{grade_id}/courses
POST   /academic/grades/{grade_id}/courses
GET    /academic/my-courses
GET    /academic/courses/{course_id}
PATCH  /academic/courses/{course_id}
POST   /academic/courses/{course_id}/teacher
GET    /academic/courses/{course_id}/students
POST   /academic/courses/{course_id}/students
DELETE /academic/courses/{course_id}/students/{user_id}
GET    /academic/courses/{course_id}/content
GET    /academic/courses/{course_id}/units
POST   /academic/courses/{course_id}/units
PATCH  /academic/units/{unit_id}
POST   /academic/units/{unit_id}/publish
DELETE /academic/units/{unit_id}/publish
GET    /academic/units/{unit_id}/materials
POST   /academic/units/{unit_id}/materials          (FormData)
DELETE /academic/materials/{material_id}
GET    /academic/materials/{material_id}/download
GET    /academic/units/{unit_id}/assignments
POST   /academic/units/{unit_id}/assignments
PATCH  /academic/assignments/{assignment_id}
POST   /academic/assignments/{assignment_id}/publish
DELETE /academic/assignments/{assignment_id}/publish
GET    /academic/assignments/{assignment_id}/submissions
POST   /academic/assignments/{assignment_id}/submit  (FormData)
GET    /academic/assignments/{assignment_id}/my-submission
POST   /academic/submissions/{submission_id}/grade
POST   /academic/students/{student_id}/transfer
```

Frontend: /academic/schools, /academic/schools/[id],
/academic/grades, /academic/grades/[id],
/academic/courses, /academic/courses/[id]
Vista docente (2 paneles: unidades + contenido) y vista estudiante
(unidades expandibles con materiales + tareas + entregas)

## Control de acceso — Módulo Académico

Roles fijos del sistema:
`ADMIN`, `DIRECTOR`, `TEACHER`, `STUDENT`, `OPERATIVO`, `COMERCIAL`

Helper: `backend/app/core/permissions.py`
- `require_roles(*roles)` → devuelve un `Depends()` de FastAPI
- ADMIN siempre tiene acceso total (bypass implícito en el helper)

Cookie `user_roles`: no httpOnly, seteada en login y refresh.
Usada por `frontend/middleware.ts` para proteger rutas del frontend.
Si el usuario no tiene el rol requerido → redirige a `/dashboard?error=unauthorized`.

GET /auth/me retorna `roles: ["ADMIN", ...]` en el JSON.
Store Zustand: `roles`, `hasRole(role)`, `isAdmin()`.

Matriz de acceso académico:
- ADMIN → acceso total a todos los endpoints y páginas
- DIRECTOR → gestiona sus grados y cursos (`/academic/grades`, `/academic/courses`)
- TEACHER → gestiona contenido de sus cursos (units, materials, assignments)
- STUDENT → ve contenido publicado y entrega tareas (`/academic/courses`, submissions)

Nav condicional (`frontend/components/nav.tsx`): cada rol ve solo las secciones
que le corresponden. Si una sección no tiene ítems visibles, se oculta completa.

Dashboard personalizado (`frontend/app/(app)/dashboard/page.tsx`): métricas
diferentes según el rol del usuario (operativas, académicas, estudiantiles).

Endpoints protegidos en `backend/app/domains/academic/routes/`:
- `schools.py` → todos ADMIN
- `grades.py` → DIRECTOR (my-grades), ADMIN+DIRECTOR (resto), ADMIN (director assign)
- `courses.py` → get_current_user (lectura general), ADMIN+DIRECTOR (escritura), ADMIN+DIRECTOR+TEACHER (students list)
- `units.py` → get_current_user (list), ADMIN+TEACHER (escritura)
- `materials.py` → get_current_user (list/download), ADMIN+TEACHER (add/delete)
- `assignments.py` → get_current_user (list), ADMIN+TEACHER (escritura/submissions)
- `submissions.py` → STUDENT (submit/my-submission), ADMIN+TEACHER (grade)
- `students.py` → ADMIN+DIRECTOR (transfer)

## Precauciones con Docker

Los volúmenes de MySQL y MinIO son nombrados (`mysql_data_dev`, `minio_data_dev`)
para que persistan entre reinicios de contenedores.

**NUNCA ejecutar:**
```bash
docker compose down -v              # elimina TODOS los volúmenes y datos
docker volume rm rbsoftware_mysql_data_dev   # elimina la DB manualmente
```

**Para parar sin perder datos:**
```bash
docker compose -f compose.dev.yml down       # sin -v
docker compose -f compose.dev.yml stop       # solo detiene, no remueve
```

**Para reiniciar limpio intencionalmente:**
```bash
docker compose -f compose.dev.yml down -v
docker compose -f compose.dev.yml up --build -d
docker compose -f compose.dev.yml exec backend alembic upgrade head
docker compose -f compose.dev.yml exec backend python scripts/seed.py
```

---

## Deuda técnica conocida
- POST /auth/users no valida complejidad de contraseña en backend
- change_password solo permite cambiar la propia contraseña —
  extender cuando exista guard de permisos admin
- Dominios no académicos (commercial, inventory, production, fulfillment)
  no tienen require_roles aplicado aún — solo usan get_current_user
- N+1 en get_course_content del servicio (queries por unidad en loop)
- No hay límite de tamaño en subida de archivos (await file.read())

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hnpz22)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hnpz22)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
