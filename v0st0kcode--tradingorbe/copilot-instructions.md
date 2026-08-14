## tradingorbe

> Sistema de análisis técnico con IA para operativa en Forex/XAU/Índices. Estrategia principal: CRT (Candle Range Theory) + SMC/Price Action + Wyckoff.

# TradingOrbe — Proyecto de Análisis de Trading

## Propósito
Sistema de análisis técnico con IA para operativa en Forex/XAU/Índices. Estrategia principal: CRT (Candle Range Theory) + SMC/Price Action + Wyckoff.

## Agente Principal
Usa el agente `trading-expert` para todo análisis. Invócalo con: "Usa el agente trading-expert para..."

## Estructura del Proyecto

```
TradingOrbe/
├── strategies/       # Documentación de estrategias
├── indicators/       # Pine Script: indicadores custom
├── analysis/         # Análisis guardados por sesión
└── notebooks/        # Referencias e índice de NotebookLM
```

## Herramientas MCP Disponibles

### TradingView MCP (`tradingview`)
Requiere TradingView Desktop con debug port activo:
```bash
open -a TradingView --args --remote-debugging-port=9222
```
Permite: leer estado del gráfico, OHLCV data, desarrollar Pine Script, dibujar niveles, crear alertas.

### NotebookLM MCP (`notebooklm-mcp`)
Ver `notebooks/index.md` para IDs de cuadernos de trading.

## Flujo de Trabajo Recomendado

1. **Análisis gráfico** → `chart_get_state` + `capture_screenshot` via TradingView MCP
2. **Consultar estrategia** → query al notebook CRT via NotebookLM MCP
3. **Documentar setup** → guardar en `analysis/YYYY-MM-DD-SYMBOL.md`
4. **Crear indicador** → Pine Script en `indicators/` + `pine_set_source` en TradingView

## Instrumentos Principales
- **FX**: EURUSD, GBPUSD, USDJPY, XAUUSD
- **Sesiones**: Londres (08:00-12:00 CET), NY (14:30-17:00 CET)
- **Timeframes**: D1 (bias) → H4/H1 (estructura) → M15/M5 (entrada)

## Gestión de Riesgo
- Riesgo por operación: 0.5%–1% (challenge), 1%–2% (funded)
- R:R mínimo: 1:2
- Máximo 3 operaciones activas simultáneas

---
> Source: [v0st0kCode/TradingOrbe](https://github.com/v0st0kCode/TradingOrbe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
