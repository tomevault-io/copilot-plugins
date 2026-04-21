## proyecto-tyasabi

> Plataforma de Inteligencia Comercial multi-área para TYASA.

# TYASA BI — Contexto para Claude Code

## Proyecto
Plataforma de Inteligencia Comercial multi-área para TYASA.
Stack: Python · Streamlit · BigQuery · Plotly · XGBoost · Statsmodels
GCP Project: project-d0cf2519-d089-47d3-930
Dataset BQ: tyasa_bi

## Estructura del repo
app.py                          ← Hub principal, navegación st.navigation()
config.py                       ← Colores, parámetros, rutas, áreas
core/
  db_connector.py               ← Cliente BigQuery singleton
  validators.py                 ← Validaciones de DataFrames
  components/
    charts.py                   ← Gráficos Plotly reutilizables (paleta TYASA)
    filters.py                  ← Filtros sidebar
    kpi_cards.py                ← Tarjetas KPI ejecutivas
    tables.py                   ← Tablas exportables a Excel
aceros_planos/
  negros/                       ← COMPLETADO (5 módulos)
    loaders.py                  ← Carga desde BQ con st.cache_data
    analytics/
      kpis.py                   ← KPIResumen dataclass
      segmentacion.py           ← Clasificación ABC, Pareto, HHI
      series_tiempo.py          ← Variación, volatilidad, heatmap
      forecasting.py            ← ETS | SARIMA | XGBoost | Naive | Auto
      mix_productos.py          ← Co-ocurrencia, cross-sell
  galvanizados/                 ← EN DESARROLLO
  formados/                     ← EN DESARROLLO
aceros_largos/                  ← PRÓXIMO (otro equipo)
aceros_sbq/                     ← PRÓXIMO (otro equipo)
pages/
  hub.py                        ← Bienvenida con estado del proyecto
  ap_negros/01-05_*.py          ← 5 páginas completas
  ap_galvanizados/coming_soon.py
  ap_formados/coming_soon.py

## Convenciones
- Naming tablas BQ: bronze_ | silver_ | gold_ + descripción
- Columnas clave: PESO_TON, CLIENTE, PRODUCTO_LIMPIO, PERIODO, AREA, DIVISION
- AREA_FILTER por loader: "NEGROS" | "GALVANIZADOS" | "FORMADOS"
- Paleta: primary=#1B3A5C | secondary=#4A7BA7 | success=#2E7D32 | danger=#C62828
- Todos los loaders usan @st.cache_data(ttl=600)
- Nuevas páginas siguen el patrón de pages/ap_negros/01_resumen.py

## Estado actual
- Aceros Planos Negros: CASI COMPLETO EN DESARROLLO (5 módulos)
- Aceros Planos Galvanizados: 🚧 EN DESARROLLO
- Aceros Planos Formados: 🚧 EN DESARROLLO
- Aceros Largos: 🔜 PRÓXIMO (otro equipo)
- Aceros SBQ: 🔜 PRÓXIMO (otro equipo)

## Reglas
1. Nunca hardcodear credenciales — usar get_bq_client()
2. Siempre usar table_ref() para referenciar tablas BQ
3. Nuevas páginas importan desde core/ y siguen estructura existente
4. Al agregar área nueva: actualizar config.py → AREAS y app.py → st.navigation()
5. Nuevos loaders van en aceros_planos/<subseccion>/loaders.py con AREA_FILTER propio
6. Cada página nueva tiene sidebar_header() + st.spinner() en carga de datos

## Notas de sesiones
Las notas de cada sesión se guardan en:
C:\Users\OTONIEL\Desktop\obsidian\sesiones\
Plantilla: templates/sesion-claude.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/otoniel378) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
