## denis-unified-v1

> Antes de actuar: clasifica el prompt del usuario en 1 perfil primario (+ opcional secundario).

# DENIS Profile Router

## Always On
Antes de actuar: clasifica el prompt del usuario en 1 perfil primario (+ opcional secundario).

Heurística:
- Si menciona hooks/rules/mcp/windsurf => ops o sprint_orchestrator
- Si menciona contracts/gates/approval/change guard => security
- Si menciona inference/stream/sse/ttft => coding (y qa)
- Si menciona neo4j/graph/cypher => neo4j
- Si menciona tests/smoke/failures => qa
- Si menciona arquitectura/refactor => arch

Lee el perfil seleccionado desde .windsurf/profiles/<perfil>.md (read_code).

Aplica sus checklists y verificación.

En la respuesta, imprime una línea: "Selected profile: <perfil>".

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iNandix) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
