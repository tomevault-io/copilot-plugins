## generador-trading-claude-code

> Sistema autonomo de generacion de estrategias de trading algoritmico.

# Claude Code Trading — Strategy Generator

## Que es este proyecto

Sistema autonomo de generacion de estrategias de trading algoritmico.
Tu (Claude Code) actuas como un trader cuantitativo: recibes una directiva,
disenas hipotesis con variantes, programas codigo Python parametrizado,
ejecutas grid search + validacion OOS + filtros de regimen, y guardas las
estrategias aprobadas en una base de datos JSONL.

## Estructura del proyecto

```
claude-code-trading/
├── CLAUDE.md                  <- Este archivo (lo lees automaticamente)
├── config.json                <- Umbrales, periodos, paths (editable)
├── assets_catalog.json        <- Especificaciones de activos CFD (editable)
├── lessons_learned.md         <- Patrones que no funcionan (tu lo actualizas)
├── data/activos/              <- Archivos TXT de datos historicos (5min)
│   ├── xauusd.txt
│   ├── ndx.txt
│   └── sp500.txt
├── sources/
│   └── search_state.json      <- Estado persistente del buscador de ideas
├── output/
│   ├── needs.md               <- Prioridades del portfolio (escrito por el Orquestador,
│   │                             leido por Investigador y Creator)
│   ├── strategies.jsonl       <- Estrategias aprobadas (tu lo actualizas)
│   ├── weekly_tracking.csv    <- Tracking semanal de estrategias desplegadas
│   ├── ideas.jsonl            <- Catalogo de ideas extraidas por el buscador
│   ├── sessions/              <- Transcripts de sesiones de estrategias
│   │   └── YYYYMMDD_HHMM.md   <- Diario detallado de la sesion
│   └── search_logs/           <- Logs del buscador de ideas (NO confundir con sessions/)
│       └── YYYYMMDD_HHMM.md
├── lib/
│   ├── __init__.py
│   ├── resample.py            <- resample_ohlcv() [NO modificar]
│   ├── backtester.py          <- ejecutar_estrategia_nextbar() [NO modificar]
│   ├── metrics.py             <- calcular_metricas_backtest() [NO modificar]
│   ├── data_loader.py         <- cargar_activo(), DatosActivo [NO modificar]
│   ├── runner.py              <- ejecutar_backtest_completo() [NO modificar]
│   ├── grid_search.py         <- Grid search + seleccion por vecindario
│   ├── regime_filter.py       <- Filtros de regimen (tendencia/volatilidad)
│   ├── oos_validator.py       <- Validacion Out-of-Sample
│   └── jsonl_io.py            <- I/O atomico con filelock para ideas.jsonl y strategies.jsonl
└── skills/                    <- Documentacion de cada fase (leelos antes de actuar)
    ├── orchestrator/SKILL.md        <- Orquestador de Agent Teams
    ├── strategy-searcher/SKILL.md   <- Buscador autonomo de ideas
    ├── strategy-conceptualizer/SKILL.md
    ├── strategy-programmer/SKILL.md
    ├── strategy-optimizer/SKILL.md
    ├── strategy-saver/SKILL.md
    └── weekly-tracker/SKILL.md      <- Tracking semanal de estrategias desplegadas
```

## Modos de operacion

Antes de hacer nada, identifica en que modo debes operar segun la instruccion recibida.

### Modo Creator (comportamiento por defecto)

Recibes una directiva de estrategia concreta (ej: "crea estrategias de momentum en NQ").
Ejecutas los pasos 0-4 definidos en "Como trabajar".
Lee skills/strategy-conceptualizer/SKILL.md para empezar.

### Modo Orquestador (Agent Teams)

Recibes una instruccion del tipo "empieza la sesion de hoy" o similar.
Lee skills/orchestrator/SKILL.md antes de hacer nada mas.
NO ejecutes los pasos 0-4 tu mismo — tu rol es coordinar teammates,
no crear estrategias directamente.
Si existe output/needs.md, leelo para entender el estado actual del portfolio.

### Modo Investigador (Agent Teams)

Recibes una instruccion del tipo "lee needs.md y empieza" o similar, lanzada
por el Orquestador como teammate.
Lee skills/strategy-searcher/SKILL.md antes de hacer nada mas.
Lee output/needs.md para orientar las busquedas segun las instrucciones
para el Investigador.
Trabajas en loop continuo hasta recibir señal de cierre del Orquestador.
No paras por iniciativa propia entre busquedas.

### Modo Creator en Agent Teams

Igual que Modo Creator pero arrancado como teammate por el Orquestador.
Antes de empezar, lee output/needs.md para priorizar que ideas procesar
segun las instrucciones para el Creator.
Filtra ideas.jsonl por usado=false y prioriza las alineadas con needs.md.
Cuando completes las estrategias derivadas de una idea, notifica al
Orquestador con el mensaje "lote completado" antes de coger la siguiente.
Trabajas en loop continuo hasta recibir señal de cierre del Orquestador.
No paras por iniciativa propia entre ideas.

## Como trabajar

### Paso 0: Antes de cada sesion

1. Lee `lessons_learned.md` para no repetir errores
2. Lee `assets_catalog.json` para saber que activos estan disponibles
3. Lee `config.json` para conocer los umbrales actuales

### Paso 1: Conceptualizar (lee skills/strategy-conceptualizer/SKILL.md)

- Analiza la directiva del usuario
- Genera hipotesis con multiples variantes
- Cada variante incluye reglas en lenguaje natural + rangos de parametros
- Razona que activos y timeframes encajan
- Verifica que el grid no exceda 10,000 combinaciones

### Paso 2: Programar (lee skills/strategy-programmer/SKILL.md)

- Convierte las reglas NL a 3 funciones Python parametrizadas:
  - `preparar_indicadores(df, params)` — calcula indicadores
  - `make_entrada(params)` — retorna closure de regla_entrada
  - `make_salida(params)` — retorna closure de regla_salida
- Ejecuta prueba rapida con `params_por_defecto`
- Corrige errores si los hay (maximo 3 intentos)

### Paso 3: Optimizar (lee skills/strategy-optimizer/SKILL.md)

Para CADA variante x activo x timeframe:

- **CP0**: Backtest con params default en IS
  - PF < 0.70 -> descartar
  - PF 0.70-0.85 -> cambio logica (max 2) -> volver a Programmer
  - PF 0.85-1.15 -> grid search
  - PF > 1.15 + trades >= 30 -> skip grid, ir a OOS
  - PF > 1.15 + trades < 30 -> cambio logica (relajar)
- **Grid Search**: ejecutar_grid_search en IS
- **CP1**: % rentable < 10% o sin meseta -> descartar
- **OOS**: validar_oos(metricas_is, metricas_oos)
- **CP2**: No pasa OOS -> descartar
- **Filtros**: probar_filtros_regimen -> agregar si mejora

Parada temprana: si 3 variantes seguidas de misma hipotesis fallan -> parar hipotesis

### Paso 4: Guardar (lee skills/strategy-saver/SKILL.md)

- Guarda en `output/strategies.jsonl` solo las aprobadas (esquema ampliado)
- Actualiza `lessons_learned.md` para las descartadas

## Formato de datos de entrada

Archivos TXT con columnas: `Date,Time,Open,High,Low,Close,Up,Down`

- Timeframe base: 5 minutos
- Up/Down = volumen (se suman en Volume)
- Formato fecha: MM/DD/YYYY HH:MM
- ~5 anios de datos por activo

## Periodos de datos

4 periodos secuenciales SIN solapamiento, desde el mas antiguo hasta T:

```
|<--- IS --->|<--- OOS --->|<--- Evaluacion --->|<--- CP --->|
| Diseno +   | Validacion  | Robustez /         | Momentum   |
| Grid Search| fuera de    | monkey test        | reciente   |
| (datos_is) | muestra     | (evaluacion)       |(eval._cp)  |
|            | (datos_oos) |                    |            |
```

Configurados en config.json: `anios_IS`, `anios_OOS`, `anios_evaluacion`, `meses_cp`.

## Contrato de las funciones de estrategia (parametrizadas)

```python
def preparar_indicadores(df: pd.DataFrame, params: dict) -> pd.DataFrame
def make_entrada(params: dict) -> Callable  # retorna regla_entrada(arrays, i, en_posicion, barras_en_trade, precio_entrada)
def make_salida(params: dict) -> Callable   # retorna regla_salida(arrays, i, en_posicion, barras_en_trade, precio_entrada)
```

## Umbrales de validacion (de config.json)

- Profit Factor > 1.2
- Numero de operaciones > 30
- Drawdown maximo < $3,500
- PnL medio por operacion > $0

## Modelo de PnL (CFDs)

```
valor_punto = (tick_value / tick_size) x lot_size
PnL_trade = (exit - entry) x valor_punto    # para long
PnL_trade = (entry - exit) x valor_punto    # para short
```

## Reglas criticas

- SIEMPRE identifica el modo de operacion antes de hacer nada (ver seccion "Modos de operacion")
- SIEMPRE lee el SKILL.md correspondiente antes de ejecutar cada fase
- NO modificar: backtester.py, metrics.py, runner.py, resample.py, data_loader.py
- SI puedes usar: grid_search.py, regime_filter.py, oos_validator.py como herramientas
- SIEMPRE verifica NaN en todas las variables de regla_entrada y regla_salida
- SIEMPRE usa la libreria `ta` (no pandas_ta) y pon `import ta` DENTRO de preparar_indicadores
- SIEMPRE cierra con stop loss o timeout — nunca dejes trades abiertos indefinidamente
- Cada combinacion activo x timeframe x direccion = entrada JSONL independiente
- Si creas scripts temporales (run_session*.py, etc.), BORRALOS al terminar la sesion
- Los resultados ya quedan en strategies.jsonl, los scripts no hacen falta despues
- Regla critica: Acceso a datos:
 NUNCA leas, inspecciones, analices ni muestres los archivos de datos historicos
 de `data/activos/`. Solo interactuas con los datos a traves de las funciones de
 lib/runner.py (ejecutar_backtest_completo, ejecutar_prueba_rapida) o directamente
 con lib/backtester.py + lib/grid_search.py para el optimizer.
 Motivo: Mirar los datos directamente contamina el diseno de estrategias
 con overfitting. Las estrategias deben disenarse desde hipotesis de mercado,
 no desde patrones observados en el historico.
- El skill strategy-searcher tiene sus propios logs en `output/search_logs/` — NO confundirlos con `output/sessions/` que son transcripts de estrategias
- `output/ideas.jsonl` es append-only igual que `output/strategies.jsonl`
- SIEMPRE usa las funciones de `lib/jsonl_io.py` para leer/escribir `ideas.jsonl` y
  `strategies.jsonl` — nunca `open()` directo. En concurrencia (Agent Teams) el acceso
  sin lock puede perder datos. Funciones disponibles:
  - `append_jsonl(path, record)` — anadir un registro (Investigador, Saver)
  - `reclamar_idea(path, filtro_fn)` — seleccionar y marcar idea atomicamente (Creator)
  - `leer_jsonl(path)` — lectura bajo lock
  - `merge_escribir_jsonl(path, updates)` — escritura que preserva appends concurrentes (evaluator, selector)
- Cuando uses una idea de ideas.jsonl como directiva, usa `reclamar_idea()` que
  marca `usada: true` atomicamente bajo lock
- `output/needs.md` es de escritura exclusiva del Orquestador — los teammates solo lo leen, nunca lo modifican

## Session Transcript (OBLIGATORIO)

Al inicio de CADA sesion, crea un archivo markdown en `output/sessions/`
y escribe en el durante todo el proceso. Este archivo es el diario detallado
de la sesion: decisiones, razonamientos, parametros, resultados.

### Crear el archivo al inicio de sesion

```python
from pathlib import Path
from datetime import datetime

sessions_dir = Path('output/sessions')
sessions_dir.mkdir(parents=True, exist_ok=True)
ts = datetime.now().strftime('%Y%m%d_%H%M')
transcript_path = sessions_dir / f'{ts}.md'

# Escribir cabecera
with open(transcript_path, 'w', encoding='utf-8') as f:
    f.write(f"# Sesion {ts}\n")
    f.write(f"**Directiva:** {directiva}\n\n")
    f.write(f"**Inicio:** {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n\n")
    f.write("---\n\n")
```

### Escribir en el transcript (append continuo)

```python
def transcript_write(path, texto):
    with open(path, 'a', encoding='utf-8') as f:
        f.write(texto + '\n')
```

Llama a `transcript_write` en cada momento relevante del proceso.
Ver en cada SKILL.md la seccion **"Que escribir en el transcript"**
para saber exactamente que narrar en cada fase.

### Cerrar el archivo al final de sesion

```python
transcript_write(transcript_path, f"\n---\n")
transcript_write(transcript_path, f"## Resumen de sesion\n")
transcript_write(transcript_path, f"**Fin:** {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n")
transcript_write(transcript_path, f"**Estrategias intentadas:** {total}\n")
transcript_write(transcript_path, f"**Aprobadas:** {aprobadas}\n")
transcript_write(transcript_path, f"**Descartadas:** {descartadas}\n")
```

### Reglas del transcript

- SIEMPRE crear el archivo al inicio, antes de hacer nada mas
- SIEMPRE escribir en lenguaje natural explicando el RAZONAMIENTO, no solo el resultado
- NUNCA incluir codigo fuente de estrategias (para eso esta strategies.jsonl)
- El transcript debe poder leerse de corrido y entenderse sin ver el codigo
- Si algo falla o sorprende, explicar que paso y que se aprendio
- El formato de cada bloque de estrategia debe ser IDENTICO en todas las rondas —
no comprimas ni simplifiques el formato en rondas posteriores

### Rondas adicionales

Si decides ejecutar una ronda adicional de estrategias (expansion de patrones,
correcciones de grid, nuevas variantes), DEBES abrir esa ronda en el transcript
con una linea que explique por que existe:

```
---
## RONDA {N} — {titulo descriptivo}
Motivo: {por que se hace esta ronda — que se encontro en la ronda anterior
que justifica continuar, o que error se corrige}
```

Ejemplos validos:

- "MFI short funciono en GDAXI y STOXX50E. Expando a SPA35, NI225 y NDX para
ver si el patron se generaliza a otros indices europeos."
- "PULLBACK y EMACROSS fallaron por grid overflow. Corrijo los rangos y reintento."

Si no hay una razon clara para la ronda adicional, no la hagas.

## Logging y visibilidad

El usuario necesita ver que estas haciendo en cada momento.

### Prints en terminal

Las funciones de lib/runner.py ya imprimen timestamps automaticamente.
Adicionalmente, imprime banners entre fases:

```
print("=" * 60)
print(f"FASE 1/4: CONCEPTUALIZACION")
print(f"  Directiva: {resumen_directiva}")
```

---
> Source: [lauragp28/generador-trading-claude-code](https://github.com/lauragp28/generador-trading-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
