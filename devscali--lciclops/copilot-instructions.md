## lciclops

> Antes de hacer cualquier operación que pueda afectar datos en producción:

# CLAUDE.md - Base de Conocimiento del Proyecto

## Advertencias Importantes

### SIEMPRE ANTES DE CAMBIOS DESTRUCTIVOS
Antes de hacer cualquier operación que pueda afectar datos en producción:
- Migraciones de base de datos
- Cambios en modelos/schemas
- DELETE, UPDATE masivos
- Cambios en estructura de tablas
- Deploy de cambios grandes

**ADVERTIR AL USUARIO:** "¿Quieres que corra un backup antes de entrar a los vergazos?"

### Sistema de Backups
- Backups automáticos diarios a las 3:00 AM UTC (9:00 PM México)
- Ubicación: `backups/` en el repo
- Formato: `db_YYYY-MM-DD_HH-MM-SS.sql.gz`
- Retención: 14 días
- Restaurar: `./scripts/restore.sh --timestamp YYYY-MM-DD_HH-MM-SS`
- Backup manual: Ejecutar workflow desde GitHub Actions

## Stack del Proyecto

- **Backend:** FastAPI + Python
- **Base de datos:** PostgreSQL (Railway)
- **AI:** Anthropic Claude / OpenAI (fallback)
- **Deploy:** Railway
- **Dominio:** lc.calidevs.com

## Archivos Clave

- `app/main_postgres.py` - Entry point de la API (incluye SmartDocumentProcessor)
- `app/database.py` - Conexión a PostgreSQL
- `app/db_models.py` - Modelos SQLAlchemy
- `index.html` - Dashboard frontend con 19 gráficas
- `railway.json` - Config de deploy

## Estado Actual del Sistema (Marzo 2026)

### Datos en Producción
- **16 documentos procesados:** P1-P13 (Estados de Resultados 2025 completos)
- **17 tiendas** con datos financieros
- **221 registros** en `monthly_summaries` (17 tiendas × 13 períodos)
- **Dashboard funcional** con datos reales verificados
- **Cascada financiera verificada** - Waterfall cuadra $0 gap en todos los períodos
- **MANEADERO** sin datos P1-P9 (abrió después)

### Tiendas Activas
```
GUAYMITAS, CABOS, MI PLAZA, VISTA, CAMINO, MANEADERO, ´5 FEB,
CONSTITUCION, FORJADORES, DELANTE, OLIVARES, SANTA FE, PASEO,
SOL, TAMARAL, SAN MARINO, CENTRO
```

### Períodos Fiscales Little Caesars
- 13 períodos por año (P1-P13)
- Cada período = 4 semanas
- Semana empieza Martes, termina Lunes
- P13 = Diciembre (semanas 49-52)

## SmartDocumentProcessor (AI-Powered)

Sistema inteligente que procesa CUALQUIER documento automáticamente.

### Qué hace:
1. **Detecta tipo de documento** - estado_resultados, estado_cuenta, nomina, inventario
2. **Extrae período del CONTENIDO** - no solo del nombre de archivo
3. **Identifica tiendas** - automáticamente de los datos
4. **Mapea conceptos** - NOMINAS, ALQUILER, CFE → categorías estándar
5. **Guarda en BD** - `monthly_summaries` listo para gráficas

### Conceptos que entiende:
```
ingresos     → INGRESOS, VENTAS, REVENUE, TOTAL VENTAS
costo_ventas → Se calcula desde UTILIDAD BRUTA (ver abajo)
nomina       → NOMINAS, AGUINALDOS, IMSS, BONOS, VACACIONES
renta        → ALQUILER, MTTO ALQUILER, ARRENDAMIENTO
servicios    → CFE, AGUA, GAS, TELMEX, INTERNET
utilidad     → UTILIDAD BRUTA, UTILIDAD NETA, GANANCIA
```

### Lógica de Extracción Financiera (IMPORTANTE)

La función `extract_financial_data_to_summaries()` en `main_postgres.py:~1702` extrae datos del Excel:

```python
# 1. COSTO DE VENTAS - Se calcula desde UTILIDAD BRUTA (más confiable)
cost_of_sales = total_sales - gross_profit_from_excel

# 2. GASTOS OPERATIVOS - Se resta el costo de ventas del TOTAL EGRESOS
operating_expenses = TOTAL_EGRESOS - cost_of_sales

# 3. VALIDACIÓN - Si no cuadra, se recalcula
expected_op = gross_profit - net_profit
if operating_expenses > expected_op * 1.2:
    operating_expenses = expected_op  # Forzar que cuadre
```

**IMPORTANTE - Hojas duplicadas (fix 2026-03-10):**
Los Excel tienen 2 hojas con datos idénticos: "EDO RESUL C INV" y "EDO RESUL S INV".
La extracción SOLO procesa la primera hoja (filtro por `_hoja_origen`).
Sin esto, nómina/renta/servicios se duplican porque usan `+=`.

**Cascada que debe cuadrar:**
```
Ventas - Costo Venta = Utilidad Bruta
Utilidad Bruta - Gastos Op = Utilidad Neta
```

**Métricas esperadas (Little Caesars):**
- Margen bruto: 20-30%
- Margen neto: 0-10%
- Ratio nómina: 15-22% de ventas (verificado con 13 períodos reales)
- Ratio renta: 2-3% de ventas (verificado con 13 períodos reales)

### Endpoints de Procesamiento:
```bash
# Procesar 1 documento con AI
POST /process/smart/{doc_id}

# Procesar todos los documentos en batch
POST /process/smart-batch

# Preview sin guardar (para validar)
GET /process/smart-preview/{doc_id}

# Confirmación con smart processing (default)
POST /upload/confirm?smart_process=true
```

## Dashboard y Gráficas

### 19 Gráficas Disponibles:
| Gráfica | Descripción | Datos |
|---------|-------------|-------|
| revenueChart | Ingresos/Utilidad por tienda | ✅ Real |
| expenseChart | Desglose de gastos | ✅ Real |
| trendChart | Tendencia P11→P13 | ✅ Real |
| rankingChart | Ranking por margen | ✅ Real |
| correlationChart | Ventas vs Gastos | ✅ Real |
| expenseDistChart | Distribución % | ✅ Real |
| compareChart | Actual vs Anterior | ✅ Real |
| topCategoriesChart | Top gastos | ✅ Real |
| radarChart | Multidimensional | ✅ Real |
| efficiencyChart | Gauge eficiencia | ✅ Real |
| marginCompareChart | Margen bruto/neto | ✅ Real |
| laborRatioChart | Ratio nómina | ✅ Real |
| fixedCostsChart | Costos fijos | ✅ Real |
| costStructureChart | Estructura costos | ✅ Real |
| marginEvolutionChart | Evolución márgenes | ✅ Real |
| waterfallChart | Cascada P&L | ✅ Real |
| scatterMarginChart | Dispersión margen | ✅ Real |
| rentRatioChart | Ratio renta | ✅ Real |
| momVariationChart | Variación MoM | ✅ Real |

### Modales
Todas las gráficas tienen modal con datos reales del API (no demo data).

## Troubleshooting: Nuevos Datos

### Si las gráficas muestran datos incorrectos:

1. **Verificar cascada financiera:**
   ```bash
   # En el dashboard, el waterfall debe cuadrar:
   # Ventas - Costo Venta = Ut. Bruta ✓
   # Ut. Bruta - Gastos Op = Ut. Neta ✓
   ```

2. **Si hay duplicados (más de 17 tiendas por período):**
   ```bash
   # Reprocesar con limpieza:
   curl -X POST "https://lc.calidevs.com/process/reprocess-all?clean=true"
   ```

3. **Si el margen bruto es muy bajo (<10%) o muy alto (>60%):**
   - Verificar que el Excel tenga línea "UTILIDAD BRUTA"
   - Si no existe, el sistema suma items individuales (menos preciso)

4. **Si gastos operativos son 0 o muy altos:**
   - El sistema valida: `gastos_op = utilidad_bruta - utilidad_neta`
   - Si TOTAL EGRESOS del Excel no cuadra, se recalcula automáticamente

5. **Si nómina/renta/servicios aparecen duplicados (ratio nómina >30%):**
   - Probablemente se procesaron ambas hojas del Excel
   - Verificar que el fix de `_hoja_origen` esté activo en `extract_financial_data_to_summaries()`
   - Reprocesar: `curl -X POST "https://lc.calidevs.com/process/reprocess-all?clean=true"`

### Endpoint temporal para reprocesar:
```bash
POST /process/reprocess-all?clean=true
# NOTA: Sin auth, borrar después de usar en producción
```

## Bugs Históricos

### 2026-03-10: Nómina/Renta/Servicios duplicados (commit 9683df1)
- **Causa:** Excel tiene 2 hojas idénticas (C INV, S INV). Items con `+=` se duplicaban.
- **Fix:** Filtro por `_hoja_origen` + safety net en `INGRESOS` duplicado.
- **Verificación:** Ratio nómina debe ser 15-22%, NO 30-45%.

### 2026-03-15: Gráficas mostraban solo 6-7 tiendas (commit 3b93729)
- **Causa:** Límites hardcodeados: `[:10]` en backend, `.slice(0,7)` en 3 HTMLs.
- **Fix:** Eliminados todos los límites. Ahora las 17 tiendas aparecen dinámicamente.
- **Archivos:** `main_postgres.py`, `index.html`, `graficas.html`, `reports.html`.

## Seguridad (Actualizado 2026-03-23)

**Fixes aplicados:**
- `/process/reprocess-all` ahora requiere admin auth
- JWT secret key sin fallback hardcodeado (RuntimeError si no hay env var en prod)
- Print statements de secrets eliminados
- Auth re-habilitado en TODOS los endpoints de datos (8 endpoints)
- Auth re-habilitado en TODOS los HTML files (9 archivos)
- XSS fix en julia.html (formatMessage sanitiza HTML)
- Swagger/docs deshabilitados en producción
- CORS localhost solo en desarrollo
- require_admin acepta "superadmin"
- Login route sirve login.html (no redirect a dashboard)
- Password mínimo 8 caracteres en schemas
- Backup workflow: fix pg_dump stderr + validación tamaño mínimo

**Env vars REQUERIDAS en Railway:**
- `JWT_SECRET_KEY` — OBLIGATORIA (app no arranca sin ella)
- `LC_ACCESS_CODE` — Código de acceso para clientes LC
- `RAILWAY_ENVIRONMENT` — Detecta producción (para CORS, docs)

## Pendientes (Marzo 2026)

1. **PDFs Estados de Cuenta** - El SmartProcessor ya está preparado
2. ~~**Quitar endpoint /process/reprocess-all**~~ - RESUELTO: Ahora requiere auth admin
3. ~~**Endpoints sin auth**~~ - RESUELTO: Auth habilitado en todos
4. **Limpiar `lciclops_old`** - Backup del repo viejo, ya no se necesita
5. **Verificar DATABASE_URL en GitHub Secrets** - Debe ser URL PÚBLICA de Railway (no railway.internal)
6. **Configurar LC_ACCESS_CODE en Railway** - Ya no está hardcodeado en código

## Comandos Útiles

```bash
# Ver backups disponibles
./scripts/restore.sh --list

# Restaurar backup específico
./scripts/restore.sh --timestamp 2026-01-25_01-05-02

# Backup manual inmediato
gh workflow run backup.yml --repo devscali/lciclops

# Variables de Railway
railway variables

# Verificar sintaxis Python
python3 -m py_compile app/main_postgres.py

# Ver logs de Railway
railway logs
```

## Flujo de Datos

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────────┐     ┌──────────────┐
│   Cliente   │────▶│  SmartProcessor  │────▶│ monthly_        │────▶│  Dashboard   │
│  sube Excel │     │  (Claude/GPT-4)  │     │ summaries (BD)  │     │  (19 charts) │
│  o PDF      │     │                  │     │                 │     │              │
└─────────────┘     └──────────────────┘     └─────────────────┘     └──────────────┘
                           │
                           ▼
                    ┌──────────────────┐
                    │ Detecta:         │
                    │ • Tipo documento │
                    │ • Período fiscal │
                    │ • Tiendas        │
                    │ • Conceptos      │
                    └──────────────────┘
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devscali) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
