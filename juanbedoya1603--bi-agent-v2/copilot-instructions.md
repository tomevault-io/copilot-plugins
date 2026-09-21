## bi-agent-v2

> Construir un asistente interno de BI simple, rápido y útil. Debe responder preguntas sobre ventas, productos y tiendas usando SQL Server real y ahorrar tiempo al equipo.

# AGENTS.md

## Objetivo

Construir un asistente interno de BI simple, rápido y útil. Debe responder preguntas sobre ventas, productos y tiendas usando SQL Server real y ahorrar tiempo al equipo.

## Principio principal

El LLM puede generar SQL directamente. No crear una capa intermedia de intención analítica ni un compilador semántico.

Flujo esperado:

```text
pregunta -> agente -> SQL -> validación read-only -> SQL Server -> filas -> agente -> respuesta
```

## Stack

- Python 3.11
- FastAPI
- OpenAI Agents SDK
- SQL Server
- SQLAlchemy/pyodbc o pyodbc directo para lectura
- `sqlglot` para validación AST T-SQL
- Next.js + TypeScript + shadcn/ui para frontend
- SQLite para sesiones del MVP si se necesita persistencia
- pytest + Ruff

## Reglas obligatorias

1. Solo se puede consultar:
   - `dbo.VW_SalesLast13Months`
   - `dbo.VW_Stores`
   - `dbo.VW_Products`
2. Solo lectura.
3. Una sola sentencia.
4. Permitir `SELECT` y `WITH ... SELECT`.
5. Bloquear DDL/DML, `EXEC`, `SELECT INTO`, linked servers y otras bases.
6. El usuario SQL también debe tener permisos SELECT solamente.
7. Máximo 200 filas devueltas a la aplicación. No limitar agregaciones internas.
8. Timeout SQL configurable; default 600 s.
9. El agente puede corregir y reintentar SQL ante errores, máximo 3 intentos SQL por turno.
10. El agente nunca inventa cifras si no tuvo un resultado SQL exitoso.

## Simplicidad

No reintroducir arquitectura del proyecto anterior salvo que un fallo real demostrado lo justifique.

No crear:

- semantic DSL;
- `target/context` en código;
- registry de universos;
- compilador SQL propio;
- grounding por referencias de métricas;
- tool específica por cada KPI.

Las reglas de negocio viven principalmente en el system prompt y en ejemplos SQL claros.

## Forma de trabajo

- Implementar vertical slices pequeñas.
- Primero hacer que funcione una pregunta real end-to-end.
- Agregar guardrails solo ante fallos reproducibles.
- Mantener tests sobre seguridad SQL y casos BI críticos.
- No avanzar de fase si los casos de aceptación de esa fase fallan.

---
> Source: [juanbedoya1603/bi-agent-v2](https://github.com/juanbedoya1603/bi-agent-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
