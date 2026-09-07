## gestion-academica

> Documento de gobernanza técnica. Todo agente que interactúe con este repositorio

# AGENTS.md — G_ACADEMICA

Documento de gobernanza técnica. Todo agente que interactúe con este repositorio
debe leerlo y seguirlo antes de proponer o implementar cambios.

---

## 1. Identidad del proyecto

| Atributo | Valor |
|---|---|
| **Nombre** | G_ACADEMICA |
| **Dominio** | Gestión académica (estudiantes, docentes, cursos, calificaciones, inscripciones, planes de estudio) |
| **Usuarios objetivo** | Administradores académicos, docentes, estudiantes, personal de secretaría |
| **Alcance** | Plataforma para administrar el ciclo de vida académico: inscripción de estudiantes, asignación de docentes, creación y gestión de cursos, registro de calificaciones, generación de reportes, control de planes de estudio |
| **Estado actual** | Proyecto greenfield — sin código, sin dependencias, sin historia. Cada decisión fundacional debe documentarse como ADR. |

El agente debe entender que **todas las decisiones técnicas deben servir al dominio académico**. No se introducen tecnologías o patrones por moda: cada elección debe justificarse frente a un problema real del negocio.

---

## 2. Stack tecnológico

> Este stack es **propuesto por defecto** hasta que existan archivos de configuración
> que lo contradigan. El agente debe leer `package.json`, `tsconfig*`, `composer.json`,
> `Cargo.toml` o equivalentes antes de asumir el stack.

| Tecnología | Rol | Motivación |
|---|---|---|
| **Node.js + TypeScript** | Runtime y lenguaje | Tipado estático para reducir errores en dominio complejo; ecosistema maduro para APIs; las interfaces y tipos de TS permiten aplicar DIP e ISP con naturalidad |
| **Express / Fastify** | Framework HTTP | Ligero, extensible con plugins, sin opiniones arquitectónicas fuertes que choquen con Clean Architecture |
| **PostgreSQL** | Base de datos relacional | El dominio académico exige integridad referencial (estudiantes ↔ inscripciones ↔ cursos ↔ calificaciones); las relaciones many-to-many y las restricciones CHECK son esenciales |
| **Prisma / Drizzle** | ORM / query builder | Tipado seguro desde la BD hasta la aplicación; migrations como código; evita el anemic domain model al separar persistencia de dominio |
| **Vitest** | Testing | Rápido, compatible con TypeScript nativo, soporte para cobertura y mocks |
| **Docker** | Entorno de desarrollo y despliegue | Reproducibilidad; evita el "en mi máquina funciona"; permite levantar PostgreSQL sin instalación manual |

Cualquier cambio en el stack debe documentarse como ADR en `/docs/decisions/`.

---

## 3. Arquitectura del sistema

Se adopta **Clean Architecture** (también conocida como arquitectura hexagonal o de puertos y adaptadores) por las siguientes razones del dominio:

- El negocio académico tiene reglas complejas (cálculo de promedios, prerequisitos, choques de horario) que **no deben depender de frameworks ni de la base de datos**.
- Los casos de uso deben ser testeables sin infraestructura.
- La UI puede cambiar (web, mobile, API REST, GraphQL) sin afectar el núcleo.

### Diagrama de capas

```
┌─────────────────────────────────────────────────────────┐
│                    Infraestructura                        │
│  (Express, Prisma, JWT, Nodemailer, AWS S3, logger...)  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │              Presentation / API                   │   │
│  │  (controllers, middleware, serializers, routes)   │   │
│  └──────────────────┬───────────────────────────────┘   │
│                     │                                    │
│  ┌──────────────────▼───────────────────────────────┐   │
│  │              Application / Use Cases              │   │
│  │  (orquestar flujos, coordinar repos y servicios)  │   │
│  └──────────────────┬───────────────────────────────┘   │
│                     │                                    │
│  ┌──────────────────▼───────────────────────────────┐   │
│  │                   Domain                           │   │
│  │  (entidades, value objects, domain services,      │   │
│  │   repository interfaces, domain events)            │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Reglas de dependencia

- **Domain**: NO depende de ninguna otra capa. Es el centro del sistema.
- **Application**: depende de Domain. NO depende de infraestructura.
- **Infrastructure**: depende de Domain y Application. Implementa interfaces definidas en Domain.
- **Presentation**: depende de Application. Conecta el mundo exterior con los casos de uso.

La lógica de negocio vive exclusivamente en **Domain** (entidades, value objects, domain services) y **Application** (casos de uso). Nunca en infraestructura o presentación.

### Estructura de carpetas esperada

```
src/
├── domain/
│   ├── entities/
│   ├── value-objects/
│   ├── services/
│   ├── repositories/  (interfaces)
│   └── events/
├── application/
│   ├── use-cases/
│   ├── dtos/
│   └── ports/         (interfaces de salida)
├── infrastructure/
│   ├── persistence/   (implementaciones de repositorios)
│   ├── auth/
│   ├── email/
│   ├── logging/
│   └── config/
├── presentation/
│   ├── controllers/
│   ├── middleware/
│   ├── serializers/
│   └── routes/
└── shared/
    ├── errors/
    └── utils/
```

Si el código existente no sigue esta estructura, el agente debe proponer una migración progresiva documentada en ADRs.

---

## 4. Principios y estándares de código

### SOLID

| Principio | Cómo se aplica en el proyecto |
|---|---|
| **SRP — Single Responsibility** | Cada clase tiene una sola razón de cambio. Un caso de uso hace una sola cosa. Un controlador no mezcla lógica de negocio con serialización. |
| **OCP — Open/Closed** | Las abstracciones (interfaces) permiten extender comportamiento sin modificar código existente. Ej: un nuevo método de calificación implementa `GradingStrategy` sin tocar las entidades. |
| **LSP — Liskov Substitution** | Cualquier implementación de una interfaz debe poder sustituir a otra sin romper el sistema. Ej: `PostgresStudentRepository` e `InMemoryStudentRepository` son intercambiables en tests. |
| **ISP — Interface Segregation** | Interfaces pequeñas y específicas por rol. No existe `UserRepository` con 20 métodos; existen `StudentRepository`, `TeacherRepository`, `AdminRepository` con los métodos que cada rol necesita. |
| **DIP — Dependency Inversion** | Las capas altas definen interfaces; las capas bajas las implementan. El dominio nunca importa nada de infraestructura. La inyección de dependencias se configura en la entrada de la aplicación. |

### Clean Code

- **Nombres que revelan intención**: `calculateFinalGrade(studentId, courseId)` no `proc1(a, b)`.
- **Funciones pequeñas**: una función hace una cosa, con un solo nivel de abstracción. Máximo ~20 líneas.
- **Sin comentarios que expliquen código mal escrito**: si necesita comentario, refactoriza. Solo se permiten comentarios JSDoc en interfaces públicas y ADRs en decisiones técnicas.
- **Sin números mágicos**: toda constante tiene nombre. `const MAX_PREREQUISITES = 5` no `if (count > 5)`.
- **Sin flags booleanos como parámetro**: partir la función en dos. `registerStudent(data)` y `registerGuest(data)` en lugar de `register(data, { isGuest: true })`.

### Convenciones del proyecto

> El agente debe inferir y actualizar esta sección automáticamente cada vez que explore
> el código existente. Si hay discrepancias entre lo documentado aquí y el código real,
> el código tiene la razón y esta sección debe actualizarse.

Convenciones por defecto (hasta que el código diga lo contrario):

- **Nomenclatura de archivos**: `kebab-case` para archivos (ej: `calculate-final-grade.use-case.ts`), `PascalCase` para clases/interfaces/tipos, `camelCase` para funciones/variables/métodos.
- **Named exports**: siempre exportar por nombre, no `export default`. Facilita refactors y autoimports.
- **TypeScript strict mode**: activar `strict: true` en `tsconfig.json`. No usar `any`.
- **Tests**: archivos `*.test.ts` o `*.spec.ts` junto al archivo que prueban.
- **Manejo de errores**: usar excepciones con `AppError` y sus subtipos (`ValidationError`, `NotFoundError`, `ConflictError`, `UnauthorizedError`, `ForbiddenError`). Los casos de uso lanzan `AppError` y un middleware central las transforma en respuestas HTTP. Ver ADR-007 para la justificación.

---

## 5. Patrones de diseño permitidos y preferidos

El agente puede proponer cualquiera de estos patrones, pero debe **justificar la elección en el contexto del dominio académico**.

### Creacionales

| Patrón | Cuándo usarlo |
|---|---|
| **Factory / Abstract Factory** | Para instanciar entidades con validaciones complejas (ej: `CourseFactory.createFromPlan(planData)` que aplica reglas de prerrequisitos al construir). |
| **Builder** | Para objetos con muchas configuraciones opcionales (ej: `new ReportBuilder().withGrades().withAttendance().withPeriod(semester).build()`). |

### Estructurales

| Patrón | Cuándo usarlo |
|---|---|
| **Repository** | **Obligatorio**. Toda interacción con persistencia pasa por una interfaz definida en Domain y una implementación en Infrastructure. Permite testear casos de uso con repositorios in-memory. |
| **Adapter** | Para envolver servicios externos (email, S3, pasarela de pagos, LDAP) detrás de interfaces del dominio. Ej: `EmailSender` es una interfaz en Domain; `SmtpEmailAdapter` es la implementación en Infrastructure. |

### Comportamiento

| Patrón | Cuándo usarlo |
|---|---|
| **Strategy** | Para algoritmos intercambiables. Ej: `GradingStrategy` con implementaciones `WeightedAverageStrategy`, `PassFailStrategy`, `CurveGradingStrategy`. |
| **Observer / Domain Events** | Para reaccionar a cambios del dominio sin acoplar. Ej: cuando `StudentEnrolled` se emite, un suscriptor envía email de bienvenida y otro actualiza el cupo del curso. |

El agente debe incluir en la justificación: (1) qué problema del dominio resuelve, (2) cómo mejora los atributos de calidad, (3) qué principio SOLID aplica o refuerza.

---

## 6. Antipatrones prohibidos

El agente debe detectar y rechazar explícitamente cualquiera de estos:

| Antipatrón | Por qué está prohibido |
|---|---|
| **God Object** | Clases que saben o hacen demasiado (ej: `UniversityService` con 50 métodos). Viola SRP y dificulta testear, mantener y escalar. |
| **Anemic Domain Model** | Entidades sin comportamiento, solo getters/setters. La lógica termina en servicios procedurales. El dominio debe encapsular reglas. |
| **Lógica de negocio en controladores** | Los controladores solo reciben requests, llaman casos de uso y devuelven respuestas. Cualquier `if` con regla de negocio es una violación. |
| **Dependencias circulares** | Módulo A depende de B y B depende de A. El compilador/linter debe rechazarlo; si no es posible, el agente debe rediseñar las abstracciones. |
| **Magic strings / numbers** | `if (role === 'admin')` debe ser `if (role === UserRole.ADMIN)`. Números literales sin constante nombrada están prohibidos. |
| **Consultas BD fuera del repositorio** | Ninguna capa fuera de Infrastructure/Persistence puede tener SQL, llamadas ORM ni conexiones. |
| **`any` en TypeScript** | Usar `unknown` y type guards si el tipo no se conoce en tiempo de compilación. `any` desactiva el verificador de tipos. |
| **Catch silencioso** | `catch (e) {}` o `catch (e) { console.error(e) }` sin relanzar o manejar el error. Todo error debe registrarse y manejarse (reintentar, fallar gracefulmente, o propagar). |

---

## 7. Atributos de calidad (Quality Attributes)

Toda decisión técnica debe evaluarse contra estos atributos. El agente debe explicar explícitamente cómo su propuesta impacta cada uno.

| Atributo | Definición | Cómo se mide/práctica |
|---|---|---|
| **Mantenibilidad** | El código debe poder modificarse sin efectos colaterales inesperados | Bajo acoplamiento, alta cohesión, tests que cubren contratos públicos |
| **Testabilidad** | La lógica de negocio debe ser testeable sin infraestructura real | Casos de uso y entidades se testean con mocks/stubs in-memory; pruebas de integración para repositorios con testcontainers o BD real |
| **Escalabilidad** | Los módulos deben crecer independientemente | Separación por contextos acotados (bounded contexts); las capas permiten escalar verticalmente (más instancias) y horizontalmente (dividir contexto) |
| **Seguridad** | Validación en cada entrada, autorización en cada operación, sin exponer datos sensibles | Input validation con Zod/Joi en la capa de presentación; autorización con policies en casos de uso; PII en tránsito y reposo; sin secrets en código ni logs |
| **Observabilidad** | El sistema debe poder entenderse en producción sin depurar | Logs estructurados (JSON), IDs de correlación por request, métricas (tiempos de respuesta, tasas de error, throughput), trazabilidad de errores con stack trace y contexto |

Ejemplo de evaluación: *"Usar Prisma como ORM mejora la mantenibilidad (tipado generado) y testabilidad (fácil de mockear), pero introduce acoplamiento a Infrastructure en las migrations — lo mitigamos generando tipos compartidos y manteniendo las interfaces de repositorio en Domain."*

---

## 8. ADRs — Architecture Decision Records

Las decisiones arquitectónicas significativas se registran en `/docs/decisions/`.

### Cuándo crear un ADR

- Creación o modificación de la estructura de carpetas raíz
- Elección o cambio de tecnología en el stack
- Introducción de un nuevo patrón de diseño (más allá de los listados en la sección 5)
- Cambios en autenticación, autorización, manejo de errores global, logging, o despliegue
- Cualquier decisión que tenga impacto en más de un módulo o capa

### Formato

```markdown
# ADR-[número] — [Título]

**Estado**: Propuesto | Aceptado | Deprecado | Reemplazado por ADR-XXX
**Fecha**: YYYY-MM-DD
**Contexto**: situación o problema que motivó esta decisión
**Opciones consideradas**: alternativas evaluadas y por qué se descartaron
**Decisión**: qué se decidió hacer
**Consecuencias**: impacto positivo y negativo de la decisión
**Atributos de calidad afectados**: cuáles mejoran, cuáles se sacrifican y cómo se mitiga
```

---

## 9. Proceso de toma de decisiones

Cuando el agente enfrente una decisión técnica no trivial, debe seguir este proceso secuencial:

```
1. IDENTIFICAR → definir el problema real sin asumir solución
2. INVESTIGAR → listar ≥2 alternativas viables con sus trade-offs
3. EVALUAR → valorar cada alternativa contra los atributos de calidad (sección 7)
4. DECIDIR → elegir la opción con mejor balance para el contexto actual
5. DOCUMENTAR → crear ADR en /docs/decisions/
6. IMPLEMENTAR → codificar usando el patrón más apropiado (sección 5)
7. VERIFICAR → comprobar que no se violan principios SOLID ni se introducen antipatrones (sección 6)
```

Este proceso aplica incluso cuando el agente propone una solución que parece obvia. La documentación (paso 5) puede omitirse si la decisión ya fue registrada en un ADR previo.

---

## 10. Comandos del proyecto

> Esta sección debe actualizarse automáticamente cada vez que el agente descubra
> scripts en `package.json`, `Makefile`, `Taskfile.yml` o archivos similares.

### Desarrollo

```bash
# Instalar dependencias (backend)
npm install

# Instalar dependencias (frontend)
npm install --prefix frontend

# Iniciar backend en modo desarrollo (con hot-reload) — Express API en :3000
npm run dev

# Iniciar frontend en modo desarrollo — Next.js App en :3001
npm run dev:frontend

# Compilar TypeScript a dist/
npm run build

# Iniciar en producción
npm start
```

### Testing — Backend

```bash
# Tests unitarios (Vitest)
npm run test

# Tests en modo watch
npm run test:watch

# Tests con cobertura
npm run test:coverage

# Smoke tests locales (reinicia servidor automáticamente para evitar rate limit)
npm run test:smoke:local

# Smoke tests contra Docker (requiere backend-test en :3002)
npm run test:smoke:local:docker

# Smoke tests (Docker Compose — levanta BD + backend + tests)
npm run test:smoke

# Tests de integración (levanta Docker Compose con BD + backend + smoke tests)
npm run test:integration
```

> ⚠️ `test:smoke:local` ejecuta `scripts/smoke-test-local.sh` que **mata el servidor anterior** en el puerto, inicia uno nuevo, corre los tests, y lo detiene. Esto evita que el rate limiter de login (10 req/15min) se acumule entre ejecuciones y cause falsos 429.

### Testing — Frontend

```bash
# Tests unitarios y de componentes (Vitest + Testing Library + jsdom)
npm run test --prefix frontend

# Tests en modo watch
npm run test:watch --prefix frontend
```

Los tests de frontend usan `@testing-library/react` para renderizar componentes y `jsdom` como entorno DOM. Los mocks de `next/navigation` y `@/lib/auth` se configuran globalmente en `frontend/src/test-setup.tsx`. Los tests se colocalizan junto a los archivos que prueban con extensión `.test.tsx`.

### Base de datos (Prisma)

```bash
# Generar Prisma Client tras cambios en schema
npm run db:generate

# Crear y aplicar migración en desarrollo
npm run db:migrate

# Aplicar migraciones en producción
npm run db:migrate:deploy

# Ejecutar seed de datos de prueba
npm run db:seed

# Abrir Prisma Studio (UI para explorar datos)
npm run db:studio

# Resetear BD (borra datos, reapplica migraciones y seed)
npm run db:reset
```

### Calidad

```bash
# Linter
npm run lint

# TypeScript check (typecheck)
npm run typecheck

# Todo en orden (lint + typecheck + test)
npm run check
```

### Docker

```bash
# Construir imágenes
npm run docker:build

# Levantar stack completo (postgres + backend + frontend)
npm run docker:up

# Ver logs
docker compose logs -f

# Detener entorno
npm run docker:down

# Ejecutar tests de integración (full stack: BD + backend + smoke tests)
npm run test:integration

# Ejecutar tests E2E con Docker Compose
npm run test:integration:e2e

# O manualmente con BD aislada para tests unitarios
docker compose -f docker-compose.test.yml up -d postgres-test
DATABASE_URL=postgresql://postgres:postgres@localhost:5433/g_academica_test npm run test
docker compose -f docker-compose.test.yml down
```

---

**Este documento es vivo.** El agente debe actualizarlo cuando descubra nuevas
convenciones en el código, tecnologías no documentadas, o comandos que no estén
listados aquí. Todas las actualizaciones deben ser coherentes con el estado real
del repositorio.

---
> Source: [cj200xyz/Gestion_Academica](https://github.com/cj200xyz/Gestion_Academica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
