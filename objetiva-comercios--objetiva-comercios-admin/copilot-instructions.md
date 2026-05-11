## objetiva-comercios-admin

> Monorepo pnpm + Turborepo: `apps/backend` (NestJS), `apps/web` (Next.js 14), `apps/mobile` (Vite + Capacitor). Auth via Supabase JWT, datos en PostgreSQL con Drizzle ORM. UI con shadcn/ui + estetica Tabler.

# CLAUDE.md — Objetiva Comercios Admin

## Proyecto

Monorepo pnpm + Turborepo: `apps/backend` (NestJS), `apps/web` (Next.js 14), `apps/mobile` (Vite + Capacitor). Auth via Supabase JWT, datos en PostgreSQL con Drizzle ORM. UI con shadcn/ui + estetica Tabler.

## Regla principal

Antes de responder cualquier mensaje, evaluar si alguna skill, MCP server o plugin aplica. Si hay aunque sea un 1% de probabilidad de que una herramienta sea util, invocarla. No racionalizar para evitarla.

---

## MCP Servers

### Playwright (`playwright`)

Automatizacion de navegador. Navegar paginas, hacer click, llenar formularios, tomar screenshots, generar PDFs, grabar video.

**Usar cuando:**

- El usuario pide probar la app web visualmente o verificar que una pagina se ve bien
- Necesitas hacer screenshot de un componente o pagina para validar UI
- Testing de integracion E2E en el navegador (login flow, formularios, navegacion)
- Scraping de datos de paginas web
- Verificacion visual de cambios de CSS/layout despues de modificar componentes
- El usuario dice "abrí", "mostrá", "probá en el navegador", "screenshot", "captura"
- Debugging visual: "no se ve bien", "el boton no aparece", "la tabla se rompe"

### Context7 (`context7`)

Documentacion actualizada de librerias. Dos herramientas: `resolve-library-id` (buscar libreria) y `query-docs` (consultar docs).

**Usar cuando:**

- Necesitas la API actual de una libreria (Next.js, NestJS, Drizzle, Supabase, shadcn/ui, Radix, TanStack, Zod, Capacitor, React Hook Form, Recharts, Tailwind)
- No estas seguro de la sintaxis correcta de una funcion o componente
- Trabajas con una version especifica y necesitas docs de esa version
- El usuario pregunta "como se usa X" o "cual es la API de Y"
- Antes de implementar algo con una libreria que no usas frecuentemente
- Para evitar inventar APIs que no existen (alucinaciones)

**Flujo:** Primero `resolve-library-id` con el nombre, luego `query-docs` con el ID resuelto y la pregunta especifica.

### Supabase (`supabase`)

Gestion del proyecto Supabase. SQL, migraciones, Edge Functions, logs, tipos TypeScript.

**Usar cuando:**

- El usuario pide cambios en la autenticacion (Supabase Auth)
- Necesitas ejecutar SQL en Supabase (crear usuarios, asignar roles, modificar auth.users)
- Generar tipos TypeScript desde el schema de Supabase
- Consultar logs de Supabase (auth, API, edge functions)
- Deploy de Edge Functions
- El usuario menciona "supabase", "auth", "login", "signup", "roles", "JWT"
- Diagnostico de problemas de autenticacion ("no puedo loguear", "token invalido")
- Consultar la documentacion oficial de Supabase (knowledge base)

### shadcn-tabler-mcp (`shadcn-tabler-mcp`)

Transformacion de componentes shadcn/ui a estetica Tabler. Tres herramientas: `query-aesthetic` (consultar patron), `transform-component` (leer componente transformado), `map-tokens` (mapear variables CSS).

**Usar cuando:**

- Modificas o creas componentes UI en `apps/web/src/components/ui/`
- Necesitas saber como transformar un componente shadcn a estetica Tabler
- Aplicas estilos a nuevos componentes y necesitas consistencia con el sistema de diseno
- Mapeas variables CSS de Tabler a aliases de shadcn
- El usuario pide "mejorar el diseno", "que se vea como Tabler", "ajustar estilos"
- Cualquier cambio visual en componentes base (button, card, input, table, select, etc.)

**Reglas Tabler clave:** border-radius xl→md, lg→sm. Alturas h-10→h-9, h-9→h-8. Sombras reducir 1 nivel. Padding/gaps reducir (py-6→py-4, gap-6→gap-4). Texto base 14px (text-sm). Controles de formulario necesitan bg explicito.

---

## Plugins

### Superpowers (`superpowers`)

Framework de disciplina de desarrollo. Contiene 14 skills que cubren todo el ciclo: brainstorming, planificacion, implementacion, testing, debugging, code review y finalizacion.

#### superpowers:brainstorming

**Usar cuando:** Antes de cualquier trabajo creativo — crear features, construir componentes, agregar funcionalidad, modificar comportamiento. Explora la intencion del usuario y produce un documento de diseno antes de escribir codigo. Obligatorio antes de implementar.

#### superpowers:writing-plans

**Usar cuando:** Tienes spec o requisitos para una tarea multi-paso. Produce plan con tareas de 2-5 minutos cada una, con paths exactos, codigo completo y pasos de verificacion. Se invoca despues de brainstorming.

#### superpowers:subagent-driven-development

**Usar cuando:** Ejecutas un plan en la sesion actual con tareas independientes. Despacha un subagente fresco por tarea con review de dos fases (spec compliance + code quality).

#### superpowers:executing-plans

**Usar cuando:** Ejecutas un plan escrito en una sesion separada. Ejecuta tareas en lotes de 3 con checkpoints de feedback entre lotes.

#### superpowers:test-driven-development

**Usar cuando:** Implementas cualquier feature o bugfix. Ciclo RED-GREEN-REFACTOR. Regla de hierro: NO escribir codigo de produccion sin un test fallando primero.

#### superpowers:systematic-debugging

**Usar cuando:** Cualquier bug, test fallando o comportamiento inesperado. 4 fases: leer errores → comparar con referencia → hipotesis unica → test + fix. Regla de hierro: NO proponer fixes sin investigar la causa raiz primero. Si 3+ fixes fallan, parar y cuestionar la arquitectura.

#### superpowers:verification-before-completion

**Usar cuando:** Estas por declarar que el trabajo esta completo, que los tests pasan, o que un bug esta arreglado. Regla de hierro: NO declarar exito sin evidencia fresca de verificacion en ESTE mensaje.

#### superpowers:requesting-code-review

**Usar cuando:** Completaste tareas, implementaste features mayores, o antes de mergear. Despacha un subagente revisor que evalua contra el plan y requisitos.

#### superpowers:receiving-code-review

**Usar cuando:** Recibes feedback de code review. Evaluar tecnicamente, no aceptar ciegamente. Verificar contra el codebase antes de implementar sugerencias.

#### superpowers:dispatching-parallel-agents

**Usar cuando:** 2+ tareas independientes que pueden trabajarse sin estado compartido. Despacha un agente por dominio para investigacion concurrente.

#### superpowers:using-git-worktrees

**Usar cuando:** Feature work que necesita aislamiento del workspace actual. Crea worktrees aislados con setup automatico del proyecto.

#### superpowers:finishing-a-development-branch

**Usar cuando:** Implementacion completa, tests pasan. Presenta 4 opciones: merge local, crear PR, mantener branch, descartar.

#### superpowers:writing-skills

**Usar cuando:** Crear, editar o verificar skills. TDD adaptado para documentacion: baseline → write skill → test con agentes.

### Frontend Design (`frontend-design`)

Genera interfaces frontend distintivas y production-grade. Evita estetica generica de AI.

**Usar cuando:**

- Creas una pagina o componente visual nuevo desde cero
- El usuario pide "diseñar", "crear un dashboard", "landing page", "panel de configuracion"
- Necesitas decisiones esteticas fuertes (tipografia, paleta, animaciones)
- Cualquier trabajo frontend donde la apariencia visual importa

### UI/UX Pro Max (`ui-ux-pro-max`)

Skill avanzada de diseno UI/UX. Enfoque en experiencia de usuario profesional.

**Usar cuando:**

- Diseño de interfaces complejas que requieren atencion especial a UX
- El usuario pide mejorar la experiencia de usuario, no solo el aspecto visual
- Flujos de interaccion multi-paso (wizards, onboarding, checkout)
- Accesibilidad, responsive design, micro-interacciones
- Complementa a frontend-design cuando el foco es UX mas que estetica

---

## Skills Personalizadas

### generar-readme (`/generar-readme`)

Genera o actualiza README.md profesional en español desde `.planning/`.

**Usar cuando:** El usuario pide "README", "documentá el proyecto", "actualizá la documentación". Tambien despues de completar trabajo significativo.

### git-pushear (`/git-pushear`)

Flujo completo de push a GitHub: auth, remote, .gitignore, commit, push, descripcion.

**Usar cuando:** El usuario dice "pushear", "subir a git", "subir a github", "git push", "subir el codigo".

### vps-docs (`/vps-docs`)

Genera documentacion Docusaurus desde datos del VPS (docker-compose, Traefik, PostgreSQL).

**Usar cuando:** El usuario dice "documenta el VPS", "agrega este servicio", "actualiza la doc del VPS", "que puertos tengo libres", "genera el runbook".

### skill-creator (`/skill-creator`)

Ciclo completo de creacion de skills: draft → test → eval → improve.

**Usar cuando:** El usuario quiere crear, modificar, testear o mejorar una skill.

---

## GSD (Get Shit Done)

Framework de gestion de proyecto con `.planning/`. Fases, planes, ejecucion, verificacion.

### Inicio y planificacion

| Skill                           | Usar cuando                                                                 |
| ------------------------------- | --------------------------------------------------------------------------- |
| `/gsd:new-project`              | Proyecto nuevo desde cero. Questioning → research → requirements → roadmap. |
| `/gsd:new-milestone`            | Nuevo milestone en proyecto existente (v1.1, v2.0).                         |
| `/gsd:map-codebase`             | Antes de new-project en codebases existentes. Genera 7 docs de analisis.    |
| `/gsd:discuss-phase N`          | Clarificar decisiones de implementacion antes de planificar una fase.       |
| `/gsd:research-phase N`         | Investigacion profunda para dominios especializados (3D, ML, audio).        |
| `/gsd:list-phase-assumptions N` | Ver el enfoque planeado de Claude sin crear archivos.                       |
| `/gsd:plan-phase N`             | Crear plan detallado para una fase. Research → plan → verify loop.          |

### Ejecucion

| Skill                  | Usar cuando                                                         |
| ---------------------- | ------------------------------------------------------------------- |
| `/gsd:execute-phase N` | Ejecutar todos los planes de una fase con paralelizacion por waves. |
| `/gsd:quick "tarea"`   | Tarea rapida con commits atomicos pero sin ceremonia completa.      |
| `/gsd:verify-work [N]` | UAT conversacional despues de construir una fase.                   |

### Gestion de roadmap

| Skill                        | Usar cuando                                                        |
| ---------------------------- | ------------------------------------------------------------------ |
| `/gsd:add-phase "desc"`      | Agregar fase al final del milestone.                               |
| `/gsd:insert-phase N "desc"` | Insertar fase urgente entre fases existentes (numeracion decimal). |
| `/gsd:remove-phase N`        | Eliminar fase futura y renumerar.                                  |

### Milestone

| Skill                      | Usar cuando                                                |
| -------------------------- | ---------------------------------------------------------- |
| `/gsd:audit-milestone`     | Evaluar gaps y tech debt antes de cerrar milestone.        |
| `/gsd:plan-milestone-gaps` | Crear fases para cerrar gaps del audit.                    |
| `/gsd:complete-milestone`  | Archivar milestone, crear tag, preparar siguiente version. |

### Sesion y utilidades

| Skill                  | Usar cuando                                                |
| ---------------------- | ---------------------------------------------------------- |
| `/gsd:progress`        | Ver estado actual y siguiente accion. Routing inteligente. |
| `/gsd:resume-work`     | Restaurar contexto al iniciar nueva sesion.                |
| `/gsd:pause-work`      | Crear handoff de contexto antes de pausar.                 |
| `/gsd:add-todo "desc"` | Capturar idea para despues sin interrumpir el flujo.       |
| `/gsd:check-todos`     | Revisar y seleccionar todos pendientes.                    |
| `/gsd:settings`        | Configurar workflow (research on/off, modelo, branching).  |
| `/gsd:set-profile`     | Cambiar perfil de modelo (quality/balanced/budget).        |
| `/gsd:health`          | Diagnosticar integridad de `.planning/`.                   |
| `/gsd:cleanup`         | Archivar fases de milestones completados.                  |
| `/gsd:update`          | Actualizar GSD a ultima version.                           |
| `/gsd:debug`           | Debugging sistematico con estado persistente.              |

---

## Prioridad de herramientas

Cuando multiples herramientas aplican, usar este orden:

1. **Skills de proceso** primero (brainstorming, systematic-debugging, gsd:discuss-phase)
2. **MCP servers** para consultar informacion (context7, supabase, shadcn-tabler)
3. **Skills de implementacion** despues (writing-plans, test-driven-development, frontend-design)
4. **Skills de verificacion** al final (verification-before-completion, requesting-code-review)

## Convenciones del proyecto

- Idioma de la UI y documentacion: español (es-MX)
- Moneda: MXN
- Estetica: shadcn/ui + Tabler (consultar shadcn-tabler-mcp para transformaciones)
- Auth: Supabase (solo auth, no datos de negocio)
- DB: PostgreSQL con Drizzle ORM (datos de negocio)
- Roles: `admin` (lectura/escritura) y `viewer` (solo lectura) desde `app_metadata.role`
- Commits: conventional commits en ingles

---
> Source: [objetiva-comercios/objetiva-comercios-admin](https://github.com/objetiva-comercios/objetiva-comercios-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
