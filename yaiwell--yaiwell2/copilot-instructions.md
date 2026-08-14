## yaiwell2

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

---

# AGENTS.md — Orquestación de subagentes en Yaiwell

> Este archivo le dice al agente orquestador **cuántos subagentes lanzar, de qué tipo, con qué prompt y en qué orden**, para cada clase de tarea habitual en este repo.
>
> Si una tarea encaja con un playbook de aquí, **el orquestador ejecuta sin preguntar**. Solo pregunta cuando la tarea no encaja en ninguno o cuando hay decisión de producto pendiente.
>
> Este archivo lo importa `CLAUDE.md`. Las reglas de código, stack y workflow viven allí; aquí solo viven reglas de orquestación.

---

## 1. Agentes disponibles

| Agente | Cuándo usar | Cuándo NO usar |
|---|---|---|
| **Explore** | Búsqueda abierta en el código (>3 queries esperadas), entender una feature antes de tocarla, auditorías de un área. Aceptar parámetro `thoroughness: quick \| medium \| very thorough`. | Buscar un símbolo concreto (usar `Grep`), leer 1-3 ficheros (usar `Read`). |
| **Plan** | Diseñar la implementación de una feature mediana/grande antes de tocar código. Devuelve plan paso a paso + archivos críticos + trade-offs. | Cambios triviales, bugfixes locales, decisiones ya tomadas. |
| **general-purpose** | Ejecutar trabajo de implementación o investigación multi-paso. Es el agente "operario" cuando paralelizamos. | Tareas que el orquestador puede hacer en 2-3 tool calls directos. |
| **claude-code-guide** | Dudas sobre Claude Code, Agent SDK, API de Anthropic. | Dudas sobre Next.js, Prisma, Stripe, Tailwind. |
| **statusline-setup** | Configurar la status line de Claude Code. | Cualquier otra cosa. |

**Custom agents** (`.claude/agents/`): ninguno por ahora. Si alguna vez se añaden, listarlos aquí.

---

## 2. Reglas de orquestación (no negociables)

1. **Subagentes NUNCA hacen `git commit` ni `git push`.** El orquestador es el único que toca el historial. Los subagentes editan ficheros y devuelven un resumen.
2. **Paralelizar siempre que las tareas sean independientes.** Si lanzas 3 subagentes que no se pisan, lánzalos en **un único mensaje con 3 bloques de tool call** (no secuenciales).
3. **Si dos subagentes pueden tocar los mismos ficheros, usar `isolation: "worktree"`** en ambos para evitar conflictos, y luego mergear/aplicar tú el resultado. Para refactors cross-cutting esto es obligatorio.
4. **Prompts de subagente siempre completos.** Cada invocación empieza fresca: incluye contexto, ficheros relevantes, criterio de éxito y formato de respuesta esperado. No asumas que el subagente recuerda nada.
5. **Dile explícitamente al subagente si debe escribir código o solo investigar.** Es la fuente nº1 de malentendidos.
6. **El orquestador valida.** Después de cada lote de subagentes: `typecheck`, `lint`, `test` antes de commitear. Si rompe, no abrir más subagentes hasta arreglar.
7. **No usar subagentes para tareas <3 tool calls.** El overhead no compensa.
8. **Respetar `TODO.md` / `DO.md`.** Antes de empezar, mirar `TODO.md`. Al terminar, mover a `DO.md` con fecha. Esto lo hace el orquestador, no los subagentes.

---

## 3. Heurística de tamaño

| Señal | Estrategia |
|---|---|
| 1 fichero, <50 líneas de cambio, sin lógica nueva | **Solo orquestador**, sin subagentes. |
| 1-3 ficheros, cambio acotado, conocimiento del área ya cargado | **Solo orquestador**. |
| Cambio en >3 ficheros o área que no conozco | **1 Explore** primero → orquestador implementa. |
| Feature nueva con UI + lógica + tipos | **1 Plan** → orquestador implementa serial. |
| Feature grande dividida en piezas independientes (A, B, C que no se pisan) | **1 Plan** → **N general-purpose en paralelo** (uno por pieza) → orquestador integra. |
| Auditoría / review de un área | **N Explore en paralelo** (uno por ángulo: color, a11y, perf, etc.). |
| Migración o renombrado cross-cutting | **1 Explore** (mapeo de impacto) → ejecución directa por orquestador con `Grep`+`Edit`. |

---

## 4. Playbooks por tipo de tarea

> Plantilla común: **Trigger / Agentes / Prompt base / Criterio de éxito**.

### P1 · Bugfix puntual
- **Trigger:** "no funciona X", "error en Y", "se ve mal Z".
- **Agentes:** ninguno. El orquestador reproduce, localiza con `Grep`, lee con `Read`, edita con `Edit`.
- **Excepción:** si el bug no se reproduce o el área es desconocida → **1 Explore (thoroughness: medium)**.
- **Éxito:** test que reproduce el bug pasa + `typecheck` + `lint` verde.

### P2 · Feature pequeña (1 vista o 1 servicio nuevo)
- **Trigger:** "añade un botón que…", "crea endpoint que…".
- **Agentes:** ninguno por defecto. El orquestador implementa siguiendo §6.bis de `CLAUDE.md` (separación `.tsx` / `.styles.ts` / `.logic.ts` / `.types.ts` / `index.ts`).
- **Excepción:** si toca 2+ módulos no familiares → **1 Plan** primero.
- **Éxito:** UI accesible, i18n es+ca, tests del happy path, `npm run validate` verde.

### P3 · Feature mediana (multivista, multimódulo)
- **Trigger:** "geolocalización completa", "panel del proveedor con X secciones".
- **Agentes:**
  1. **1 Plan** → devuelve plan con piezas A/B/C y dependencias.
  2. Si A/B/C son independientes → **N general-purpose en paralelo**, uno por pieza, cada uno en `isolation: "worktree"` si tocan ficheros compartidos.
  3. Orquestador integra, ejecuta validación y commitea.
- **Prompt base de implementador:** "Implementa la pieza [X] descrita en este plan: [...]. Sigue §6.bis de CLAUDE.md (separación de archivos, comentarios en castellano, i18n). No commitees. Devuelve: ficheros tocados, decisiones tomadas, dudas pendientes."
- **Éxito:** piezas integradas + tests + `npm run validate` verde + entrada en `DO.md`.

### P4 · Feature grande (atraviesa dominio + UI + datos)
- **Trigger:** "flujo de reserva", "Stripe Connect end-to-end".
- **Agentes:**
  1. **1 Plan** con desglose por dominios (data → service → API → UI).
  2. **1 Explore** en paralelo si hay áreas que el orquestador no conoce.
  3. Implementación serial por capas (data primero, UI al final) para no romper tipos a mitad. Paralelizar **dentro** de cada capa si hay piezas independientes.
- **Éxito:** schema migrado, servicios con errores tipados, endpoints con Zod, UI con i18n, tests E2E del happy path.

### P5 · Refactor cross-cutting (rename, restructure, dark-mode pass)
- **Trigger:** "renombra X a Y en toda la app", "alinea el componente Z al sistema de diseño".
- **Agentes:**
  - Si es **mecánico** (rename literal): **0 subagentes**. Orquestador hace mapeo con `Grep`, ejecuta con `Edit`/sed, valida. (Patrón usado en commit `f9a6147`.)
  - Si es **semántico** (tocar lógica/estilos según patrones distintos por archivo): **1 Explore** para mapear impacto → **2-3 general-purpose en paralelo en worktrees** por subárea (ej. `components/features/landing/*`, `components/features/search/*`) → orquestador integra.
- **Éxito:** 0 referencias residuales (`Grep` final), `npm run validate` verde, 1 commit limpio.

### P6 · Migración de dependencia (Next.js minor, shadcn major, Prisma…)
- **Trigger:** "subir Next a 16.3", "migrar Tailwind v3→v4".
- **Agentes:**
  1. **1 Explore (very thorough)** del changelog/breaking changes en `node_modules/next/dist/docs/` (o equivalente).
  2. **1 Plan** con orden de migración y puntos de riesgo.
  3. Ejecución serial por el orquestador, con validación entre pasos.
- **Éxito:** build, tests y dev server funcionan; `DO.md` registra la versión final.

### P7 · Auditoría de diseño / UX (modo oscuro, accesibilidad, mobile-first)
- **Trigger:** "revisa el modo oscuro", "audita la accesibilidad de la landing".
- **Agentes:** **3 Explore en paralelo**, cada uno con un ángulo distinto:
  - Color/contraste/legibilidad (sobre `globals.css` + tokens).
  - Coherencia con shadcn/ui y estructura §6.bis.
  - Accesibilidad (ARIA, foco, navegación teclado) y responsive (375→1440).
- **Prompt base:** "Eres revisor de [ángulo]. Recorre [scope]. Devuelve: hallazgos con `archivo:línea`, severidad (P0/P1/P2), propuesta concreta. NO escribas código."
- **Tras los hallazgos:** el orquestador prioriza, aplica los P0/P1 directamente o lanza un P3/P5 según volumen.

### P8 · Auditoría técnica (perf, seguridad, bundle, tests)
- **Trigger:** "mira si hay N+1", "revisa el bundle de la landing", "qué endpoints van sin auth".
- **Agentes:** **1-3 Explore en paralelo** por ángulo (queries Prisma, endpoints API, dependencias pesadas, cobertura de tests).
- **Éxito:** informe en `docs/audit-YYYY-MM-DD.md` con hallazgos y prioridades. Implementación es una tarea separada (entra a `TODO.md`).

### P9 · Exploración / research del repo
- **Trigger:** "cómo funciona el flujo de búsqueda", "dónde se calcula la comisión".
- **Agentes:**
  - <3 queries esperadas → **`Grep`/`Glob` directos**.
  - Pregunta abierta → **1 Explore (thoroughness: medium)**.
  - Mapa completo de un dominio → **1 Explore (very thorough)**.
- **Éxito:** respuesta con citas `archivo:línea`.

### P10 · Code review pre-PR / pre-commit grande
- **Trigger:** "antes de hacer push, revisa".
- **Agentes:** **2 Explore en paralelo**:
  1. Cumplimiento de §6.bis (estructura, idioma de comentarios, i18n, separación de archivos).
  2. Cumplimiento de reglas de negocio §4.bis (cancelaciones 2h, valoraciones solo si `completed`, RLS, validación con Zod).
- **Éxito:** lista de findings con propuesta de fix, aplicada antes del commit.

### P11 · i18n review (es + ca)
- **Trigger:** "faltan textos en catalán", "revisar coherencia es/ca".
- **Agentes:** **1 Explore** que compare `src/messages/es.json` y `src/messages/ca.json` y detecte claves desbalanceadas, hardcoded strings en JSX, y faltas de neutralidad de género.
- **Éxito:** ambos ficheros con mismas claves, 0 strings hardcoded en componentes tocados.

---

## 5. Anti-patterns (no hacer nunca)

- **No lances subagentes en serie cuando podrían ir en paralelo.** Es la pérdida de tiempo nº1.
- **No dupliques trabajo entre orquestador y subagente.** Si el subagente está investigando X, no hagas tú `Grep` de lo mismo en paralelo.
- **No dejes que un subagente decida arquitectura sin Plan previo.** Devuelven implementaciones plausibles pero incoherentes con el resto del repo.
- **No uses subagentes para tareas triviales.** Si en 2-3 tool calls lo resuelves, hazlo.
- **No invoques `general-purpose` para investigar cuando `Explore` es más rápido.** `Explore` está optimizado para eso.
- **No commitees desde un subagente.** Aunque tenga sandbox para hacerlo. Lo hace el orquestador después de validar.
- **No saltes la validación entre lotes.** Si lanzas 3 implementadores en paralelo, valida antes de lanzar el siguiente lote.
- **No olvides `TODO.md`/`DO.md`.** El subagente no lo va a actualizar; lo hace el orquestador al cerrar la tarea.

---

## 6. Cómo evolucionar este archivo

Cuando aparezca un patrón de orquestación nuevo (ej. la tarea se resolvió bien con una combinación no listada), añadirlo como playbook **Pxx** después de cerrarla. Cuando un playbook deje de aplicarse, retirarlo. Si dos playbooks empiezan a colisionar, fundirlos.

Actualizaciones a este archivo van en commits `docs: actualizar AGENTS.md — [resumen]`.

---
> Source: [yaiwell/yaiwell2](https://github.com/yaiwell/yaiwell2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
