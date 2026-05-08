## bendita-ia

> Este archivo proporciona orientación a Claude Code (claude.ai/code) cuando trabaja con código en este repositorio.

# CLAUDE.md

Este archivo proporciona orientación a Claude Code (claude.ai/code) cuando trabaja con código en este repositorio.

## Descripción General del Repositorio

Este es un repositorio de mejores prácticas para la configuración de Claude Code, que demuestra patrones para habilidades, subagentes, hooks y comandos. Sirve como implementación de referencia en lugar de una base de código de aplicación.

## Componentes Clave

### Sistema Climático (Flujo de Trabajo de Ejemplo)
Una demostración del patrón de arquitectura **Comando → Agente → Habilidades**:
- Comando `/weather-orchestrator` (`.claude/commands/weather-orchestrator.md`): Punto de entrada que invoca el agente del clima
- Agente `weather` (`.claude/agents/weather.md`): Ejecuta el flujo de trabajo usando habilidades precargadas
- Habilidad `weather-fetcher` (`.claude/skills/weather-fetcher/SKILL.md`): Instrucciones para obtener la temperatura de la API wttr.in
- Habilidad `weather-transformer` (`.claude/skills/weather-transformer/SKILL.md`): Instrucciones para aplicar las reglas de transformación desde `weather-orchestration/input.md`, escribe los resultados en `weather-orchestration/output.md`

El agente tiene las habilidades precargadas mediante el campo `skills`, proporcionando conocimiento del dominio para la ejecución secuencial. Consulta `weather-orchestration/weather-orchestration-architecture.md` para ver el diagrama de flujo completo.

### Estructura de Definición de Habilidades
Las habilidades en `.claude/skills/<name>/SKILL.md` usan frontmatter YAML:
- `name`: Identificador de la habilidad (opcional, usa el nombre del directorio si se omite)
- `description`: Cuándo invocar (recomendado para autodescubrimiento)
- `model`: Modelo a usar cuando la habilidad está activa
- `disable-model-invocation`: Establece en `true` para prevenir la invocación automática
- `context`: Establece en `fork` para ejecutar en contexto aislado de subagente
- `allowed-tools`: Restringe qué herramientas puede usar Claude

### Sistema de Presentación
Cualquier solicitud de actualizar, modificar o corregir la presentación (`presentation/index.html`) debe ser manejada por el agente `presentation-curator` (`.claude/agents/presentation-curator.md`). Siempre delega el trabajo de la presentación a este agente mediante la herramienta Task — nunca edites la presentación directamente.

El agente es **auto-evolutivo**: después de cada ejecución, actualiza sus propias habilidades para mantenerse sincronizado con la presentación. Tiene tres habilidades precargadas:
- `vibe-to-agentic-framework`: El marco conceptual ("Vibe Coding → Ingeniería Agéntica"), la justificación de pesos y la narrativa del recorrido. Se actualiza después de cada cambio de diapositiva.
- `presentation-structure`: Formato de diapositivas, sistema de pesos, navegación, rangos de secciones. Se actualiza cuando se añaden/eliminan/reordenan diapositivas.
- `presentation-styling`: Clases CSS, patrones de componentes, resaltado de sintaxis. Se actualiza cuando se introducen nuevos patrones de estilo.

### Sistema de Hooks
Sistema de notificación de sonido multiplataforma en `.claude/hooks/`:
- `scripts/hooks.py`: Manejador principal para los eventos de hooks de Claude Code
- `config/hooks-config.json`: Configuración compartida del equipo
- `config/hooks-config.local.json`: Anulaciones personales (ignorado por git)
- `sounds/`: Archivos de audio organizados por evento de hook (generados mediante TTS de ElevenLabs)

Eventos de hooks configurados en `.claude/settings.json`: PreToolUse, PostToolUse, UserPromptSubmit, Notification, Stop, SubagentStart, SubagentStop, PreCompact, SessionStart, SessionEnd, Setup, PermissionRequest.

Manejo especial: los commits de git activan el sonido `pretooluse-git-committing`.

## Patrones Críticos

### Orquestación de Subagentes
Los subagentes **no pueden** invocar otros subagentes mediante comandos bash. Usa la herramienta Task:
```
Task(subagent_type="agent-name", description="...", prompt="...", model="haiku")
```

Sé explícito sobre el uso de herramientas en las definiciones de subagentes. Evita términos vagos como "lanzar" que podrían interpretarse como comandos bash.

### Estructura de Definición de Subagentes
Los subagentes en `.claude/agents/*.md` usan frontmatter YAML:
- `name`: Identificador del subagente
- `description`: Cuándo invocar (usa "PROACTIVELY" para auto-invocación)
- `tools`: Lista de herramientas permitidas separadas por comas
- `model`: Típicamente "haiku" para eficiencia
- `color`: Color de salida en CLI para distinción visual
- `skills`: Lista de nombres de habilidades para precargar en el contexto del agente
- `memory`: Alcance de memoria persistente — `user`, `project`, o `local` (consulta `reports/claude-agent-memory.md`)

### Jerarquía de Configuración
1. `.claude/settings.local.json`: Configuración personal (ignorado por git)
2. `.claude/settings.json`: Configuración compartida del equipo
3. `hooks-config.local.json` anula `hooks-config.json`

### Desactivar Hooks
Establece `"disableAllHooks": true` en `.claude/settings.local.json`, o desactiva hooks individuales en `hooks-config.json`.

## Mejores Prácticas de Flujo de Trabajo

Basado en la experiencia con este repositorio:

- Mantén CLAUDE.md por debajo de 150 líneas para una adherencia confiable
- Usa comandos para flujos de trabajo en lugar de agentes independientes
- Crea subagentes específicos de características con habilidades (divulgación progresiva) en lugar de agentes de propósito general
- Realiza `/compact` manual al ~50% de uso del contexto
- Comienza con el modo de planificación para tareas complejas
- Usa el flujo de trabajo de lista de tareas con control humano para tareas de múltiples pasos
- Divide las subtareas lo suficientemente pequeñas para completarlas en menos del 50% del contexto

### Consejos de Depuración

- Usa `/doctor` para diagnósticos
- Ejecuta comandos de terminal de larga duración como tareas en segundo plano para mejor visibilidad de logs
- Usa MCPs de automatización del navegador (Claude en Chrome, Playwright, Chrome DevTools) para que Claude inspeccione los logs de la consola
- Proporciona capturas de pantalla cuando reportes problemas visuales

## Documentación

- `docs/AGENTS.md`: Resolución de problemas de orquestación de subagentes
- `weather-orchestration/weather-orchestration-architecture.md`: Diagrama de flujo del sistema climático
- `docs/COMPARISION.md`: Patrones de invocación de Comandos vs Agentes vs Habilidades

## Reportes

- `reports/claude-in-chrome-v-chrome-devtools-mcp.md`: Comparación de MCP de automatización del navegador (Playwright vs Chrome DevTools vs Claude en Chrome)
- `reports/claude-md-for-larger-mono-repos.md`: Comportamiento de carga de CLAUDE.md en monorepos (carga ascendente vs descendente)
- `reports/claude-skills-for-larger-mono-repos.md`: Comportamiento de descubrimiento y carga de habilidades en monorepos (descubrimiento estático vs dinámico)
- `reports/claude-agent-memory.md`: Frontmatter de memoria de agente — alcances de memoria persistente (user, project, local) para subagentes
- `reports/claude-advanced-tool-use.md`: Patrones avanzados de uso de herramientas — Llamada Programática de Herramientas (PTC), Búsqueda de Herramientas, Ejemplos de Uso de Herramientas

---
> Source: [DivergentiADM/Bendita-IA](https://github.com/DivergentiADM/Bendita-IA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
