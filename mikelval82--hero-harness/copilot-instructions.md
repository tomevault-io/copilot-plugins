## hero-harness

> > Punto de entrada para cualquier agente. NO es una biblia de reglas: es un

# Mapa de Metodologia — Harness Engineering

> Punto de entrada para cualquier agente. NO es una biblia de reglas: es un
> **mapa**. Lee solo lo que necesites cuando lo necesites (divulgacion progresiva).

---

## 1. Agents disponibles

Definidos en `~/.claude/agents/`. Los agentes se dividen en dos categorias:

### Pipeline (automaticos via `mission.sh`)

Registrados en `PHASE_REGISTRY`. En modo autonomo, cada agente se invoca
directamente via `claude -p` con sus instrucciones inlined. No hay capa intermedia.

| Agent | Rol | Produce |
|-------|-----|---------|
| `researcher` | Investiga el codebase, explora enfoques. No escribe codigo. | `brainstorm.md` + `context-hot.md` |
| `griller` | Interrogatorio exhaustivo para resolver ambiguedades antes de disenar. | `brief.md` |
| `structurer` | Genera estructura de tareas a partir del brainstorm. | `tasks.json` |
| `specifier` | Crea especificacion tecnica a partir del sprint. No escribe codigo. | `spec.md` + append a `context-hot.md` |
| `planner` | Crea plan de implementacion concreto. No escribe codigo. | `plan.md` + `decisions.md` + append a `context-hot.md` |
| `implementer` | Ejecuta UNA tarea del plan. Escribe codigo y tests. | Codigo + `status.md` + append a `context-hot.md` |
| `reviewer` | Aprueba o rechaza trabajo. NUNCA edita codigo. | `audit.md` |

### Invocacion manual

No estan en `PHASE_REGISTRY`. Se usan bajo demanda fuera del pipeline automatizado.

| Agent | Rol | Produce |
|-------|-----|---------|
| `content_reviewer` | Valida calidad de documentacion. No modifica contenido. | `content_review.md` |

## 2. Commands disponibles

Definidos en `~/.claude/commands/`. Solo se activan cuando el usuario los invoca con `/nombre`.

### Pipeline de desarrollo (secuencial)

En modo autonomo (`mission.sh`), estos commands se bypasean — las instrucciones
del agente se pasan directamente via `claude -p`. Los commands existen para uso
interactivo solamente.

| Command | Agente | Lee de | Escribe en |
|---------|--------|--------|------------|
| `/brainstorm` | `researcher` | Codebase | `brainstorm.md` + `context-hot.md` |
| `/spec-task` | `specifier` | `brainstorm.md`, `tasks.json`, `context-cold.md`, `context-hot.md` | `spec.md` + append `context-hot.md` |
| `/plan-task` | `planner` | `spec.md`, `brainstorm.md`, `context-cold.md`, `context-hot.md` | `plan.md`, `decisions.md` + append `context-hot.md` |
| `/implement-task` | `implementer` | `plan.md`, `spec.md`, `decisions.md`, `context-cold.md`, `context-hot.md` | Codigo + `status.md` + append `context-hot.md` |
| `/review-task` | `reviewer` | `spec.md`, `plan.md`, `status.md` | `audit.md` |

### Skills de productividad (independientes)

| Command | Proposito |
|---------|-----------|
| `/mission-align` | Sesion de alineacion previa a una mision autonoma |
| `/grill` | Interrogatorio exhaustivo antes de disenar/implementar |
| `/grill-content` | Alineacion antes de crear/modificar documentacion |
| `/checkpoint` | Verificacion activa de comprension sobre un documento |
| `/diagnose` | Loop de debugging disciplinado (6 fases) |
| `/zoom-out` | Mapa de modulos y callers para orientacion rapida |
| `/caveman` | Modo ultra-comprimido (~75% menos tokens) |
| `/context-audit` | Auditoria de contexto: trayectoria, relevancia, limpieza |
| `/refine-harness` | Refiner offline: propone mejoras a prompts/agentes desde fallos recurrentes sin aplicarlas |
| `/write-a-skill` | Meta-skill para crear nuevos commands |
| `/harness-clean` | Limpia workspace temporal manualmente |

## 3. Workspace efimero

Todos los artefactos del pipeline se escriben en el directorio apuntado por
`$CLAUDE_HARNESS`. En ejecucion normal, `setup_harness()` lo crea como:

`$HOME/.harness/<project>/<branch-safe>/`

NUNCA se escriben artefactos del harness dentro del directorio del proyecto
target. El proyecto target solo recibe los cambios de codigo solicitados.

- `mission.sh` crea el workspace al inicio y exporta `CLAUDE_HARNESS`.
- `/harness-clean` limpia manualmente el workspace activo (`$CLAUDE_HARNESS`).

## 4. Pizarra compartida (layered context)

La pizarra se divide en dos capas para evitar crecimiento lineal del contexto:

### Hot layer (`context-hot.md`)
Hallazgos de la tarea/fase actual. Append-only durante la tarea, se resetea al completarla.
Cada agente anade su seccion:

- `## Researcher` — hallazgos iniciales del codebase
- `## Specifier (task_id)` — edge cases, gotchas, codebase findings
- `## Planner (task_id)` — riesgos, enfoques descartados, puntos de integracion
- `## Implementer (task_id)` — deltas del plan, sorpresas, resultados de tests

### Compactacion automatica
`_compact_context()` en `MissionRunner` se ejecuta automaticamente al completar cada tarea:
1. Lee `context-hot.md`
2. Comprime a ~20 lineas via `claude -p` (preserva paths, patrones, gotchas)
3. Append del resumen a `context-cold.md`
4. Elimina `context-hot.md`

Solo hechos verificables. Nada de opiniones ni decisiones (eso va en brainstorm.md y decisions.md).

## 5. Reglas duras

- **Opt-in**: el pipeline solo se activa cuando el usuario invoca un command o lanza `mission.sh`. El comportamiento normal de Claude no cambia.
- **Escritura a disco**: los agentes escriben resultados en `$CLAUDE_HARNESS/`. Los artefactos son la fuente de verdad.
- **Un flujo a la vez**: no mezcles desarrollo de multiples tareas en el mismo workspace.
- **Zero footprint**: ningun artefacto del harness queda en el proyecto del usuario.

## 6. Documentos de referencia

| Documento | Ubicacion | Proposito |
|-----------|-----------|-----------|
| `AGENTS.md` | `~/.claude/AGENTS.md` | Este mapa |
| `CHECKPOINTS.md` | `~/.claude/CHECKPOINTS.md` | Criterios de calidad universales |
| `CONTEXT.md` | `~/.claude/CONTEXT.md` | Vocabulario compartido de la metodologia |
| `harness-diagram.md` | `~/.claude/harness-diagram.md` | Diagramas de arquitectura y flujo |

## 7. Si te bloqueas

- Relee la seccion relevante de este mapa.
- Revisa `context-hot.md` y `context-cold.md` en el workspace para orientarte en el codebase.
- Si una herramienta no hace lo que esperas, documenta el bloqueo y reporta al usuario.

---
> Source: [mikelval82/hero-harness](https://github.com/mikelval82/hero-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
