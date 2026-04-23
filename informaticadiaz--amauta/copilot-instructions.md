## amauta

> Este documento proporciona contexto e información relevante para Claude Code al trabajar en el proyecto Amauta.

# CLAUDE.md

## Información del Proyecto Amauta

Este documento proporciona contexto e información relevante para Claude Code al trabajar en el proyecto Amauta.

## Descripción del Proyecto

Amauta es un sistema educativo para la gestión del aprendizaje.

## Visión y Filosofía ⭐

**IMPORTANTE**: Antes de desarrollar, entender la visión del proyecto.

- **Documento de Visión**: `README.md` - Filosofía, principios de diseño, valores
- **Propósito**: Educación como derecho social, acceso universal, offline-first
- **Nombre**: "Amauta" (quechua) = maestro/sabio al servicio de la comunidad

> _"No concebimos la educación como un producto, sino como un derecho social."_

---

## 🚦 Estado Actual y Próximos Pasos

### Fase Actual: Fase 5 - Comunidad y Colaboración 🚧 En Progreso

**Progreso**: Sprint 16 en progreso — 4/9 issues completados

#### Completado en Fase 5:

- ✅ **F5-001**: Refinar historias y criterios de aceptación de comunidad — planning
- ✅ **F5-002**: Matriz de dependencias UI/Backend para foros y comunidad
  - Modelos Prisma a crear: `ForoPost`, `ForoRespuesta`, `ReaccionForo`, `Notificacion`
  - Grafo de dependencias: F5-004 → F5-005 → F5-006 / F5-007 → F5-008 (F5-006 y F5-007 en paralelo)
  - Orden verificado: correcto; F5-009 puede ir en paralelo con F5-006 y F5-007
  - Documentado en `docs/project-management/roadmap.md` → Sección "Matriz de Dependencias UI/Backend — Fase 5"
- ✅ **F5-003**: Diseño funcional de flujos de foros, reportes y mensajes — planning
  - Ciclo de vida de ForoPost (PUBLICADO → CERRADO → ELIMINADO, soft delete, transiciones por rol)
  - Flujo de solución: solo PREGUNTA, una por post, toggle si cambia, notificación al autor
  - Moderación directa por EDUCADOR (su curso) y ADMIN_ESCUELA (toda la institución)
  - Sin modelo Reporte formal en Fase 5 — moderación directa es suficiente para comunidades pequeñas
  - Mensajes directos: fuera del alcance de Fase 5 (Fase 6+)
  - Documentado en `docs/project-management/fase-5-diseno-funcional-foros.md`
- ✅ **F5-004**: Prisma base de comunidad — modelos `ForoPost`, `ForoRespuesta`, `ReaccionForo`, `Notificacion`
  - 3 enums nuevos: `TipoForoPost`, `EstadoForoPost`, `TipoNotificacion`
  - 4 modelos nuevos con relaciones y índices
  - Migración `20260409000100_comunidad_foros_base`
  - Documentado en `docs/ai-context/database/schema.md`

#### Próximos pasos:

- 📋 **F5-005**: API de foros por curso: crear/listar posts y respuestas

**Documento guía**: `docs/project-management/roadmap.md` → Sección "Fase 5"

---

### Fase Anterior: Fase 4 - Módulo Escolar ✅ COMPLETADA

**Progreso**: Sprint 15 completado ✅ (3/3 completados) — Fase 4 completa

#### Completado en Fase 4:

- ✅ **F4-001**: Refinar historias y criterios de aceptación (Fase 4) — planning
- ✅ **F4-002**: Matriz de dependencias UI/Backend (Fase 4) — planning
- ✅ **F4-003**: Diseño funcional de flujos administrativos (roles) — planning
- ✅ **F4-004**: API Periodos Académicos + Escala de Calificación (Institución)
  - Modelos Prisma: `PeriodoAcademico`, `EscalaCalificacion` + migración
  - Módulo `instituciones` con service, controller y DTOs Zod
  - CRUD de períodos por institución (soft delete con `activo = false`)
  - Endpoint upsert de escala de calificación por institución (1:1)
  - 24 tests (98% statements, 100% funciones)
- ✅ **F4-005**: API Gestión de Grupos/Clases (CRUD + estados)
  - Relación con `PeriodoAcademico` y filtros por periodo/estado
  - Validación de institución y educador
  - Tests unitarios de controller y service
- ✅ **F4-006**: UI Gestión de Grupos/Clases (Listado + Form)
  - Endpoint backend `GET /instituciones/mi-institucion` para ADMIN_ESCUELA
  - Componentes `GruposList` (filtros estado/período) y `GrupoForm` (crear/editar)
  - Páginas: `/dashboard/grupos`, `/dashboard/grupos/nuevo`, `/dashboard/grupos/[id]/editar`
  - Proxies Next.js: `/api/grupos`, `/api/grupos/[id]`, `/api/mi-institucion`
  - "Grupos" en el sidebar (solo ADMIN_ESCUELA)
  - 19 tests frontend (2 suites, 100% pasando)
- ✅ **F4-010**: API listado de estudiantes y educadores por institución
  - Endpoints `GET /instituciones/:id/estudiantes` y `GET /instituciones/:id/educadores`
  - Filtros opcionales por búsqueda (`buscar`) y paginación (`page`, `limit`)
  - Validación de permisos (ADMIN_ESCUELA/SUPER_ADMIN)
- ✅ **F4-009**: UI Asignación de estudiantes y educadores a grupos
  - Páginas: `/dashboard/grupos/[id]/estudiantes` y `/dashboard/grupos/[id]/educadores`
  - Selector de usuarios por institución con búsqueda y paginación
  - Preview de asignaciones (agregados/duplicados/errores)
  - Gestión de asignaciones actuales con remoción y confirmación
- ✅ **F4-011**: API Registro diario de asistencias por grupo
  - Endpoints `GET /grupos/:grupoId/asistencias` y `PUT /grupos/:grupoId/asistencias`
  - Nómina activa por fecha con asistencia existente del día
  - Registro masivo por grupo/fecha con estados `PRESENTE`, `AUSENTE`, `TARDANZA` y `JUSTIFICADO`
  - Permisos para `ADMIN_ESCUELA` y educadores asignados al grupo
  - Edición solo el mismo día con observación obligatoria cuando cambia un registro existente
  - Tests unitarios de controller y service + documentación del contrato público
- ✅ **F4-012**: UI Carga rápida de asistencias por grupo
  - Página `/dashboard/asistencias` para `ADMIN_ESCUELA` y `EDUCADOR`
  - Selector de grupo/fecha con origen de datos según rol
  - Proxies Next.js `GET/PUT /api/grupos/:id/asistencias` y `GET /api/educadores/me/grupos`
  - Grilla por estudiante con estados rápidos, observación y cambios pendientes
  - Tests de UI, navegación y proxies frontend
- ✅ **F4-013**: API Resumen mensual de asistencias por grupo
  - Endpoint `GET /grupos/:grupoId/asistencias/resumen-mensual?mes&anio`
  - Resumen por estudiante con presentes, ausencias, tardanzas, justificados y porcentaje de asistencia mensual
  - Resumen agregado del grupo con total de registros y distribución por estado
  - Permisos para `ADMIN_ESCUELA` y educadores asignados al grupo
  - Solo considera estudiantes activos del grupo al momento de la consulta
  - Tests unitarios de controller y service + documentación de limitaciones iniciales

- ✅ **F4-014**: Prisma Calificaciones por periodo académico e institución
  - Modelo `Calificacion` alineado con `PeriodoAcademico` (FK real vs campo libre)
  - Eliminados campos `periodo String` y `notaMaxima Float` (redundante con `EscalaCalificacion`)
  - Unique constraint `[grupoId, estudianteId, periodoAcademicoId, materia]`
  - Índices compuestos para consultas por grupo+periodo y estudiante+periodo
  - Migración versionada `20260329000100_calificaciones_periodo_academico`
- ✅ **F4-015**: API Carga y listado de calificaciones por periodo
  - Módulo `calificaciones` con service, controller y DTOs Zod
  - `GET /grupos/:grupoId/calificaciones?periodoAcademicoId=&materia=` — nómina con notas (null si no existe)
  - `PUT /grupos/:grupoId/calificaciones` — upsert masivo validando escala y nómina activa
  - Validación: nota dentro del rango de `EscalaCalificacion` de la institución
  - Validación: período académico perteneciente a la misma institución del grupo
  - Permisos: ADMIN_ESCUELA (por institución) y EDUCADOR (por asignación activa al grupo)
  - 8 tests unitarios (86.88% statements)
- ✅ **F4-016**: UI Carga rápida de calificaciones por grupo y periodo
  - Componente `CalificacionesRapidasSection` con selectores de grupo, período académico y materia
  - Grilla editable de notas y observaciones; guarda solo los cambios pendientes
  - Exportación de resumen descargable en CSV
  - Página `/dashboard/calificaciones` (ADMIN_ESCUELA y EDUCADOR)
  - Proxies Next.js: `GET/PUT /api/grupos/:id/calificaciones` y `GET /api/instituciones/:id/periodos`
  - "Calificaciones" en el sidebar para ADMIN_ESCUELA y EDUCADOR
  - 5 tests frontend (2 suites, 5/5 pasando)

#### Próximos pasos:

- Sprint 15 completado ✅ — Fase 4 completada

#### Issues de infraestructura completados:

- ✅ **Issue #71**: CD Pipeline automático con GitHub Actions + Dokploy
  - Tests activados en CI (API + Web, antes comentados)
  - Job `deploy` dispara webhook de Dokploy solo en push a master
  - Healthcheck post-deploy confirma que la API responde
  - Documentado en `DEPLOYMENT_PROGRESS.md`
  - **Pendiente manual**: configurar secret `DOKPLOY_WEBHOOK_URL` en GitHub Settings

**Documento guía**: `docs/project-management/roadmap.md` → Sección "Fase 4"

### Fase Anterior: Fase 3 - Evaluaciones ✅ COMPLETADA

**Progreso**: 12/12 issues completados ✅ FASE 3 COMPLETADA

**Documento guía**: `docs/project-management/roadmap.md` → Sección "Fase 3"

#### Completado en Fase 1:

- ✅ **F1-001**: Autenticación con NextAuth.js v5
  - Login y registro funcionales
  - Páginas: `/login`, `/register`, `/dashboard`
  - Endpoints: `/api/v1/auth/login`, `/api/v1/auth/register`

- ✅ **F1-002**: Autorización por roles (RBAC)
  - Guards globales: `JwtAuthGuard`, `RolesGuard`
  - Decoradores: `@Roles()`, `@Public()`, `@CurrentUser()`
  - Hook frontend: `useAuthorization`
  - Middleware con protección por rol

- ✅ **F1-003**: Layout base responsive
  - Tailwind CSS v3 configurado
  - Componentes: `Header`, `Footer`, `Sidebar`, `MobileMenu`
  - Layouts: `MainLayout`, `DashboardLayout`
  - Navegación condicional por rol
  - Responsive: mobile, tablet, desktop

- ✅ **F1-004**: API CRUD de cursos
  - Endpoints REST completos para gestión de cursos
  - Validación con class-validator

- ✅ **F1-005**: UI para crear y editar cursos
  - Formularios de creación y edición
  - Integración con API

- ✅ **F1-006**: Sistema de subida de imágenes
  - Upload de imágenes para cursos
  - Almacenamiento en servidor

- ✅ **F1-007**: API CRUD de lecciones
  - Endpoints REST para lecciones
  - Asociación con cursos

- ✅ **F1-008**: UI para gestión de lecciones
  - Interfaz para crear/editar lecciones
  - Ordenamiento de lecciones

- ✅ **F1-009**: Catálogo público de cursos
  - Página `/cursos` con listado público
  - Filtros y búsqueda

- ✅ **F1-010**: Página de detalle de curso
  - Vista detallada del curso
  - Información de lecciones

- ✅ **F1-011**: API sistema de inscripción
  - Endpoints: inscribirse, cancelar, mis cursos, estado de inscripción
  - Tests unitarios: 26 tests (100% cobertura service y controller)

- ✅ **F1-012**: UI de inscripción y mis cursos
  - Botón inscripción funcional en página de detalle de curso
  - Página `/dashboard/mis-cursos` con grid de cursos inscritos
  - Componente `MiCursoCard` con barra de progreso y estado
  - Modal de confirmación para cancelar inscripción
  - Toast de feedback al inscribirse/cancelar
  - API Routes proxy: inscribir (POST/DELETE), estado (GET), mis-cursos (GET)

- ✅ **F1-013**: Visualizador de lecciones para estudiantes
  - Página `/cursos/[slug]/lecciones/[leccionId]` con control de acceso (solo inscritos)
  - Componente `LeccionSidebar` con lista de lecciones, indicador de progreso, colapsable
  - Componente `LeccionContent` para texto (HTML) y video (YouTube/Vimeo/local)
  - Componente `LeccionNavigation` con botones anterior/siguiente
  - `MobileSidebarSheet` drawer para navegación en móvil
  - Botón "Continuar curso" en InscripcionBtn ahora lleva a la primera lección

- ✅ **F1-014**: API seguimiento de progreso
  - Módulo `progreso` con service, controller y tests
  - `POST /lecciones/:id/completar` — marcar lección completada (idempotente)
  - `GET /cursos/:id/progreso` — progreso del estudiante en el curso (incluye `leccionesCompletadasIds`)
  - `GET /cursos/:id/estudiantes/progreso` — progreso de todos los estudiantes (educador)
  - Actualización automática del porcentaje en la inscripción
  - Marca inscripción como COMPLETADO al llegar al 100%
  - 21 tests (100% statements, 89% branches)

- ✅ **F1-015**: UI marcar lecciones completadas
  - Componente `CompletarLeccionBtn` con actualización optimista del UI
  - Componente `ProgresoBar` con barra visual, porcentaje y conteo de lecciones
  - `LeccionSidebar` actualizado: checkmarks basados en datos reales de la API
  - Visualizador de lección muestra progreso real y botón completar
  - API Routes proxy: completar lección (POST), progreso curso (GET)
  - 40 tests frontend (5 suites) + 21 tests backend

- ✅ **F1-016**: Dashboard de estudiante
  - Página `/dashboard` rediseñada con secciones personalizadas por rol
  - Componente `ContinuarAprendiendo`: muestra cursos activos con progreso y botón "Continuar"
  - Componente `ResumenProgreso`: estadísticas (en progreso, completados, total inscritos)
  - Componente `CursosRecomendados`: cursos del catálogo no inscritos aún
  - Dashboard diferenciado para estudiantes vs educadores/admins
  - 19 tests frontend (3 suites, 100% statements)

### Fase Anterior: Fase 2 - Offline-First PWA ✅ COMPLETADA

**Progreso**: 8/8 issues completados ✅ FASE 2 COMPLETADA

**Documento guía**: `docs/project-management/roadmap.md` → Sección "Fase 2"

### Fase Anterior: Fase 0 ✅ COMPLETADA

**Fecha de completitud**: 30/12/2024

- ✅ Infraestructura (monorepo, CI/CD, Docker)
- ✅ Base de datos (Prisma, 15 modelos, seed completo)
- ✅ Deployment en producción (Dokploy)
- ✅ Documentación técnica y de gestión

---

## 🗺️ Desarrollo Ordenado (CRÍTICO)

### Regla Principal

> **Para desarrollar features nuevas, SIEMPRE consultar `docs/project-management/roadmap.md`**

El roadmap define:

- 10 fases incrementales con prioridades claras
- Historias de usuario para cada fase
- Stack tecnológico específico por feature
- Criterios de éxito medibles
- Código de ejemplo y patrones

### Proceso de Desarrollo por Fases

```
1. CONSULTAR ROADMAP
   └── Leer la fase correspondiente en roadmap.md
   └── Entender historias de usuario y criterios de éxito

2. CREAR ISSUES
   └── Desglosar la fase en issues específicos (gh issue create)
   └── Usar labels: phase-1, phase-2, etc.
   └── Referenciar sección del roadmap en cada issue

3. IMPLEMENTAR
   └── Seguir workflow de WORKFLOW.md
   └── Usar TodoWrite para tracking
   └── Respetar stack técnico definido

4. DOCUMENTAR
   └── Actualizar docs/sistema/ con funcionalidades completadas
   └── Actualizar AGENTS.md y CLAUDE.md si cambia el estado del proyecto

5. CERRAR FASE
   └── Verificar criterios de éxito del roadmap
   └── Actualizar estado en AGENTS.md, CLAUDE.md y roadmap.md
```

### Checklist Antes de Empezar una Fase Nueva

- [ ] ¿Leí la sección completa de la fase en `roadmap.md`?
- [ ] ¿Entiendo las historias de usuario?
- [ ] ¿Conozco el stack tecnológico específico para esta fase?
- [ ] ¿Existen issues creados para esta fase? Si no, crearlos primero
- [ ] ¿Las dependencias de fases anteriores están completas?

### Jerarquía de Documentos para Desarrollo

| Prioridad | Documento                            | Propósito                        |
| --------- | ------------------------------------ | -------------------------------- |
| 1         | `README.md`                          | Visión, filosofía, principios    |
| 2         | `docs/project-management/roadmap.md` | **Qué construir y en qué orden** |
| 3         | `WORKFLOW.md`                        | Cómo trabajar con issues         |
| 4         | `docs/technical/architecture.md`     | Decisiones técnicas              |
| 5         | `docs/technical/coding-standards.md` | Cómo escribir código             |

### Fases del Roadmap (Resumen)

| Fase | Nombre            | Estado         | Documento             |
| ---- | ----------------- | -------------- | --------------------- |
| 0    | Fundamentos       | ✅ Completado  | `fase-0-tareas.md`    |
| 1    | MVP Cursos        | ✅ Completado  | `roadmap.md` → Fase 1 |
| 2    | Offline-First PWA | ✅ Completado  | `roadmap.md` → Fase 2 |
| 3    | Evaluaciones      | ✅ Completado  | `roadmap.md` → Fase 3 |
| 4    | Módulo Escolar    | ✅ Completado  | `roadmap.md` → Fase 4 |
| 5    | Comunidad         | 🚧 En Progreso | `roadmap.md` → Fase 5 |
| 6-10 | Avanzadas         | 📋 Futuro      | `roadmap.md`          |

---

## Estructura del Proyecto

```
amauta/
├── README.md
├── CLAUDE.md                    # Este archivo - Contexto para Claude
├── WORKFLOW.md                  # ⭐ Metodología de trabajo con issues
├── DEPLOYMENT_PROGRESS.md       # ⭐ Estado del deployment en producción
├── CONTRIBUTING.md              # Guía de contribución
├── CODE_OF_CONDUCT.md           # Código de conducta
├── LICENSE                      # AGPL-3.0
├── .gitignore
│
├── package.json                 # Workspace raíz
├── package-lock.json
├── turbo.json                   # Configuración de Turborepo
│
├── .github/
│   ├── workflows/
│   │   └── ci.yml                    # Workflow de CI/CD
│   ├── README.md                     # Documentación de workflows
│   └── SECURITY_SANITIZATION.md      # Guía de sanitización
│
├── apps/                        # Aplicaciones del monorepo
│   ├── web/                    # Frontend Next.js PWA (@amauta/web)
│   │   ├── package.json
│   │   └── README.md
│   └── api/                    # Backend API REST (@amauta/api)
│       ├── package.json
│       ├── README.md
│       └── prisma/
│           ├── README.md            # ⭐ Documentación de DB y Seed
│           ├── schema.prisma        # Schema de base de datos
│           ├── seed.ts              # Entry point del seed
│           └── seeds/               # Datos de prueba por etapas
│               ├── index.ts         # Orquestador
│               └── usuarios.ts      # Etapa 1: Usuarios (✅ completado)
│
├── packages/                    # Packages compartidos
│   ├── shared/                 # Código compartido (@amauta/shared)
│   │   ├── package.json
│   │   ├── README.md
│   │   └── index.ts
│   └── types/                  # Tipos TypeScript (@amauta/types)
│       ├── package.json
│       ├── README.md
│       └── index.ts
│
└── docs/
    ├── sistema/                         # ⭐ Documentación del sistema (no técnica)
    │   ├── README.md                    # ⭐ Guía general del sistema
    │   ├── etapa-1-usuarios.md          # ✅ Usuarios y perfiles
    │   ├── etapa-2-categorias.md        # ⏳ Categorías e instituciones
    │   ├── etapa-3-cursos.md            # ⏳ Cursos y lecciones
    │   ├── etapa-4-inscripciones.md     # ⏳ Inscripciones y progreso
    │   └── etapa-5-administrativo.md    # ⏳ Asistencias, calificaciones
    ├── project-management/
    │   ├── README.md
    │   ├── sistema-gestion.md         # ⭐ Guía completa del sistema de gestión
    │   ├── metodologia.md
    │   ├── roadmap.md
    │   ├── sprints.md
    │   ├── tareas.md
    │   ├── fase-0-tareas.md
    │   ├── backlog.md
    │   └── project-board.md
    ├── glosario.md                      # Terminología del proyecto
    └── technical/
        ├── README.md
        ├── onboarding.md                  # ⭐ Guía día 1-3 para nuevos devs
        ├── cheatsheet.md                  # Referencia rápida de comandos
        ├── architecture.md
        ├── coding-standards.md
        ├── database.md
        ├── setup.md
        ├── environment-variables.md
        ├── testing.md                     # Guía de testing
        ├── patterns.md                    # Patrones y recetas
        ├── code-review.md                 # Proceso de code review
        ├── debugging.md                   # Guía de debugging
        ├── security-guide.md              # Seguridad para devs (OWASP)
        ├── performance.md                 # Optimización y métricas
        ├── SECURITY_README.md             # Índice de seguridad
        ├── vps-deployment-analysis.md     # Plan de deployment
        ├── PRIVATE_DATA_STORAGE.md        # Almacenamiento seguro
        ├── PRIVATE_REPO_REFERENCE.md      # Repo privado
        └── adr/                           # Decisiones arquitectónicas
            ├── README.md
            ├── 001-monorepo-turborepo.md
            ├── 002-nestjs-fastify.md
            ├── 003-prisma-orm.md
            ├── 004-nextjs-app-router.md
            └── 005-deployment-dokploy.md
```

## Convenciones del Proyecto

### Commits

- Mensajes de commit en español
- Seguir formato descriptivo y claro
- Incluir contexto del cambio

### Documentación

- Toda la documentación en español
- Mantener documentos actualizados en `/docs`
- Separar documentación técnica de gestión de proyecto

## Referencias Importantes

### Documentos Principales

- **Metodología de trabajo**: `WORKFLOW.md` ⭐ **LEER PRIMERO**
- **Estado de Deployment**: `DEPLOYMENT_PROGRESS.md` ⭐ **Estado actual del deployment en producción**
- **Guía de contribución**: `CONTRIBUTING.md` - Cómo contribuir al proyecto
- **Código de conducta**: `CODE_OF_CONDUCT.md` - Expectativas de la comunidad
- **Licencia**: `LICENSE` - AGPL-3.0

### Documentación Técnica

- `docs/technical/README.md` - Índice de documentación técnica
- `docs/technical/architecture.md` - Arquitectura del sistema
- `docs/technical/coding-standards.md` - Estándares de código
- `docs/technical/database.md` - Diseño de base de datos
- `docs/technical/setup.md` - Guía de configuración
- `docs/technical/environment-variables.md` - Estrategia de variables de entorno

#### Base de Datos y Seed ⭐

- `apps/api/prisma/README.md` - ⭐ **Documentación completa de Prisma y Seed**
- `apps/api/prisma/schema.prisma` - Schema de todos los modelos
- `apps/api/prisma/seeds/` - Datos de prueba por etapas

**Usuarios de prueba** (password: `password123`):

- `superadmin@amauta.test` (SUPER_ADMIN)
- `admin1@amauta.test`, `admin2@amauta.test` (ADMIN_ESCUELA)
- `educador1@amauta.test`, `educador2@amauta.test`, `educador3@amauta.test` (EDUCADOR)
- `estudiante1-4@amauta.test` (ESTUDIANTE)

Ver `apps/api/prisma/README.md` para tabla completa con nombres y descripciones.

#### Formación para Desarrolladores ⭐

**Para Empezar (Onboarding)**:

- `docs/technical/onboarding.md` - ⭐ **EMPEZAR AQUÍ** - Guía día 1-3
- `docs/technical/cheatsheet.md` - Referencia rápida de comandos
- `docs/glosario.md` - Terminología del proyecto

**Guías de Desarrollo**:

- `docs/technical/testing.md` - Cómo escribir y ejecutar tests
- `docs/technical/patterns.md` - Patrones y recetas comunes
- `docs/technical/code-review.md` - Proceso de code review
- `docs/technical/debugging.md` - Diagnóstico de problemas
- `docs/technical/security-guide.md` - Seguridad (OWASP Top 10)
- `docs/technical/performance.md` - Optimización y métricas

**Decisiones Arquitectónicas (ADR)**:

- `docs/technical/adr/README.md` - Índice de ADRs
- `docs/technical/adr/001-monorepo-turborepo.md` - Por qué Turborepo
- `docs/technical/adr/002-nestjs-fastify.md` - Por qué NestJS + Fastify
- `docs/technical/adr/003-prisma-orm.md` - Por qué Prisma
- `docs/technical/adr/004-nextjs-app-router.md` - Por qué App Router
- `docs/technical/adr/005-deployment-dokploy.md` - Por qué Dokploy

#### Seguridad y Deployment

- `docs/technical/SECURITY_README.md` - ⭐ Índice maestro de seguridad
- `docs/technical/vps-deployment-analysis.md` - Análisis y plan de deployment
- `docs/technical/PRIVATE_DATA_STORAGE.md` - Guía de almacenamiento seguro
- `docs/technical/PRIVATE_REPO_REFERENCE.md` - Referencia a repositorio privado
- `.github/SECURITY_SANITIZATION.md` - Guía de sanitización de datos sensibles

**⭐ Estado del Deployment: 🟢 COMPLETADO**

- **Frontend**: https://amauta.diazignacio.ar ✅
- **Backend API**: https://amauta-api.diazignacio.ar ✅
- **Servicios**: PostgreSQL, Redis, Backend, Frontend - todos online
- **Detalles**: Ver `DEPLOYMENT_PROGRESS.md`

### Documentación de Gestión

- `docs/project-management/sistema-gestion.md` - ⭐ **Guía completa del sistema de gestión** (empezar aquí)
- `docs/project-management/README.md` - Índice de gestión
- `docs/project-management/roadmap.md` - Roadmap del proyecto
- `docs/project-management/fase-0-tareas.md` - Tareas de Fase 0
- `docs/project-management/metodologia.md` - Metodología ágil
- `docs/project-management/sprints.md` - Gestión de sprints

### CI/CD y Workflows

- `.github/workflows/ci.yml` - Pipeline de CI/CD
- `.github/README.md` - Documentación de workflows

**Nota CI/CD DB (Producción):**

- El contenedor de la API ejecuta `npx prisma migrate deploy` al iniciar.
- El workflow valida el deploy con healthcheck HTTP a `/health`; si falla, el job de deploy falla.

### Monorepo

- `turbo.json` - Configuración de Turborepo
- `apps/web/README.md` - Frontend Next.js PWA
- `apps/api/README.md` - Backend API con NestJS + Fastify
- `packages/shared/README.md` - Código compartido
- `packages/types/README.md` - Tipos TypeScript

## Flujo de Trabajo con Issues

**IMPORTANTE**: Antes de trabajar en cualquier issue, leer `WORKFLOW.md` que contiene:

1. ✅ Proceso completo paso a paso
2. ✅ Cómo usar GitHub CLI para gestionar issues
3. ✅ Formato de commits y mensajes
4. ✅ Uso de TodoWrite para tracking
5. ✅ Checklist de calidad
6. ✅ Ejemplos completos

### Resumen del Flujo

```bash
# 1. Listar issues
gh issue list --limit 100

# 2. Ver detalles
gh issue view <número> --json title,body,labels | jq -r '"\(.title)\n\n\(.body)"'

# 3. Crear todo list (TodoWrite)

# 4. Implementar solución

# 5. Commit con formato estándar
git commit -m "$(cat <<'EOF'
<tipo>: <descripción>
...
Resuelve: #<número>
EOF
)"

# 6. Cerrar issue
gh issue close <número> --comment "✅ Tarea completada..."
```

## Estado Actual del Proyecto

> **Nota**: Esta sección usa comandos dinámicos para evitar desactualización.
> La fuente de verdad es `docs/project-management/backlog.md`.

### Consultar Estado en Tiempo Real

```bash
# Ver todos los issues abiertos
gh issue list --limit 50

# Ver issues por label/fase
gh issue list --label "phase-0"
gh issue list --label "phase-1"

# Ver issues cerrados recientemente
gh issue list --state closed --limit 10

# Ver detalle de un issue específico
gh issue view <número>
```

### Fuentes de Verdad (Documentos Autoritativos)

| Información            | Documento                                    |
| ---------------------- | -------------------------------------------- |
| **Backlog completo**   | `docs/project-management/backlog.md`         |
| **Tareas Fase 0**      | `docs/project-management/fase-0-tareas.md`   |
| **Tablero visual**     | `docs/project-management/project-board.md`   |
| **Roadmap general**    | `docs/project-management/roadmap.md`         |
| **Sistema de gestión** | `docs/project-management/sistema-gestion.md` |
| **Guía del Sistema**   | `docs/sistema/README.md`                     |
| **Base de datos/Seed** | `apps/api/prisma/README.md`                  |
| **Schema Prisma**      | `apps/api/prisma/schema.prisma`              |

### Estado de Producción

| Servicio    | URL                               | Estado    |
| ----------- | --------------------------------- | --------- |
| Frontend    | https://amauta.diazignacio.ar     | 🟢 Online |
| Backend API | https://amauta-api.diazignacio.ar | 🟢 Online |

Ver `DEPLOYMENT_PROGRESS.md` para detalles del deployment.

## Notas para Claude Code

### Reglas de Oro 🏆

1. **Para features nuevas** → Consultar `roadmap.md` PRIMERO
2. **Para issues existentes** → Seguir `WORKFLOW.md`
3. **Antes de codear** → Entender la visión en `README.md`
4. **Al terminar** → Actualizar documentación y estado

### Generales

- **Fase actual**: Fase 5 en progreso (Preparación: F5-001 y F5-002 completados; F5-003 pendiente)
- Usar español para toda la comunicación y documentación
- **SIEMPRE seguir el workflow definido en `WORKFLOW.md`**
- **SIEMPRE consultar `roadmap.md` para desarrollo de features**
- Usar TodoWrite para issues con 3+ pasos
- Commits descriptivos que referencien el issue
- Verificar checklist de calidad antes de cerrar issues

### Estructura del Monorepo

- Usar Turborepo para gestión de workspaces
- Apps en `apps/`: web (Next.js), api (NestJS + Fastify)
- Packages compartidos en `packages/`: shared, types
- Scripts globales en package.json raíz ejecutan en todos los workspaces

### Stack Técnico Definido

- **Frontend**: Next.js 14+ (App Router) con TypeScript
- **Backend**: NestJS + Fastify con TypeScript strict mode
- **ORM**: Prisma
- **Base de Datos**: PostgreSQL 15+
- **Caché**: Redis 7+ (en uso desde Fase 1)
- **Deployment**: Dokploy en VPS (amauta.diazignacio.ar)

Ver `docs/technical/architecture.md` para decisiones técnicas detalladas.

### Entorno de Desarrollo (CRÍTICO) 🚨

**La base de datos está en el VPS, NO local.**

| Servicio        | Ubicación     | URL/Host                          |
| --------------- | ------------- | --------------------------------- |
| **PostgreSQL**  | VPS (Dokploy) | Producción                        |
| **Redis**       | VPS (Dokploy) | Producción                        |
| **Backend API** | VPS           | https://amauta-api.diazignacio.ar |
| **Frontend**    | VPS           | https://amauta.diazignacio.ar     |

**Implicaciones:**

1. Los comandos de Prisma se ejecutan contra la DB de producción
2. **NO hay DB local** por defecto (Docker Compose existe pero no se usa)
3. Cualquier `prisma migrate` afecta producción directamente
4. **SIEMPRE verificar** `prisma migrate status` antes de cambios

**Para migraciones y DB, seguir**: `docs/ai-skills/prisma-db-management.md`

### Orden de Desarrollo

- **Para Fase 0**: Seguir orden numérico de tareas (T-001, T-002...) en `fase-0-tareas.md`
- **Para Fase 1+**: Seguir el orden definido en `roadmap.md` para cada fase
- **Regla general**: Respetar dependencias entre tareas y fases
- **Prioridades dentro de cada fase**: El roadmap define Prioridad 1 (Core), 2 (Importante), 3 (Futuro)

### Documentación del Sistema (IMPORTANTE)

Al completar una etapa o funcionalidad, **SIEMPRE actualizar** la documentación en `docs/sistema/`:

1. **Al completar una etapa del seed**:
   - Actualizar el documento de la etapa (ej: `etapa-1-usuarios.md`)
   - Cambiar estado de ⏳ Pendiente a ✅ Completado
   - Agregar fecha de completitud
   - Documentar qué se logró de manera no técnica

2. **Al agregar funcionalidades**:
   - Actualizar `docs/sistema/README.md` con el nuevo estado
   - Agregar la funcionalidad a la tabla de "Estado Actual"

3. **Estructura de documentación**:

   ```
   docs/sistema/
   ├── README.md              ← Guía general (actualizar siempre)
   ├── etapa-1-usuarios.md    ← ✅ Completado
   ├── etapa-2-categorias.md  ← Actualizar cuando se complete
   ├── etapa-3-cursos.md      ← Actualizar cuando se complete
   ├── etapa-4-inscripciones.md
   └── etapa-5-administrativo.md
   ```

4. **Propósito de esta documentación**:
   - Lectura rápida (~5 minutos)
   - Sin comandos ni código
   - Orientada a entender el sistema
   - Útil para nuevos desarrolladores

- Consultar `docs/project-management/fase-0-tareas.md` para dependencias entre tareas

---

## 🤖 Sistema de Contexto para IA

### Reglas de Carga de Contexto (OBLIGATORIO) ⭐

**ANTES de escribir código, SIEMPRE leer los contextos correspondientes:**

#### Base de Datos y Prisma (CRÍTICO - LEER SIEMPRE) 🚨

**ANTES de escribir CUALQUIER código que interactúe con la base de datos:**

1. **OBLIGATORIO**: Leer `docs/ai-context/database/schema.md` para conocer:
   - Nombres exactos de tablas y columnas
   - Tipos de datos y enums disponibles
   - Relaciones y foreign keys
   - Campos opcionales vs requeridos

2. **OBLIGATORIO**: Leer `apps/api/prisma/schema.prisma` si el contexto no es suficiente

3. **NUNCA inventar**:
   - ❌ Columnas que no existen
   - ❌ Enums con valores no definidos
   - ❌ Relaciones que no están en el schema
   - ❌ Nombres de tablas incorrectos

4. **Verificar antes de usar**:
   - ¿El campo existe? → Leer schema
   - ¿El enum tiene ese valor? → Leer schema
   - ¿La relación existe? → Leer schema
   - ¿Es opcional o requerido? → Leer schema

**Ejemplo de verificación:**

```typescript
// ANTES de escribir esto:
await this.prisma.curso.findMany({
  where: { categoria: 'TECNOLOGIA' }, // ❌ ¿Existe 'categoria'? ¿Es string o relación?
});

// VERIFICAR en schema.prisma:
// - El campo es 'categoriaId' (string, FK) no 'categoria'
// - O usar include: { categoria: true } para la relación

// CORRECTO después de verificar:
await this.prisma.curso.findMany({
  where: { categoriaId: 'xxx' },
  include: { categoria: true },
});
```

#### Backend (API)

| Tarea                               | Leer ANTES de codear                                                     |
| ----------------------------------- | ------------------------------------------------------------------------ |
| Crear/modificar endpoint            | `docs/ai-context/_patterns.md` + `docs/ai-context/modules/{modulo}.md`   |
| Crear módulo CRUD completo          | `docs/ai-skills/crud-generator.md`                                       |
| Agregar endpoint a módulo existente | `docs/ai-skills/api-endpoint.md` + `docs/ai-context/modules/{modulo}.md` |
| Modificar schema Prisma             | `docs/ai-context/database/schema.md`                                     |
| Trabajar con validaciones           | `docs/ai-context/_patterns.md` (sección Zod)                             |
| **Cualquier query Prisma**          | `docs/ai-context/database/schema.md` + `apps/api/prisma/schema.prisma`   |

#### Frontend (Web)

| Tarea                         | Leer ANTES de codear                                                      |
| ----------------------------- | ------------------------------------------------------------------------- |
| Crear formulario              | `docs/ai-skills/react-form.md` + `docs/ai-context/frontend/components.md` |
| Crear página nueva            | `docs/ai-context/frontend/pages.md`                                       |
| Usar hooks de auth/roles      | `docs/ai-context/frontend/hooks.md`                                       |
| Crear componente reutilizable | `docs/ai-context/frontend/components.md`                                  |

#### Reglas Generales

1. **CRÍTICO: Nunca escribir queries Prisma sin verificar el schema** - Leer `docs/ai-context/database/schema.md` o `apps/api/prisma/schema.prisma` ANTES de cualquier operación de DB
2. **Nunca escribir código backend sin leer `_patterns.md`** - Contiene validación Zod, estructura de controllers/services, paginación
3. **Para módulos existentes, leer su contexto** - `docs/ai-context/modules/{modulo}.md` tiene endpoints, modelo Prisma y ejemplos reales
4. **Usar skills para generación repetitiva** - CRUD, endpoints, formularios ya tienen templates
5. **Si no existe contexto del módulo, usar `cursos.md` como referencia** - Es el módulo más completo y sirve de template
6. **Ante la duda, leer el código fuente** - Si el contexto no es claro, leer el archivo real antes de asumir

### Patrones Críticos (Resumen)

Estos patrones están en `_patterns.md` pero son tan importantes que se resumen aquí:

```typescript
// ❌ NUNCA usar parse directo
const data = schema.parse(dto); // MAL

// ✅ SIEMPRE usar safeParse
const result = schema.safeParse(dto);
if (!result.success) {
  throw new BadRequestException(result.error.issues[0]?.message);
}
const data = result.data; // BIEN
```

```typescript
// ❌ NUNCA eliminar físicamente
await this.prisma.curso.delete({ where: { id } }); // MAL

// ✅ SIEMPRE soft delete (cambiar estado)
await this.prisma.curso.update({
  where: { id },
  data: { estado: 'ARCHIVADO' },
}); // BIEN
```

```typescript
// Estructura de respuestas
// Singular: { curso, message }
// Lista: { cursos, total, page, limit, totalPages }
```

### Enums Disponibles (Referencia Rápida)

**IMPORTANTE**: Solo usar estos valores. No inventar otros.

```typescript
// Roles de usuario
enum Rol {
  ESTUDIANTE,
  EDUCADOR,
  ADMIN_ESCUELA,
  SUPER_ADMIN,
}

// Cursos
enum Nivel {
  PRINCIPIANTE,
  INTERMEDIO,
  AVANZADO,
}
enum EstadoCurso {
  BORRADOR,
  REVISION,
  PUBLICADO,
  ARCHIVADO,
}

// Lecciones
enum TipoLeccion {
  VIDEO,
  TEXTO,
  QUIZ,
  INTERACTIVO,
  DESCARGABLE,
}

// Inscripciones
enum EstadoInscripcion {
  ACTIVO,
  COMPLETADO,
  ABANDONADO,
}

// Instituciones
enum TipoInstitucion {
  ESCUELA,
  COLEGIO,
  UNIVERSIDAD,
  CENTRO_FORMACION,
}

// Asistencias
enum EstadoAsistencia {
  PRESENTE,
  AUSENTE,
  TARDANZA,
  JUSTIFICADO,
}

// Comunicados
enum TipoComunicado {
  GENERAL,
  ACADEMICO,
  ADMINISTRATIVO,
  EVENTO,
  URGENTE,
}
enum Prioridad {
  BAJA,
  NORMAL,
  ALTA,
  URGENTE,
}
```

### Documentación Disponible

| Tipo         | Ubicación                            | Propósito                       |
| ------------ | ------------------------------------ | ------------------------------- |
| **Patrones** | `docs/ai-context/_patterns.md`       | Patrones de código del proyecto |
| **Módulos**  | `docs/ai-context/modules/`           | Contexto por módulo backend     |
| **Frontend** | `docs/ai-context/frontend/`          | Páginas, componentes, hooks     |
| **Database** | `docs/ai-context/database/schema.md` | Schema Prisma                   |
| **Skills**   | `docs/ai-skills/`                    | Generadores de código           |

### Módulos Documentados

- `auth` - Autenticación (NextAuth + JWT)
- `cursos` - CRUD de cursos (⭐ usar como template de referencia)
- `lecciones` - Gestión de lecciones
- `inscripciones` - Sistema de inscripción
- `uploads` - Subida de archivos
- `categorias` - Categorías de cursos

### Skills Disponibles

| Skill                     | Archivo                                   | Cuándo usar                                                                            |
| ------------------------- | ----------------------------------------- | -------------------------------------------------------------------------------------- |
| **Prisma & DB**           | `docs/ai-skills/prisma-db-management.md`  | Migraciones, verificar DB, resolver errores                                            |
| **CRUD Generator**        | `docs/ai-skills/crud-generator.md`        | Crear módulo nuevo completo                                                            |
| **API Endpoint**          | `docs/ai-skills/api-endpoint.md`          | Agregar endpoint a módulo existente                                                    |
| **React Form**            | `docs/ai-skills/react-form.md`            | Crear formulario nuevo                                                                 |
| **Complete Issue**        | `docs/ai-skills/complete-issue.md`        | Ejecutar un issue completo de forma autónoma                                           |
| **Performance Review**    | `docs/ai-skills/performance-review.md`    | Analizar performance y generar informe con mejoras                                     |
| **Security Audit**        | `docs/ai-skills/security-audit.md`        | Auditar vulnerabilidades de seguridad (OWASP Top 10)                                   |
| **Fix Security Findings** | `docs/ai-skills/fix-security-findings.md` | Aplicar fixes de un informe de auditoría previo (contexto limpio, build verificado)    |
| **NotebookLM Cuadernos**  | `docs/ai-skills/notebooklm-cuadernos.md`  | Generar cuadernos de estudio genéricos para NotebookLM                                 |
| **PWA Mobile Design**     | `docs/ai-skills/pwa-mobile-design.md`     | Diseñar/auditar PWA orientada a mobile: manifest, SW, IndexedDB, sync, UI táctil       |
| **Feature Audit**         | `docs/ai-skills/feature-audit.md`         | Auditar que las features implementadas funcionan y cumplen sus criterios de aceptación |
| **Functional Docs**       | `docs/ai-skills/functional-docs.md`       | Generar documentación funcional de módulos para usuarios finales (no técnica)          |
| **AI Context Validator**  | `docs/ai-skills/ai-context-validator.md`  | Verificar que docs/ai-context está sincronizado con el código real                     |
| **Issue Inspector**       | `docs/ai-skills/issue-inspector.md`       | Auditar issue: verifica requisitos, tests, y funcionalidad en producción               |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/informaticadiaz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
