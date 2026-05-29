## 42jobs

> Este es un proyecto **personal de aprendizaje**. El objetivo principal no es entregar rápido, sino **entender cada línea de código** que se escribe y cada decisión que se toma.

# AGENTS.md — 42jobs

## Propósito del proyecto

Este es un proyecto **personal de aprendizaje**. El objetivo principal no es entregar rápido, sino **entender cada línea de código** que se escribe y cada decisión que se toma.

## Reglas de oro para la IA

1. **Nunca hacer cambios grandes de golpe.** Todo cambio debe ser pequeño, atómico y explicado.
2. **Preguntar antes de implementar.** Antes de escribir una sola línea de código, la IA debe explicar qué va a hacer, cómo lo va a hacer y por qué. El usuario debe dar el visto bueno.
3. **Explicar cada cambio.** Después de cada modificación, la IA debe explicar qué se ha hecho de forma clara y concisa.
4. **El usuario debe entenderlo todo.** Si algo es complejo, se desglosa. Si el usuario no entiende algo, la IA debe ser capaz de explicarlo con otros ejemplos o analogías.
5. **Nada de código mágico ni patrones oscuros.** Código limpio, legible, bien estructurado y comentado solo cuando sea necesario para clarificar algo no obvio.
6. **Siempre consultar AGENTS.md y roadmap.md** al comenzar una sesión para saber en qué punto del proyecto estamos.

## Stack tecnológico

| Capa | Tecnología | Detalles |
|------|-----------|----------|
| Backend | .NET 10 (ASP.NET Core) | Web API MVC, C#, EF Core, JWT |
| Base de datos | PostgreSQL 16 | Migraciones SQL en `database/migrations/` |
| Frontend | React + React Router (Vite) + TypeScript | Sin framework CSS, estilos propios |
| Package manager | pnpm | Más seguro que npm, estricto en dependencias |
| Infraestructura | Docker + Docker Compose | Dev y prod con override files |
| APIs externas | LinkedIn RapidAPI, Google Gemini / OpenAI / DeepSeek | Para búsqueda de empleos, filtrado IA y generación de CV |

## Estructura del proyecto

```
42jobs/
├── AGENTS.md              ← Este archivo
├── roadmap.md             ← Punto actual del proyecto y siguientes pasos
├── Makefile               ← Orquestación (dev-up, prod-up, etc.)
├── docker-compose.yml     ← Base (db, backend, frontend)
├── docker-compose.override.yml ← Overrides de desarrollo
├── docker-compose.prod.yml     ← Overrides de producción
├── backend/
│   ├── Dockerfile
│   └── src/
│       ├── src.csproj              ← Proyecto .NET 10 con EF Core, Npgsql, JWT, BCrypt
│       ├── Program.cs              ← Entry point (JWT, DbContext, servicios, JSON snake_case)
│       ├── appsettings.json
│       ├── Controllers/            ← 14 controladores (partial classes, 78 archivos de endpoints)
│       ├── Data/AppDbContext.cs    ← EF Core DbContext (Fluent API, 21 entidades)
│       ├── Models/                 ← 21 modelos C# + 7 DTOs
│       ├── Services/               ← IAiService, JobFetchService, GithubImportService, AiReadinessService, JWT, Encryption, AdminLog
│       └── Utils/                  ← DatabaseUrlParser
├── database/
│   └── migrations/            ← 32 archivos SQL (categorías, keywords, jobs, perfil, user_categories...)
├── frontend/
│   ├── Dockerfile              ← Multi-stage: dev (vite) + prod (nginx + build)
│   ├── nginx.conf              ← Proxy /api -> backend, sirve estáticos de dist/
│   ├── package.json            ← react, react-router-dom, vite, chart.js
│   ├── pnpm-lock.yaml
│   ├── tsconfig.json           ← Config TypeScript para src/
│   ├── tsconfig.node.json      ← Config TypeScript para vite.config.ts
│   ├── vite.config.ts          ← Dev server con proxy /api, build output a dist/
│   ├── index.html              ← Entry point Vite
│   ├── public/
│   │   └── resources/          ← Imágenes y estáticos
│   └── src/
│       ├── main.tsx            ← Entry point (RouterProvider)
│       ├── router.tsx           ← createBrowserRouter con loaders/actions
│       ├── index.css            ← Importa los 13 módulos CSS de styles/
│       ├── types/               ← Tipos compartidos (User, Job, ProfileData...)
│       │   └── index.ts
│       ├── utils/               ← Funciones utilitarias (api, format, match)
│       │   ├── index.ts         ← Barrel
│       │   ├── api.ts
│       │   ├── format.ts
│       │   └── match.ts
│       ├── hooks/               ← Custom hooks compartidos
│       │   ├── index.ts         ← Barrel
│       │   └── useDebounce.ts   ← useDebounce, usePolling
│       ├── context/             ← React contexts (Auth, Toast)
│       │   ├── index.ts         ← Barrel
│       │   ├── AuthContext.tsx
│       │   └── ToastContext.tsx
│       ├── styles/              ← 13 módulos CSS por responsabilidad
│       │   ├── variables.css, base.css, layout.css, cards.css
│       │   ├── modals.css, forms.css, chart.css
│       │   ├── profile.css, cv.css, toast.css
│       │   ├── auth.css, admin.css, utilities.css
│       ├── components/          ← Componentes con barrel por dominio
│       │   ├── index.ts         ← Barrel raíz
│       │   ├── auth/            ← RequireAuth
│       │   ├── categories/      ← CategoriesBar, AddCategoryDialog
│       │   ├── jobs/            ← NotesModal, CvModal
│       │   ├── keywords/        ← KeywordsChart, KeywordTag, KeywordModal
│       │   ├── layout/          ← AuthLayout, AdminLayout, FreeTierBanner
│       │   ├── profile/         ← ProfileInfo, ProfileList, ProfileProjects, LinkedInImport
│       │   └── ui/              ← ToastContainer, AiNotConfiguredModal
│       └── pages/               ← Páginas organizadas por dominio con barrels
│           ├── index.ts         ← Barrel raíz
│           ├── auth/            ← Login, Register (+ login.action.ts, register.action.ts)
│           ├── dashboard/       ← Dashboard, Offers, Tracking, KeywordsPage
│           │   ├── index.ts
│           │   ├── Dashboard.tsx + dashboard.loader.ts + dashboard.types.ts
│           │   ├── Offers.tsx + offers.loader.ts
│           │   ├── Tracking.tsx + tracking.loader.ts + tracking.types.ts
│           │   └── KeywordsPage.tsx + keywordsPage.loader.ts
│           ├── profile/         ← Profile.tsx + profile.loader.ts
│           └── admin/           ← 8 páginas admin + barrel
└── examples/                  ← Ejemplos de respuestas de API de LinkedIn
```

## Estado actual del proyecto

1. **Base de datos:** ✅ Migraciones SQL con 32 archivos. Tablas: categorías, empresas, keywords, jobs, perfil de usuario, idiomas, certificaciones, educación, proyectos, experiencias, user_providers, user_jobs, user_categories, resumes, cv_templates, ai_services, ai_models, ai_prompts, admin_logs y tablas M2M (job_keywords, project_keywords, work_experience_keywords).
2. **Backend:** ✅ Funcional. 14 controladores REST con autenticación JWT vía cookie (`42jobs_auth`). EF Core con snake_case naming convention. Servicios: LinkedIn RapidAPI, Gemini/OpenAI/DeepSeek (filtro + keywords + CV), background job queue con Channel<T> para fetch de trabajos con scheduler automático (00:00 UTC). Pipeline de reintentos y validación de readiness.
3. **Frontend:** ✅ Funcional. React 19 + React Router 7 con Vite. Data router (`createBrowserRouter`) con loaders y actions. Barrel pattern en todas las carpetas. CSS modular (13 archivos). Hooks y utils compartidos. Tipos centralizados. Modal AiNotConfiguredModal para errores de readiness.

## Próximos pasos (visión general)

1. ✅ Backend .NET con API MVC funcional (controladores, modelos, servicios).
2. ✅ Conexión a PostgreSQL vía Entity Framework Core.
3. ✅ Autenticación de usuarios (JWT + cookies).
4. ✅ Endpoints del frontend (todos implementados).
5. ✅ React + React Router + Vite. Frontend completo.
6. ✅ APIs externas (LinkedIn RapidAPI + Gemini/OpenAI/DeepSeek).
7. ✅ Frontend refactorizado (barrel pattern, CSS modules, data router, loaders/actions).
8. ✅ Auto-update automático con scheduler, reintentos, validación de readiness.
9. ⬚ Notificaciones por email de nuevos jobs.
10. ⬚ Más providers de jobs (InfoJobs, Indeed).
11. ⬚ Tests de UI con Playwright.

## Releases y deploy

Las releases usan **versionado semántico** con tags `v{major}.{minor}.{patch}-alpha` (ej. `v0.1.4-alpha`).

Para crear una nueva release:

```bash
# Automático: detecta el último tag, incrementa el patch y hace push
make release

# O manualmente:
git push origin master
git tag v0.1.4-alpha
git push origin v0.1.4-alpha
```

El script `scripts/release.sh` se encarga de detectar el último tag `v*`, parsearlo (`vX.Y.Z-alpha`) e incrementar en +1 el patch automáticamente.

### Qué sucede al pushear un tag

El [release workflow](.github/workflows/release.yml) se dispara con cualquier push de tag `v*`:

1. **Build** — Se construyen las imágenes Docker de backend y frontend y se suben a GitHub Container Registry (`ghcr.io`) con el tag de versión y `:latest`
2. **GitHub Release** — Se genera un release automático a partir del historial de commits
3. **Deploy** — El VPS hace pull del código, pull de las nuevas imágenes, y reinicia los contenedores

```
git tag vX.Y.Z-alpha  ──>  GitHub Actions  ──>  ghcr.io images  ──>  VPS deploy
```

> Solo los commits en `master` deben taggearse. El tag debe empezar con `v` para disparar el pipeline.

## Cómo trabajar

- Al iniciar una tarea, **leer `roadmap.md`** para saber en qué checkpoint estamos.
- Proponer el siguiente paso pequeño al usuario.
- Esperar confirmación antes de tocar código.
- Al completar un paso, actualizar `roadmap.md` para reflejar el progreso.
- Usar `Makefile` para levantar/bajar la infraestructura Docker (`make dev-up`, `make dev-down`).
- Para instalar dependencias del frontend, usar siempre `pnpm install` (nunca `npm install`).

---
> Source: [samuelhm/42Jobs](https://github.com/samuelhm/42Jobs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
