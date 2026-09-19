## appstack

> 1. Lee `PROGRESO.md`.

# =============================================
# PROTOCOLO DE CONTINUIDAD Y TRASPASO DE ESTADO (HANDOFF PROTOCOL)
# Proyecto: AppStack
# Aplicable a: cualquier agente IA (Cline, Copilot, Claude Code, etc.)
# =============================================

## 1. Primera acción OBLIGATORIA al iniciar sesión

1. Lee `PROGRESO.md`.
2. Busca el bloque `[STATE_SNAPSHOT] ... [END STATE_SNAPSHOT]`.
3. Si existe, ese es TU punto de partida. NO vuelvas a investigar lo ya resuelto.

## 2. Fase de Entrada (Análisis de Continuidad)

Reglas estrictas:

- **Prohibición de Repetición:** No vuelvas a ejecutar búsquedas, lecturas o análisis ya listados en `COMPLETED_STEPS` o `DECISIONS_MADE`. Asume esos resultados como verdades absolutas.
- **Goal Anchoring:** Tu único foco es `ACTIVE_GOAL`. Si tus acciones se desvían hacia tareas secundarias no listadas, DETENTE y vuelve al objetivo.
- **Punto de Partida:** Empieza exactamente en `NEXT_STEP`.
- **Evitación de Errores Históricos:** Si `ERRORS_AND_RESOLUTIONS` contiene fallos previos, no intentes esos mismos enfoques. Elige una alternativa distinta.

## 3. Fase de Salida (Emisión de Ticket de Estado)

Al finalizar tu trabajo DEBES generar un bloque de estado actualizado:

```
[STATE_SNAPSHOT]
- **ACTIVE_GOAL:** ...
- **COMPLETED_STEPS:**
  1. ... -> ... 
- **DECISIONS_MADE:**
  - ... -> ...
- **ERRORS_AND_RESOLUTIONS:**
  - ... -> ...
- **NEXT_STEP:** ...
- **PENDING_ITEMS:**
  - [ ] ...
[END STATE_SNAPSHOT]
```

El bloque DEBE:
- Reemplazar al anterior en `PROGRESO.md`.
- Commit + push (`git add PROGRESO.md && git commit -m "docs: update state snapshot" && git push`).
- Ser factual, técnico y denso. Nada de "Claro, continúo...", resúmenes sociales ni despedidas.

## 4. Handoff a otro agente

Cuando otro agente retome: copia el bloque `[STATE_SNAPSHOT]` actual y pégalo como `[LAST_STATE_SNAPSHOT]` al inicio de su contexto.

## 5. Script auxiliar

`scripts/handoff.mjs` existe para:
- `node scripts/handoff.mjs show`   -> imprime el snapshot actual
- `node scripts/handoff.mjs commit` -> commit + push del PROGRESO.md
- `node scripts/handoff.mjs import <archivo.md>` -> extrae el bloque de un archivo

## 6. Regla de Oro

Sé puramente factual y técnico. El código y el bloque final de estado son más importantes que las transiciones conversacionales.

---
> Source: [transittaxmc-create/appstack](https://github.com/transittaxmc-create/appstack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
