## 00-master

> SIEMPRE ACTIVO. Constitución maestra del sistema de trading cuantitativo. Lee esto antes de cualquier tarea.


# 🎯 QUANTSTATION — CONSTITUCIÓN MAESTRA v3.0

## IDENTIDAD DEL SISTEMA
`deep-funnel-station` es una estación de trading cuantitativo institucional.
- **Backend**: Python 3.12+ — motor asíncrono de 4 fases (A→B→C→D)
- **Frontend**: Next.js 16 / React 19 — shell de interfaz (Fase 1 activa)
- **Estándares**: NautilusTrader · FINOS CDM · Wall Street grade

## PROTOCOLO DE ARRANQUE (OBLIGATORIO)
Antes de escribir CUALQUIER línea de código:
1. Lee `CLAUDE.md` para el contexto maestro
2. Identifica: ¿backend Python? ¿frontend TypeScript? ¿qué fase?
3. Carga la regla `.cursor/rules/` específica para el dominio
4. Declara tu plan. Si es destructivo, pide confirmación.

## LAS 10 REGLAS ABSOLUTAS (Nunca violables)

| Código | Regla | Consecuencia |
|--------|-------|--------------|
| `PD-1` | Una función = una tarea. Máx. 30 líneas | RECHAZAR |
| `PD-2` | Cero secrets en código fuente | RECHAZAR |
| `PD-3` | Fase B/C no llaman APIs externas directamente | RECHAZAR |
| `PD-4` | Todo dato inter-fase: Pydantic `frozen=True` | RECHAZAR |
| `PD-5` | No `print()` — solo `logging` | AUTO-CORREGIR |
| `PD-6` | No `except: pass` — cada error se loggea | AUTO-CORREGIR |
| `PD-7` | No `time.sleep()` en contexto async | AUTO-CORREGIR |
| `PD-8` | Sin números mágicos — todo va en `config/` | RECHAZAR |
| `PD-9` | Fase A NO conecta directo a Fase D | RECHAZAR |
| `PD-10` | YAGNI — solo lo que la tarea actual necesita | ADVERTIR |

## MANDATOS SIEMPRE ACTIVOS
- `[TH]` Type hints en toda firma de función
- `[DL]` `data_lineage` en todo `MarketSnapshot`
- `[IM]` Pydantic `frozen=True` en modelos inter-fase
- `[GS]` Google-Style docstrings en funciones públicas
- `[AL]` asyncio para I/O · ProcessPoolExecutor para CPU

## FLUJO DE TRABAJO DEL AGENTE
```
CLARIFICAR → PLANIFICAR → VERIFICAR → CODIFICAR → CONFIRMAR
```
- **CLARIFICAR**: "Esta tarea es [fase/dominio]"
- **PLANIFICAR**: Lista archivos antes de escribir código
- **VERIFICAR**: ¿Viola PD-1..PD-10? 
- **CODIFICAR**: Etiqueta reglas: `# [PD-3][TH][IM]`
- **CONFIRMAR**: Lista qué gates de CI deben pasar

## GESTIÓN DE TOKENS (Anti-degradación)
- Máximo 2 reglas cargadas por sesión
- Tras 10 exchanges → CHECKPOINT y resumen
- Si ves código espagueti → REFACTORIZA antes de agregar features
- Si el contexto se llena → DETENTE y pide sesión nueva con `CLAUDE.md`

## TABLA DE FASES
| Fase | Stack | Estado |
|------|-------|--------|
| Backend A/B/C/D | Python 3.12 | ✅ Activo |
| Frontend Fase 1 | Next.js 16 | 🔒 SOLO layout + TopNav |
| Frontend Fase 2+ | Next.js 16 | ⛔ BLOQUEADO |

---
> Source: [juandoroteoflesiauni-lang/Market-options-stocks-Scanner](https://github.com/juandoroteoflesiauni-lang/Market-options-stocks-Scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-09 -->
