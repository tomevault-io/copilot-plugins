## mol

> **⛔ ANTES DE DEPLOYEAR: leer `DEPLOY_RULES.md` — producción es solo para Gerardo.**

# MOL - Monitor de Ofertas Laborales

## LEER PRIMERO: Estado Actual

**⛔ ANTES DE DEPLOYEAR: leer `DEPLOY_RULES.md` — producción es solo para Gerardo.**

**ANTES DE HACER CUALQUIER COSA, leer `.ai/learnings.yaml` para el estado actual del trabajo.**

El archivo `learnings.yaml` contiene:
- `current_state`: Qué estamos haciendo AHORA (ofertas, versiones, próximo paso)
- `ultimo_trabajo`: Qué se hizo en la última sesión
- `problemas_conocidos`: Issues actuales
- **`conteos`**: SINGLE SOURCE OF TRUTH para cantidades (reglas, skills, etc.)

**Comando del usuario: "Guardá estado"** → Ejecutar `python scripts/sync_learnings.py` + agregar notas en ultimo_trabajo si es necesario.

**AUTO-SYNC + REPORTE DE FASES (v2.0):** Al iniciar sesión, Claude recibe automáticamente:
- **Reporte de las 3 fases** con métricas actuales (ofertas, NLP, matching, validación)
- **Sugerencia de fase** basada en qué necesita atención (errores, pendientes, etc.)
- **Conteos actualizados** desde configs y BD

Triggers:
- **Al iniciar sesión** → Hook SessionStart (`.claude/settings.json`)
- Al ejecutar pipeline → `sync_learnings_yaml()` al final
- Manual: `python scripts/sync_learnings.py` (con `--human` para formato detallado)

---

## PENDIENTE CRITICO: Rotar Supabase service_role_key (S-01)

**La `service_role_key` actual está comprometida** (existe en el historial de git). El código ya fue limpiado (commit `9f904093`) pero la key vieja sigue siendo válida.

**Pasos a realizar (manual):**
1. Ir a **Supabase Dashboard → Settings → API** → regenerar `service_role_key`
2. Actualizar `config/supabase_config.json` local con la nueva key
3. Actualizar `SUPABASE_SERVICE_ROLE_KEY` en **Vercel** (si se usa)
4. Verificar que `sync_to_supabase.py` y `generate_mol_skills_profile.py` sigan funcionando
5. **Borrar esta sección de CLAUDE.md** una vez completado

---

## Descripcion
Sistema de monitoreo del mercado laboral argentino para OEDE. Scrapea ofertas de empleo, extrae informacion con NLP, clasifica segun taxonomia ESCO, y provee dashboards para analistas.

## Estado Actual

> **CONTEOS OFICIALES:** Ver `.ai/learnings.yaml` sección `conteos` (single source of truth)

- **NLP v11.4** (20 campos + source-aware pre-fill + postprocessor + qwen2.5:7b)
- **NLP Gate v1.1** (35+ reglas pre-matching, bloquea critico/alto)
- **Multi-Position Detection** (regex + LLM, crea sub-ofertas)
- **Matching v3.5.4 ESCO-FIRST** - ESCO es target, ISCO se deriva
- **Skills v2.5** - BGE-M3 base (LoRA fine-tuned NO disponible — model_lora no existe en disco, umbral 0.40)
- **Canonización planificada** - Reducir 14,247 skills ESCO a ~3,000 canónicas (ver `docs/plan/14_CANONIZACION_SKILLS_TAREAS.md`)
- **Pipeline v3.3** (8 pasos integrados con NLP Gate + multi-position)
- **Fine-tuning data** (llm_raw_json + postprocessor_diff + valor_actual/corregido)
- **Conteos dinámicos** (ver `learnings.yaml`): reglas_negocio, reglas_validacion, sinonimos_argentinos
- **Auto-sync** de learnings.yaml activado (v2.1)

### Matching v3.5.4 ESCO-First (2026-03-15)
```
PRINCIPIO: ESCO es el TARGET, ISCO es CONSECUENCIA

FLUJO DE PRIORIDAD:
1. REGLAS DE NEGOCIO (si aplican) → GANAN SIEMPRE
   - Buscan ocupación ESCO por label exacto
   - ISCO se deriva de la ocupación encontrada
   - titulo_original_contiene_alguno (v3.5.4: busca en título SIN limpiar)

2. DICCIONARIO ARGENTINO (si no hay regla)
   - Mapea términos argentinos a ISCO

3. SEMÁNTICO (fallback)
   - Skills + título embeddings (LoRA fine-tuned model)

METADATA GUARDADA:
- isco_semantico, score_semantico (siempre calculado)
- isco_regla, regla_aplicada (si aplica regla)
- dual_coinciden: 1=mismo, 0=difieren, NULL=solo semántico
- decision_metodo: "regla_prioridad" | "semantico_default"
```

### Fábrica de Procesamiento (Bloques I+G — 2026-03-22)

El procesamiento se organiza como una fábrica con dos líneas:

**LÍNEA 1 — FABRICACIÓN** (producir datos clasificados):
```
Scraping → NLP v11.4 → Gate NLP (35+ reglas) → Matching v3.5.4 → Gate Matching → Validación → Sync
```

**LÍNEA 2 — MEJORA CONTINUA** (mejorar la fábrica):
```
Errores → Issues → Training Pairs (602+) → Fine-tuning → Catálogo MOL → Perfil Argentino
```

Correcciones de L1 alimentan L2. Mejoras de L2 vuelven como reglas y modelos a L1.

**3 actores:** Pipeline (automático), Claude (operario calificado), Humano (gerente + QA)

**Admin UI** (`/admin/procesamiento/`):
- **Fábrica**: vista dual con control por nodo (botones ejecutar/reprocesar/configurar por etapa)
- **Diccionarios**: 6 configs editables en tabs (reglas, NLP, sinónimos, oficios, skills, limpieza)
- **Catálogo MOL**: curación skills/ocupaciones argentinas (detectada → revisión → catalogada)
- **Perfil Argentino**: publicación versionada del perfil consolidado
- **Validación**: estación del analista (OK/Error/Revisar/Basura + wizard + auto-issues)

**Gateway local**: tabla `pipeline_commands` en Supabase + poller local (mismo patrón que scraping_commands). Admin da órdenes desde UI sin terminal ni Claude.

**Configs editables desde UI** (override Supabase → fallback JSON local):
6 configs via `load_config()` en pipeline Python: matching_rules_business, nlp_inference_rules, sinonimos_argentinos_esco, skills_rules, oficios_arg, nlp_titulo_limpieza.

> Diseño completo: `docs/plan/03_WIREFRAMES/fabrica-procesamiento.md`

### Trabajo en Curso
- ~4.6K ofertas pendientes NLP (requiere Ollama)
- 15,968 ofertas validadas en BD
- 602 training pairs acumulados (103 ISCOs, 9/10 grupos)
- Gold Set de referencia: 49 casos (archivo histórico)

### Sistema de Priorización v1.0 (2026-01-20)

El pipeline procesa ofertas **por prioridad**, no por orden de scraping.

**Criterios de scoring:**
| Criterio | Peso | Lógica |
|----------|------|--------|
| Fecha publicación | 40% | Más reciente = mayor score |
| Cantidad vacantes | 30% | Más vacantes = mayor impacto |
| Permanencia | 30% | Baja permanencia = alta demanda |

**Tabla:** `ofertas_prioridad` (estados: pendiente → en_proceso → procesado)

**Comandos:**
```bash
# Ver estado de la cola
python scripts/get_priority_batch.py --queue-status

# Procesar lote de 100 por prioridad
python scripts/run_validated_pipeline.py --limit 100

# Ver próximo lote sin procesar
python scripts/get_priority_batch.py --size 100
```

**Bloqueo por errores:** El sistema **NO avanza** al siguiente lote si hay errores pendientes.
```
Lote 1 → 5 errores → [BLOQUEADO] → Resolver errores → [DESBLOQUEADO] → Lote 2
```

Para forzar (NO recomendado): `--force-new-batch`

---

## Modelo de 3 Fases

El proyecto se organiza en 3 macro-fases independientes:

| Fase | Descripcion | Ubicacion Principal | Salida |
|------|-------------|---------------------|--------|
| 1. Adquisicion | Scraping, deteccion bajas | `01_sources/`, `run_scheduler.py` | BD cruda |
| 2. Procesamiento | NLP, Skills, Matching, **Validacion** | `database/`, `config/`, `export_validation_excel.py` | Excel validacion + datos validados |
| 3. Presentacion | Dashboard (solo validados) | `fase3_dashboard/`, `Visual--/`, `sync_to_supabase.py` | Dashboard usuarios finales |

**Para trabajar en una fase especifica**, indicar en `learnings.yaml`:
```yaml
fase_actual: "procesamiento"  # adquisicion | procesamiento | presentacion
```

**REGLA:** Si modificas el pipeline de una fase, actualizar `docs/reference/ARQUITECTURA_3_FASES.md`.

> Arquitectura completa: `docs/reference/ARQUITECTURA_3_FASES.md`

---

## Deploy Dashboard (Fase 3)

**⚠️ ALIASES DE DEPLOY — NO CONFUNDIR:**

| Alias | Quién deploya | Para qué |
|-------|---------------|----------|
| `mol-nextjs.vercel.app` | **Solo Gerardo** | Producción |
| `mol-dev.vercel.app` | **Solo Sergio** | Desarrollo frontend |

**NUNCA** hacer `vercel alias ... mol-nextjs.vercel.app` si no sos Gerardo.
**NUNCA** hacer `vercel alias ... mol-dev.vercel.app` si no sos Sergio.

**Arquitectura:**
```
fase3_dashboard/mol-dashboard/  →  Vercel (mol-dashboard)
         ↓
    Gerardo deploya → mol-nextjs.vercel.app (producción)
    Sergio deploya  → mol-dev.vercel.app (desarrollo)
```

**IMPORTANTE:** El deploy NO es automático. No está vinculado a GitHub por limitaciones del plan gratuito de Vercel.

### Flujo de trabajo

1. **Desarrollar** en localhost:
   ```bash
   cd fase3_dashboard/mol-dashboard
   npm run dev
   # Abrir http://localhost:3000
   ```

2. **Cuando está listo** → decirle a Claude: "commitear y deployar"

3. **Claude ejecuta (verificar qué alias usar según el desarrollador):**
   ```bash
   # Regenerar architecture JSON (si cambiaron pantallas o pipeline)
   python scripts/generate_architecture_json.py

   # Commit a GitHub
   git add fase3_dashboard/mol-dashboard/
   git commit -m "feat(dashboard): descripción del cambio"
   git push origin main

   # Deploy a Vercel
   cd fase3_dashboard/mol-dashboard
   npx vercel --prod --yes

   # IMPORTANTE: Usar el alias correcto según quién deploya
   # Gerardo:
   npx vercel alias [url-del-deploy] mol-nextjs.vercel.app
   # Sergio:
   npx vercel alias [url-del-deploy] mol-dev.vercel.app
   ```

   **Nota:** Sin el alias, la URL queda solo en el deploy temporal.

### Comandos útiles Vercel

```bash
# Ver proyectos
npx vercel ls

# Deploy a producción
npx vercel --prod --yes

# Ver logs de un deploy
npx vercel logs [url-del-deploy]

# Login (si expira la sesión)
npx vercel login
```

### Variables de entorno (ya configuradas)

| Variable | Valor |
|----------|-------|
| NEXT_PUBLIC_SUPABASE_URL | https://uywzoyhjjofsvvsrrnek.supabase.co |
| NEXT_PUBLIC_SUPABASE_ANON_KEY | (configurada en Vercel) |

---

## Gestión de Issues (Supabase)

Los issues/feedback de usuarios están en la tabla `issues` de Supabase. La **anon key NO tiene permisos de UPDATE** (RLS). Usar siempre el **service role key** via Python.

### Actualizar estado de un issue

```python
python3 -c "
import json
from supabase import create_client

config = json.load(open('config/supabase_config.json'))
client = create_client(config['url'], config['service_role_key'])

client.table('issues').update({
    'estado': 'resuelto',
    'resuelto_at': '2026-02-12T00:00:00Z',
    'solucion_aplicada': 'Descripción de lo que se hizo',
    'config_modificada': 'archivos modificados'
}).eq('id', 'UUID-DEL-ISSUE').execute()
"
```

### Listar issues pendientes

```python
python3 -c "
import json
from supabase import create_client

config = json.load(open('config/supabase_config.json'))
client = create_client(config['url'], config['service_role_key'])

result = client.table('issues').select('id,titulo,estado,prioridad').in_('estado', ['pendiente','en_progreso']).order('created_at', desc=True).execute()
for i in result.data:
    print(f\"{i['id'][:8]} | {i['estado']:12} | {i['prioridad']:6} | {i['titulo']}\")
"
```

### Campos actualizables

| Campo | Tipo | Cuándo |
|-------|------|--------|
| `estado` | `pendiente` / `en_progreso` / `resuelto` / `descartado` | Siempre al cambiar estado |
| `resuelto_at` | ISO timestamp | Al marcar resuelto |
| `solucion_aplicada` | string | Al marcar resuelto |
| `config_modificada` | string | Si se modificaron archivos |
| `ofertas_afectadas` | number | Si aplica |
| `sprint` | string | Para tracking |

**Config:** `config/supabase_config.json` (contiene `url` + `service_role_key`)

---

## Documentación Extendida

### Planificación del Producto (FUENTE DE VERDAD)

**IMPORTANTE:** Toda la planificación del producto está en `docs/plan/`. Es la ÚNICA fuente válida para diseño, roadmap y features.

| Documento | Contenido | Cuándo leer |
|-----------|-----------|-------------|
| **`docs/plan/INDEX.md`** | Mapa completo + estado actual | **SIEMPRE al planificar** |
| `docs/plan/01_MODELO_NEGOCIO.md` | Usuarios, niveles, pricing | Definir accesos y planes |
| `docs/plan/02_ARQUITECTURA_PANTALLAS.md` | 30 pantallas (P-01 a P-30) | Agregar/modificar rutas |
| `docs/plan/03_WIREFRAMES/` | Wireframes por área | Diseñar UI |
| `docs/plan/04_MODELO_DATOS.md` | Tablas SQL, schema | Crear/modificar tablas |
| `docs/plan/05_USER_FLOWS.md` | Flujos F-01 a F-05 | Implementar journeys |
| `docs/plan/06_SEGURIDAD.md` | Issues S-01 a S-17 (4 CRITICOS) | **Fase 0 BLOQUEANTE** |
| `docs/plan/07_ESCALABILIDAD.md` | Issues E-01 a E-15 | Performance, cache, índices |
| `docs/plan/08_PROPUESTA_VALOR.md` | Features V-01 a V-15 | Qué falta vs competidores |
| `docs/plan/09_ROADMAP.md` | Fases 0-4 + dependencias | Priorizar trabajo |
| `docs/plan/10_OBSERVABILIDAD.md` | Monitoreo, métricas, pipeline | Admin dashboard |
| `docs/plan/11_CONFIGURACION_ADMIN.md` | /admin/configuracion | Panel admin |
| `docs/plan/12_INSIGHTS_SISTEMA.md` | Performance + ubicación | Insights SQL |
| `docs/plan/13_LABORATORIO_INDICADORES.md` | Indicadores experimentales, ciclo de vida | Agregar/priorizar indicadores |
| `docs/plan/ANALISIS_ISSUES_USUARIO.md` | Feedback usuario, sprints | Issues reportados |

### Documentación Técnica (Pipeline)

| Tema | Documento | Cuándo leer |
|------|-----------|-------------|
| **Arquitectura 3 Fases** | `docs/reference/ARQUITECTURA_3_FASES.md` | Entender macro-estructura |
| **Colaboracion (multi-dev)** | `docs/guides/COLABORACION.md` | Trabajo en equipo, sync git |
| Pipeline completo (5 etapas) | `docs/reference/PIPELINE.md` | Entender flujo de datos |
| Run Tracking y Versionado | `docs/guides/RUN_TRACKING.md` | Comparar corridas |
| Sistema de Validación | `docs/guides/VALIDACION.md` | Estados, protección datos |
| Flujos de Optimización | `docs/guides/OPTIMIZACION.md` | Corregir errores NLP/Matching |
| Sincronización Supabase | `docs/guides/SUPABASE_SYNC.md` | Subir datos al dashboard |

**IMPORTANTE:** Antes de optimizar el pipeline, LEER `docs/guides/OPTIMIZACION.md`.

---

## Flujo de Optimización → Validación Humana (v2.2)

El sistema tiene dos fases separadas:

```
FASE 1: OPTIMIZACIÓN (Claude)     FASE 2: VALIDACIÓN (Humano)
─────────────────────────────     ──────────────────────────
Claude itera:                     Solo cuando converge:
- Procesa lote                    - Recibe Excel
- Detecta errores                 - Revisa en Google Sheets
- Crea reglas en JSONs            - Marca OK/ERROR
- Reprocesa                       - Devuelve feedback
- Repite hasta tasa < 5%          - Aprueba o rechaza
```

### Estados de un Lote

| Estado | Descripción | Siguiente acción |
|--------|-------------|------------------|
| `optimizacion` | Claude iterando | Procesar, detectar errores, crear reglas |
| `listo_validacion` | Tasa < 5% (convergido) | Enviar a humano |
| `en_validacion` | Humano revisando | Esperar feedback |
| `validado` | Humano aprobó | Listo para producción |
| `rechazado` | Humano pidió más trabajo | Reabrir y continuar |

### Comandos del Flujo

```python
from scripts.run_tracking import RunTracker
tracker = RunTracker()

# 1. Crear lote
lote_id = tracker.create_batch("Lote 100 ofertas", offer_ids=[...])

# 2. Iterar (Claude optimiza)
while True:
    stats = run_matching_pipeline(offer_ids, source="optimizacion")
    tracker.add_run_to_batch(lote_id, stats['run_id'])

    result = tracker.check_convergence(lote_id)
    if result['convergido']:
        print(f"CONVERGIDO: Tasa {result['tasa']}% < 5%")
        break
    # ... detectar errores, crear reglas, reprocesar ...

# 3. Enviar a humano
tracker.send_to_human_validation(lote_id)  # Genera Excel

# 4. Después del feedback humano
tracker.complete_human_validation(lote_id, aprobado=True)  # o False

# Si rechazado, reabrir
tracker.reopen_batch_for_optimization(lote_id)
```

### Visualizar Estado

```bash
python scripts/show_learning_evolution.py --batches
```

---

## Pipeline Issue → Training Pairs → Fine-Tuning

**CRÍTICO:** Los issues de usuarios (Supabase) alimentan un dataset de entrenamiento para futuro fine-tuning del modelo de clasificación ESCO. Cada issue resuelto es un dato de entrenamiento valioso.

### Arquitectura

```
Usuario reporta issue (dashboard)
    → Supabase tabla `issues` (estado: pendiente)
        → Claude/dev resuelve (modifica configs, reprocesa)
            → Issue pasa a `resuelto` con solucion_aplicada
                → sync_learnings.py se ejecuta
                    → generate_training_pairs.py (auto-trigger)
                        → config/training_pairs.json (dataset acumulado)
```

### Componentes

| Componente | Ubicación | Función |
|------------|-----------|---------|
| Generador | `scripts/exports/generate_training_pairs.py` | Convierte issues resueltos en pares de entrenamiento |
| Dataset | `config/training_pairs.json` | Pares acumulados (583+ al 2026-02) |
| Auto-trigger | `scripts/sync_learnings.py` | Ejecuta generador automáticamente |

### Estructura de un Training Pair

```json
{
  "input": {
    "titulo": "Peón rural",
    "descripcion": "...",
    "tareas": "...",
    "sector": "...",
    "area_funcional": "..."
  },
  "clasificacion_incorrecta": {"isco": "0110", "label": "Oficial de las fuerzas armadas"},
  "clasificacion_correcta": {"isco": "6111", "label": "Agricultor"},
  "justificacion_humana": "Descripción del usuario sobre por qué es incorrecto"
}
```

### 3 Enfoques de Fine-Tuning Soportados

| Enfoque | Datos que usa | Para qué |
|---------|---------------|----------|
| **Supervised** | input → clasificación_correcta | Entrenamiento directo |
| **DPO/RLHF** | correcto=chosen, incorrecto=rejected | Preferencia de pares |
| **Chain-of-Thought** | input + justificación_humana → correcta | Razonamiento paso a paso |

### Comandos

```bash
# Generar training pairs manualmente
python scripts/exports/generate_training_pairs.py

# Ver estadísticas del dataset
python scripts/exports/generate_training_pairs.py --stats

# Solo desde cierta fecha
python scripts/exports/generate_training_pairs.py --since 2026-03-01

# Dry run (no guarda)
python scripts/exports/generate_training_pairs.py --dry-run
```

### Flujo de Trabajo con Issues de Usuarios

```
PASO 1: Listar issues pendientes
─────────────────────────────────
python -c "..." (ver sección Gestión de Issues)

PASO 2: Por cada issue, ver qué corrección pide
────────────────────────────────────────────────
- Leer descripcion, valor_actual, valor_esperado
- Identificar si es error de NLP, Matching, o clasificación

PASO 3: Aplicar corrección
──────────────────────────
- Crear/modificar regla en config/*.json correspondiente
- Reprocesar oferta: python scripts/run_validated_pipeline.py --ids X

PASO 4: Marcar issue como resuelto
───────────────────────────────────
- Actualizar en Supabase con solucion_aplicada y config_modificada

PASO 5: Sync → Training pair se genera automáticamente
──────────────────────────────────────────────────────
python scripts/sync_learnings.py
```

**IMPORTANTE:** Cada issue resuelto con buena justificación = mejor dato de entrenamiento.
Los issues de múltiples aspectos de la misma oferta se deduplicany y mergean justificaciones.

---

## ⛔ PROHIBIDO IMPROVISAR - FLUJO OBLIGATORIO

**Claude: ANTES de ejecutar CUALQUIER comando, verificar este checklist:**

```
□ 1. ¿Existe un script para esto? → USAR ESE SCRIPT
□ 2. ¿El script maneja dependencias (NLP antes de Matching)? → CONFIAR EN ÉL
□ 3. ¿Necesito verificar algo? → EL SCRIPT YA LO HACE
□ 4. ¿Quiero hacer una query manual? → NO, USAR EL SCRIPT
```

### Flujo ÚNICO para Optimización (NO hay alternativa)

```bash
# PASO 1: Procesar ofertas (NLP + Matching + Validación automática)
python scripts/run_validated_pipeline.py --limit 10

# PASO 2: Ver errores detectados
python scripts/review_offer_chain.py --errores --limit 5

# PASO 3: Si hay errores, crear regla en config/*.json correspondiente

# PASO 4: Reprocesar SOLO los IDs con error
python scripts/run_validated_pipeline.py --ids X,Y,Z

# PASO 5: Comparar
python scripts/compare_runs.py --latest

# PASO 6: Cuando converge, exportar Excel
python scripts/exports/export_validation_excel.py --etapa completo --ids X,Y,Z
```

### ❌ PROHIBIDO (durante ejecución del pipeline)

| Acción | Por qué está mal |
|--------|------------------|
| Queries manuales a BD para verificar estado | El script ya verifica |
| Ejecutar matching sin verificar NLP | `run_validated_pipeline` ya lo maneja |
| Crear scripts "demo" o "test" ad-hoc | Ya existen los scripts |
| Inventar pasos no documentados | Todo está en CLAUDE.md |
| Hacer verificaciones "por las dudas" | Confiá en el pipeline |

### ✅ PERMITIDO SIEMPRE

| Acción | Cuándo |
|--------|--------|
| Editar `config/*.json` | Para crear reglas nuevas |
| Leer archivos para entender código | Antes de modificar |
| Ejecutar scripts documentados | Siempre |

### ✅ PERMITIDO INTERVENIR MANUALMENTE cuando:

| Situación | Qué hacer |
|-----------|-----------|
| **Usuario pide ver datos específicos** | Queries a BD para mostrar lo que pide |
| **Diagnosticar error que el script no resuelve** | Investigar cadena completa (NLP → Skills → Matching) |
| **Crear regla nueva** | Consultar BD para ver ejemplos similares, entender el patrón |
| **Entender por qué falló algo** | Leer logs, ver datos de la oferta específica |
| **Usuario pregunta "¿qué pasó con X?"** | Investigar libremente |
| **Depurar un bug en el pipeline** | Queries diagnósticas, leer código |
| **Explorar para planificar** | Antes de ejecutar, entender el estado actual |

### 🔑 REGLA CLAVE

```
EJECUCIÓN DE PIPELINE → Usar scripts, no improvisar
DIAGNÓSTICO/INVESTIGACIÓN → Intervenir manualmente está OK
CREAR REGLAS → Necesito ver datos para entender el patrón
```

**Preguntarse:** ¿Estoy EJECUTANDO el pipeline o estoy INVESTIGANDO/DIAGNOSTICANDO?
- Ejecutando → Scripts únicamente
- Investigando → Queries manuales OK

---

## REGLAS CRÍTICAS - LEER PRIMERO

**ANTES de escribir código o crear archivos:**

1. **LEER este CLAUDE.md COMPLETO** - todo está documentado aquí
2. **NUNCA crear scripts nuevos** - buscar si ya existe uno para la tarea
3. **Cambios van en `config/*.json`** - no en código Python
4. **Si hay que modificar un `.py`**:
   - Versionar (ej: `v3.py` → `v4.py`)
   - Archivar versión anterior en `archive_old_versions/`
   - Actualizar CLAUDE.md

### Entry Points del Sistema (NO crear alternativas)

**⚠️ WSL + Ollama:** Si Ollama corre en Windows y el proyecto en WSL, usar:
```bash
OLLAMA_HOST=172.17.0.1 python scripts/run_validated_pipeline.py --limit 500
```

**⚠️ Lotes grandes (>100):** Correr en BACKGROUND, no esperar:
```bash
OLLAMA_HOST=172.17.0.1 python scripts/run_validated_pipeline.py --limit 500 > /tmp/pipeline.log 2>&1 &
# Verificar progreso:
tail -f /tmp/pipeline.log
```

| Tarea | Comando | NO hacer |
|-------|---------|----------|
| **⭐ Pipeline Completo** | `python scripts/run_validated_pipeline.py --limit 500` | Scripts separados |
| **NLP lote background** | `python scripts/launch_nlp_batch.py` (skip-matching, log a archivo) | Crear script nuevo |
| **NLP IDs específicos** | `python database/process_nlp_from_db_v11.py --ids X` | Crear script nuevo |
| **Scraping (local)** | `python run_scheduler.py --test` | Llamar scrapers directo |
| **Scraping ComputRabajo** | `python scripts/scraping/run_computrabajo_vps.py` | Usar scraper directo |
| **Scraping CABA** | `python scripts/scraping/run_caba_vps.py` | Usar scraper directo |
| **Scraping Portal Empleo** | `python scripts/scraping/run_portalempleo_vps.py` | Usar scraper directo |
| **Scraping Indeed** | `python scripts/scraping/run_indeed_vps.py` | Usar scraper directo |
| **Sync desde VPS** | `python scripts/sync_from_vps.py` (incremental) | Queries manuales al VPS |
| **Sync Full desde VPS** | `python scripts/sync_from_vps.py --full` | - |
| **Comparar runs** | `python scripts/compare_runs.py --latest` | Crear comparador custom |
| **Validar ofertas** | `python scripts/validar_ofertas.py --ids X --estado validado` | UPDATE manual en BD |
| **Export Excel** | `python scripts/exports/export_validation_excel.py --etapa completo --ids X` | - |
| **Sync Supabase** | `python scripts/exports/sync_to_supabase.py` (incremental auto v2.2) | Queries directas a Supabase |
| **Sync Full** | `python scripts/exports/sync_to_supabase.py --full` | - |
| **Backfill columnas** | `python scripts/exports/backfill_validation_columns.py` | Solo si se agregan columnas nuevas |
| **Backfill skills** | `python scripts/exports/backfill_skills.py` | Solo si ofertas_skills está incompleta |
| **Reapply Rules** | `python scripts/reapply_rules_to_validated.py` | Reprocesar validadas manualmente |
| **Generar Architecture JSON** | `python scripts/generate_architecture_json.py` | Editar dashboard_architecture.json a mano |

### Scraping — Estado y Dependencias

**Estado actual (2026-03-10):** Scraping corre en **VPS** (Hostinger KVM 2, IP 187.124.150.28).
Cron ejecuta Lun/Jue 08:00 Argentina via `/opt/mol/scripts/scraping/run_scraping_vps.sh`.

**Portales:**
| Portal | Estado | Ofertas/corrida | Método |
|--------|--------|----------------|--------|
| **Bumeran** | Activo en VPS | ~5,000 | API searchV2 + keywords (paginación funciona) |
| **ZonaJobs** | Activo en VPS | ~5,000 | API searchV2 + keywords (paginación rota, 20/keyword) |
| **ComputRabajo** | Activo en VPS | ~1,000+ | HTML scraping + keywords (~3-4h con descripción) |
| **CABA** | Activo en VPS | ~10-50 | HTML scraping listado+detalle (~30s total) |
| **Portal Empleo** | Activo en VPS | ~400-500 | HTML scraping listado+detalle (~13 min) |
| **Indeed** | Activo en VPS | ~2,000-3,000 | curl_cffi + keywords (~2.5h con detalles) |
| LinkedIn | Scraper legacy (JobSpy) | - | No integrado (0% descripciones) |

**ZonaJobs - Limitación de API:**
La API de ZonaJobs (mismo platform Navent que Bumeran) tiene paginación rota:
el parámetro `page` es ignorado y siempre devuelve las mismas 20 ofertas.
Workaround: usar `query` keyword para obtener 20 ofertas distintas por keyword.
Con 1,072 keywords de `config/scraping/master_keywords.json` se obtienen ~5,000 únicas.

**Archivos del scraper ZonaJobs:**
| Archivo | Ubicación | Función |
|---------|-----------|---------|
| Scraper v2 | `01_sources/zonajobs/scrapers/zonajobs_scraper_v2.py` | Scraper API + keywords |
| Runner VPS | `scripts/scraping/run_zonajobs_vps.py` | Ejecuta scraping + inserta en BD |
| Script cron | `scripts/scraping/run_scraping_vps.sh` | Bumeran + ZonaJobs + ComputRabajo + CABA + Portal Empleo + Indeed + export |

**ComputRabajo - Detalles:**
- HTML scraping con `requests` + `BeautifulSoup` (NO requiere JavaScript)
- Usa keywords de `config/scraping/master_keywords.json` (mismas que ZonaJobs)
- `fetch_description=True` por default: 1 request extra por oferta para descripción completa
- IDs: `data-id` del HTML → convertido a integer con prefijo `5_000_000_000` (evita colisiones)
- Campos mapeados: título, empresa, ubicación, modalidad, fecha, descripción, URL
- Campos NULL (no disponibles): id_empresa, id_area, id_subarea, cantidad_vacantes, tipo_trabajo
- Requiere `beautifulsoup4` instalado en VPS: `pip3 install beautifulsoup4`

**Archivos del scraper ComputRabajo:**
| Archivo | Ubicación | Función |
|---------|-----------|---------|
| Scraper core | `01_sources/computrabajo/scrapers/computrabajo_scraper.py` | HTML parsing + BS4 |
| Multi-keyword | `01_sources/computrabajo/scrapers/scrapear_con_diccionario.py` | Wrapper legacy multi-keyword |
| Runner VPS | `scripts/scraping/run_computrabajo_vps.py` | Ejecuta scraping + mapea + inserta en BD |

**CABA Portal de Trabajo - Detalles:**
- HTML scraping con `requests` + `BeautifulSoup` (NO requiere JavaScript)
- NO usa keywords: pagina el listado completo con `offset` (8 por página)
- Detalle: `/anuncios/{id}` con campos MUY ricos (industria, sector, vacantes, educación, IT, idiomas)
- IDs: `6_000_000_000 + id_anuncio` (evita colisiones)
- Campos extra (industria, sector, estudios, IT) se embeben en descripción como metadata
- Portal chico (~10-50 ofertas) pero datos muy estructurados
- Puede correr local (rápido, ~30s) o en VPS

**Archivos del scraper CABA:**
| Archivo | Ubicación | Función |
|---------|-----------|---------|
| Scraper core | `01_sources/caba/scrapers/caba_scraper.py` | HTML parsing listado + detalle |
| Runner | `scripts/scraping/run_caba_vps.py` | Ejecuta scraping + mapea + inserta en BD |

**Portal Empleo Nacional - Detalles:**
- HTML scraping con `requests` + `BeautifulSoup` (NO requiere JavaScript)
- NO usa keywords: pagina el listado completo con `page-number=N` (10 por página)
- Detalle: `/OfertasLaborales/Details/{uuid}` con campos ricos (vacantes, salario, tareas, beneficios, ubicación completa, horario, experiencia, estudios)
- IDs: UUIDs convertidos a integer con CRC32 + prefijo `7_000_000_000` (evita colisiones)
- Campos estructurados (salario, estudios, experiencia, horario) se embeben en descripción como metadata
- Portal nacional (~400-500 ofertas) con cobertura de todas las provincias
- Scrape completo: ~13 min (1.5s delay entre requests)

**Archivos del scraper Portal Empleo:**
| Archivo | Ubicación | Función |
|---------|-----------|---------|
| Scraper core | `01_sources/portalempleo/scrapers/portalempleo_scraper.py` | HTML parsing listado + detalle |
| Runner VPS | `scripts/scraping/run_portalempleo_vps.py` | Ejecuta scraping + mapea + inserta en BD |

**Indeed - Detalles:**
- Scraper propio con `curl_cffi` + `BeautifulSoup` (SIN JobSpy)
- `curl_cffi` bypasea Cloudflare via impersonacion TLS Chrome
- Usa keywords de `config/scraping/master_keywords.json` (mismas que ZonaJobs/CT)
- Sin paginacion (Indeed redirige a login en pagina 2): ~15 ofertas/keyword
- Detalle: `/viewjob?jk={job_key}` con JSON-LD estructurado (datePosted, employmentType, baseSalary)
- IDs: `8_000_000_000 + int(job_key_hex, 16) % 1_000_000_000` (evita colisiones)
- Descripcion completa del detalle: ~94% success rate
- Scrape completo: ~2.5h (600 keywords * 2.5s + ~3000 detalles * 2.5s)
- Requiere `curl_cffi` instalado en VPS: `pip3 install --break-system-packages curl_cffi`

**Archivos del scraper Indeed:**
| Archivo | Ubicación | Función |
|---------|-----------|---------|
| Scraper core | `01_sources/indeed/scrapers/indeed_scraper.py` | curl_cffi + BS4, multi-keyword |
| Runner VPS | `scripts/scraping/run_indeed_vps.py` | Ejecuta scraping + mapea + inserta en BD |
| Scraper legacy | `01_sources/indeed/scrapers/archive/indeed_scraper_jobspy.py` | Versión anterior con JobSpy (archivado) |

**VPS - Infraestructura:**
```
VPS (187.124.150.28)                      Local (esta máquina)
/opt/mol/                                  D:\OEDE\Webscrapping\
  ├── run_scheduler.py (Bumeran)           ├── scripts/sync_from_vps.py
  ├── scripts/scraping/                    │   (export → SCP → import → bajas)
  │   ├── run_scraping_vps.sh (cron)       ├── database/bumeran_scraping.db
  │   ├── run_zonajobs_vps.py              │   (35K+ ofertas)
  │   ├── run_computrabajo_vps.py          └── NLP + Matching + Validación (local)
  │   ├── run_caba_vps.py
  │   ├── run_portalempleo_vps.py
  │   └── run_indeed_vps.py
  ├── scripts/export_nuevas.py
  └── database/bumeran_scraping.db
```

**Sync VPS → Local:**
```bash
python scripts/sync_from_vps.py          # Incremental (solo nuevas)
python scripts/sync_from_vps.py --full   # Full (todas)
```

**Dependencias críticas del scraper (restauradas 2026-03-06):**

Los scrapers importan módulos desde `02_consolidation/scripts/`:
- `incremental_tracker.py` — Tracking de IDs ya scrapeados (modo incremental)
- `date_filter.py` — Filtrado por ventana temporal

Estos archivos fueron movidos por error a `archive/legacy_numbered_folders/` y restaurados.
Si faltan, el scraper sigue funcionando pero **sin modo incremental ni filtrado por fecha**
(los imports tienen try/except con fallback graceful).

### Sync a Supabase — Cómo funciona internamente

El sync tiene **dos partes independientes** que suben datos a tablas distintas:

```
SQLite (local)                              Supabase (cloud)
┌──────────────────┐                        ┌──────────────────┐
│ ofertas          │                        │ ofertas_dashboard │
│ ofertas_nlp      │──JOIN + transform──►   │ (16K+ rows)      │
│ ofertas_esco_    │   upsert x100          │ ~40 columnas      │
│   matching       │                        └──────────────────┘
└──────────────────┘
┌──────────────────┐                        ┌──────────────────┐
│ ofertas_esco_    │                        │ ofertas_skills    │
│   skills_detalle │──delete+insert──►      │ (300K+ rows)     │
│                  │   por oferta            │ ~12 cols          │
└──────────────────┘                        └──────────────────┘
```

**Parte 1 — Ofertas (`ofertas_dashboard`):**
- Query con 3 JOINs: `ofertas` + `ofertas_nlp` + `ofertas_esco_matching`
- Solo `estado_validacion IN ('validado', 'validado_claude', 'validado_humano')`
- `transform_oferta_for_supabase()` normaliza: ubicación, CLAE, skills como arrays
- Upsert por `id_oferta` en batches de 100
- Rápido: ~2 min para 16K ofertas

**Parte 2 — Skills (`ofertas_skills`):**
- Por cada oferta: DELETE todas sus skills + INSERT nuevas
- Parsea `source_classification` JSON → `l1`, `l2`, `es_digital`
- LENTO: ~300K HTTP requests individuales (~60-90 min)
- Free tier de Supabase limita a ~15 req/s

**Modos de ejecución:**

| Modo | Comando | Qué hace | Duración |
|------|---------|----------|----------|
| Incremental | `sync_to_supabase.py` | Solo ofertas con timestamp > último sync | ~1-5 min |
| Full | `sync_to_supabase.py --full` | Re-sube TODO (ofertas + skills + indicadores) | ~2.5h |

El timestamp incremental se lee/guarda en `config/supabase_sync_log.json`.

**Backfills (one-time, para cuando se agregan columnas nuevas):**

Si agregás columnas a `ofertas_dashboard` en Supabase:
1. Crear migration SQL en `fase3_dashboard/sql/`
2. Ejecutar el ALTER TABLE en Supabase SQL Editor
3. Agregar campo a `transform_oferta_for_supabase()` en `sync_to_supabase.py`
4. Backfill con RPC temporal (NO usar `--full`):
   - Crear función SQL temporal que reciba JSONB y haga UPDATE
   - Llamar desde Python con batches de 500 vía `client.rpc()`
   - Ejemplo: `backfill_validation_columns.py` pobló 16K rows en 38 seg

**NUNCA usar `--full` solo para backfill de columnas** — tarda 2.5h innecesariamente.

**Rate limiting (free tier Supabase):**
- Máximo ~15 req/s sostenido
- Si se excede: Cloudflare Error 1018 (host not found) = proyecto pausado
- Solución: agregar `time.sleep(1)` entre chunks en scripts de backfill
- Si se pausa: ir a Supabase Dashboard → reactivar proyecto

**⭐ REGLA CRÍTICA - Pipeline Integrado:**
- **SIEMPRE** usar `run_validated_pipeline.py` para procesar ofertas
- Ejecuta TODO automáticamente: Matching → Validación → Corrección → Reporte
- Errores se persisten en tabla `validation_errors` (no se pierden)
- Si hay errores que requieren reglas nuevas → genera `metrics/cola_claude_*.json`

### Flujo Completo de Trabajo (v3.4.3)

```
PASO 1: PROCESAR OFERTAS NUEVAS
───────────────────────────────
python scripts/run_validated_pipeline.py --limit 100
  → NLP → Matching → Validación Automática → Reporte Errores

PASO 2: RESOLVER ERRORES (si hay)
─────────────────────────────────
# Ver errores
python scripts/review_offer_chain.py --errores --limit 5

# Crear reglas según tipo de error:
│ Error NLP        │ → config/nlp_inference_rules.json
│ Error Matching   │ → config/matching_rules_business.json
│ Error Limpieza   │ → config/nlp_titulo_limpieza.json

# Reprocesar IDs con error
python scripts/run_validated_pipeline.py --ids X,Y,Z

PASO 3: APLICAR REGLAS A VALIDADAS (si se crearon reglas nuevas)
────────────────────────────────────────────────────────────────
# Las reglas nuevas pueden afectar ofertas YA validadas
# Este script las reprocesa SIN perder el estado de validación
python scripts/reapply_rules_to_validated.py

PASO 4: SINCRONIZAR A SUPABASE
──────────────────────────────
python scripts/exports/sync_to_supabase.py
```

**¿Cuándo usar `reapply_rules_to_validated.py`?**
- Después de crear reglas nuevas en `matching_rules_business.json`
- Cuando hay ofertas validadas que podrían beneficiarse de las nuevas reglas
- El script detecta automáticamente qué ofertas reprocesar (las que tuvieron errores resueltos)

---

## Pipeline de Validación con Aprendizaje (v2.0)

**Principio:** Claude REVISA casos individuales para APRENDER y crear REGLAS en JSONs.
El sistema luego aplica las reglas automáticamente. Claude NO reemplaza al LLM.

### Cadena de Dependencias

```
SCRAPING → NLP → NLP GATE → MULTI-POS → SKILLS → MATCHING
  ↓          ↓       ↓           ↓          ↓         ↓
portal    tareas  bloquea     split      extraídas  ESCO code
metadata  ubicac  critico/    sub-       de tareas
embebida  senior  alto        ofertas    +título
          área

Source-aware: CABA/Portal Empleo/Indeed embeben metadata → pre-fill NLP
Si NLP extrae mal → Gate bloquea → Auto-corrige → Escala a Claude
Si NLP OK → Multi-position split → Skills → Matching
```

### Flujo de Trabajo (UN COMANDO)

```
COMANDO ÚNICO (hace TODO automáticamente):
──────────────────────────────────────────
python scripts/run_validated_pipeline.py --limit 100

EJECUTA AUTOMÁTICAMENTE (v3.3 — 8 pasos):
  1.   NLP             → process_nlp_from_db_v11.py v11.4 (source-aware)
  1.5  NLP GATE        → nlp_validator.py v1.1 (35+ reglas, bloquea critico/alto)
  1.5b NLP AUTO-CORR   → auto_corrector.py (corrige NLP, re-valida, escala a Claude)
  1.6  MULTI-POSITION  → limpiar_titulos.py (detecta "Vendedor / Cajero", crea sub-ofertas)
  2.   MATCHING        → match_ofertas_v3.py v3.5.4 (ESCO-First)
  3.   VALIDACIÓN      → auto_validator.py (detecta errores → BD)
  4.   AUTO-CORRECCIÓN → auto_corrector.py (arregla lo que puede → BD)
  5.   NLP RE-PROCESS  → si hay errores NLP → reprocesa → vuelve a paso 1.5 (max 2 iter)
  6.   REPORTE         → genera cola_claude.json si hay errores escalados
  7.   EXPORT EXCEL    → Excel para validación humana
  8.   SYNC LEARNINGS  → actualiza .ai/learnings.yaml

OPCIONES:
  --limit N          Procesar N ofertas (por prioridad)
  --ids X,Y,Z        Procesar IDs específicos
  --skip-nlp         Saltar NLP (solo matching+validación)
  --skip-matching    Saltar matching (solo validar)
  --only-pending     Solo ofertas pendientes de matching
  --no-priority      Sin sistema de prioridad
  --export-markdown  Generar validation/feedback_*.md para GitHub

RESULTADO:
  - Errores detectados → tabla validation_errors (persistidos)
  - Errores corregidos → marcados corregido=1 en BD (con valor_corregido)
  - Errores escalados → metrics/cola_claude_*.json + escalado_claude=1 en BD
  - Fine-tuning data → llm_raw_json + postprocessor_diff_json + valor_actual en BD
```

### Si Hay Errores Escalados

```
CLAUDE REVISA cola_claude_*.json:
─────────────────────────────────
python scripts/review_offer_chain.py --errores --limit 5

Claude ve TODA la cadena:
1. SCRAPING: ¿Datos completos?
2. NLP: ¿Tareas, ubicación, seniority, área correctos?
3. SKILLS: ¿Coherentes con título y tareas?
4. MATCHING: ¿ISCO correcto?

CREAR REGLA según dónde falló:
| Falla en | Config a modificar |
|----------|-------------------|
| NLP - tareas | prompt o nlp_extraction_patterns.json |
| NLP - ubicación | config/nlp_preprocessing.json |
| NLP - seniority | config/nlp_inference_rules.json |
| NLP - área | config/nlp_inference_rules.json |
| Skills - faltan | config/skills_database.json |
| Matching | config/matching_rules_business.json |

REPROCESAR IDs afectados:
python scripts/run_validated_pipeline.py --ids X,Y,Z
```

### Archivos del Sistema de Validación

| Archivo | Función |
|---------|---------|
| `scripts/run_validated_pipeline.py` | **⭐ ENTRY POINT PRINCIPAL** v3.3 - orquesta 8 pasos |
| `database/nlp_validator.py` | **NLP Gate** v1.1 - 35+ reglas pre-matching (bloquea critico/alto) |
| `config/nlp_validation_rules.json` | Reglas NLP: V01-V26, NV02-NV11, NQ01-NQ05 (cross-field) |
| `config/validation_rules.json` | Reglas matching: auto-detección (ver conteos en learnings.yaml) |
| `config/diagnostic_patterns.json` | Patrones para identificar punto de falla |
| `config/auto_correction_map.json` | Mapeo diagnóstico → config a modificar |
| `database/auto_validator.py` | Validador matching (persiste en BD) |
| `database/auto_corrector.py` | Corrector automático (con valor_corregido tracking) |
| `database/limpiar_titulos.py` | v2.8.1 - limpieza títulos + multi-position detection |
| `scripts/review_offer_chain.py` | **Revisión UNO POR UNO** (cadena completa) |
| `scripts/launch_nlp_batch.py` | NLP batch en background con logging a archivo |

### Tablas de Validación en BD

| Tabla | Función |
|-------|---------|
| `validation_errors` | Errores detectados (con `valor_actual` y `valor_corregido`) |
| `ofertas_esco_matching` | Estado de matching y validación |
| `ofertas_nlp` | Campos NLP + `nlp_gate_status` + `llm_raw_json` + `postprocessor_diff_json` |
| `pipeline_runs` | Historial de corridas |

**Consultas útiles:**
```sql
-- Errores pendientes (no resueltos)
SELECT * FROM v_errores_pendientes;

-- Resumen por tipo de error
SELECT * FROM v_errores_por_tipo;

-- Errores escalados a Claude
SELECT * FROM validation_errors WHERE escalado_claude = 1 AND resuelto = 0;
```

### Ejemplo de Revisión Claude

```
Caso: "Gerente de Ventas" → ISCO 2433 (incorrecto, debería ser 1221)

Claude revisa cadena:
1. SCRAPING: OK
2. NLP: nivel_seniority = NULL ❌ (debería ser "manager")
3. SKILLS: OK
4. MATCHING: Sin seniority, no priorizó nivel directivo

Diagnóstico: Falla RAÍZ en NLP (seniority no inferido de "Gerente")

Claude crea reglas:
1. nlp_inference_rules.json: {"keyword": "gerente", "nivel_seniority": "manager"}
2. matching_rules_business.json: R_GERENTE_VENTAS → forzar_isco 1221

Reprocesar → ISCO correcto
Próxima vez: Sistema aplica regla automáticamente
```

→ **Plan completo:** `/home/gerardo/.claude/plans/elegant-crunching-hippo.md`
→ **Guía optimización:** `docs/guides/OPTIMIZACION.md`

---

### Flujo de Optimización LEGACY (sin revisión Claude)

```
1. PROCESAR    → run_matching_pipeline(ids, source="gold_set")
2. EXPORTAR   → export_validation_excel.py --ids X
3. CORREGIR   → Modificar config/*.json (NO código Python)
4. COMPARAR   → compare_runs.py --latest
5. REPETIR    → Pasos 2-4 hasta que esté OK
6. VALIDAR    → validar_ofertas.py --ids X --estado validado
```

→ **Detalles:** `docs/guides/VALIDACION.md`

### Feedback Loop via Google Sheets

```
FLUJO:
1. Exportar   → python scripts/exports/export_validation_excel.py --etapa completo --ids X
2. Subir      → Excel a Google Sheets (manual)
3. Humano     → Edita en Google Sheets (columnas resultado, isco_correcto, comentario)
4. Claude     → Usuario comparte link/CSV, Claude lee y crea reglas
```

**Columnas editables por humano:**
- `resultado`: `OK` | `ERROR` | `REVISAR`
- `isco_correcto`: ISCO esperado (si es ERROR)
- `comentario`: Descripción del problema

### Protección de Datos Validados (v2.0 - 2026-01-20)

**REGLA ABSOLUTA:** Una oferta con `estado_validacion = 'validado'` NUNCA debe:
- Ser reprocesada por NLP
- Ser reprocesada por Matching
- Aparecer en Excel de validación nuevo

**Capas de protección:**

| Capa | Ubicación | Qué hace |
|------|-----------|----------|
| Query filtrada | `export_validation_excel.py:476` | Excluye validadas de selección |
| Query filtrada | `auto_validator.py:590` | Excluye validadas de validación |
| Error explícito | `match_ofertas_v3.py:1368` | Lanza ValueError si hay validadas |
| Trigger BD | `migrations/016_*.sql` | Bloquea UPDATE en ofertas validadas |

**Verificar protección:**
```bash
python scripts/check_validated_protection.py
```

**Si NECESITO reprocesar una oferta validada (caso excepcional):**
```bash
# 1. Desbloquear con justificación obligatoria
python scripts/admin_unlock_validated.py --ids 123,456 --motivo "Razón del desbloqueo"

# 2. Reprocesar normalmente
python scripts/run_validated_pipeline.py --ids 123,456
```

**NO hacer:**
- Usar `force=True` en el pipeline (bypasea controles)
- UPDATE directo a BD sin usar script admin
- Cambiar estado manualmente a 'pendiente'

---

## VERSIONES ACTUALES - USAR SIEMPRE ESTAS

### NLP Pipeline

| Componente | Archivo ACTUAL | NO USAR |
|------------|----------------|---------|
| Pipeline NLP | `database/process_nlp_from_db_v11.py` v11.4 | v7, v8, v9, v10 |
| Prompt | `database/prompts/extraction_prompt_lite_v1.py` | v8, v9, v10 |
| Regex Patterns | `database/patterns/regex_patterns_v4.py` | v1, v2, v3 |
| Normalizador | `database/normalize_nlp_values.py` | - |
| Postprocessor | `database/nlp_postprocessor.py` v1.3 | - |
| NLP Validator (Gate) | `database/nlp_validator.py` v1.1 | - |
| Limpiador títulos | `database/limpiar_titulos.py` v2.8.1 | - |
| Batch background | `scripts/launch_nlp_batch.py` | - |

**Arquitectura v11.4 (source-aware):**
```
CAPA 0: Regex (salarios, jornada) + Scraping directo (modalidad, portal)
CAPA 1: LLM Qwen2.5:7b (20 campos)
CAPA 1b: Source-aware pre-fill (CABA/Portal Empleo/Indeed metadata embebida)
CAPA 2: Postprocessor (config/nlp_*.json) + LLM raw snapshot + diff tracking
CAPA 3: Skills implícitas (LoRA fine-tuned BGE-M3 + ESCO embeddings)

NLP GATE (pre-matching):
  nlp_validator.py (35+ reglas) → aprobado/bloqueado
  Bloqueados → auto-corrección → re-validación → escalar a Claude
```

**Campos fine-tuning (v11.4):**
- `llm_raw_json`: Output original del LLM (antes del postprocessor)
- `postprocessor_diff_json`: Qué campos cambió el postprocessor y por qué
- `valor_actual` / `valor_corregido`: En validation_errors, para auditoría

### Matching Pipeline v3.5.4 ESCO-First

| Componente | Archivo ACTUAL | NO USAR |
|------------|----------------|---------|
| Pipeline Matching | `database/match_ofertas_v3.py` v3.5.4 | v2.py, v8.x |
| Matcher por Skills | `database/match_by_skills.py` v1.2.0 | - |
| Skills Extractor | `database/skills_implicit_extractor.py` v2.4 | - |
| Skills Rules Config | `config/skills_rules.json` (25 reglas) | - |
| Skills Rules Matcher | `database/skills_rules_matcher.py` | - |
| Diccionario Argentino | `config/sinonimos_argentinos_esco.json` (17 ocup) | - |
| Config reglas negocio | `config/matching_rules_business.json` (297 reglas) | hardcodeados |
| Config principal | `config/matching_config.json` | - |

**Arquitectura v3.5.4 (orden de prioridad):**
```
PRINCIPIO: ESCO es TARGET, ISCO es CONSECUENCIA

1. REGLAS DE NEGOCIO (GANAN SIEMPRE si aplican)
   └── Buscan ESCO label exacto → derivan ISCO
   └── titulo_original_contiene_alguno (v3.5.4: busca en título SIN limpiar)
        ↓ (si no hay regla)
2. DICCIONARIO ARGENTINO ← Vocabulario local → ISCO
        ↓ (si no matchea)
3. SEMÁNTICO (LoRA fine-tuned) ← Skills 60% + Titulo 40%
        ↓
4. PENALIZACIONES (sector, seniority)
        ↓
5. PERSISTIR EN BD (con metadata dual: isco_semantico, isco_regla)
```

→ **Detalles:** `docs/reference/PIPELINE.md`

### Skills Dual System v2.4 (2026-03-15)

Sistema DUAL para extracción de skills (mismo patrón que ISCO matching):
- **Reglas de skills** (prioridad) + **LoRA fine-tuned model** (fallback)
- Guarda AMBOS resultados para comparación y métricas

| Componente | Archivo | Propósito |
|------------|---------|-----------|
| Skills Rules Config | `config/skills_rules.json` | 25 reglas que fuerzan skills específicas |
| Skills Rules Matcher | `database/skills_rules_matcher.py` | Evaluador de reglas |
| Skills Extractor | `database/skills_implicit_extractor.py` v2.4 | Método `extract_skills_dual()` |
| Modelo | `data/finetuning/matching/model_lora` | LoRA fine-tuned (reemplaza BGE-M3 default) |

**Arquitectura Dual:**
```
1. Evaluar REGLAS DE SKILLS (skills_rules.json)
   └── Si matchea → skills_regla (prioridad)
        ↓
2. Extraer SEMÁNTICO (LoRA fine-tuned siempre)
   └── skills_semantico
        ↓
3. Comparar ambos
   └── dual_coinciden_skills: 1=igual, 0=difieren, NULL=solo semántico
        ↓
4. Merge final
   └── skills_final = skills_regla + skills_semantico únicos
```

**Columnas en BD (`ofertas_esco_matching`):**
- `skills_regla_json`: Skills forzadas por regla (JSON array)
- `skills_semantico_json`: Skills de BGE-M3 (JSON array)
- `skills_regla_aplicada`: ID de regla aplicada (ej: "RS02_contador")
- `dual_coinciden_skills`: 1=coinciden, 0=difieren, NULL=sin regla

**Reglas de Validación (V24-V30):**
| Regla | Detecta | Severidad |
|-------|---------|-----------|
| V24 | Skills no coherentes con ISCO (< 30%) | alto |
| V25 | Tareas vacías pero skills presentes | medio |
| V26 | Formato tareas incorrecto (`,` vs `;`) | medio |
| V27 | Divergencia regla vs semántico | warning |
| V28 | Sin skills esenciales del ISCO | alto |
| V29 | Tareas muy cortas (< 50 chars) | bajo |
| V30 | Puesto IT sin skills técnicas | alto |

**Métricas en sync_learnings.py:**
```
SKILLS DUAL (v2.3):
  Por regla: 45% | Por semantico: 55%
  Dual coinciden: 78%
  Skills promedio: 6.2/oferta
```

### Tests y Gold Sets

| Conjunto | Ubicación | Casos | Uso |
|----------|-----------|-------|-----|
| **Ofertas en validación** | BD `ofertas_esco_matching` | **100** | Validar para dashboard |
| Gold Set referencia | `database/gold_set_manual_v2.json` | 49 | Test de regresión |
| NLP Extraction | `tests/nlp/gold_set.json` | 20+ | Test NLP |

**IMPORTANTE:** El trabajo actual es sobre las **100 ofertas en validación**, no el Gold Set de 49.

```bash
# Ejecutar tests Python (pipeline)
python -m pytest tests/ -v

# Test Gold Set Matching (referencia)
pytest tests/matching/test_gold_set_manual.py -v

# Ver estado de las 100 ofertas en validación
python scripts/validar_ofertas.py --status
```

### Tests Frontend (Dashboard)

**Framework:** Vitest v4 + Testing Library (React) + MSW (Mock Service Worker) + happy-dom

**Ubicación:** `fase3_dashboard/mol-dashboard/__tests__/`

```
__tests__/
  component/   ← tests de componentes React (renderizado, interacción)
  unit/        ← tests unitarios (auth-utils, build-rpc-filters)
  integration/ ← tests de integración
  security/    ← tests de seguridad (RLS, auth)
  mocks/
    handlers.ts   ← MSW handlers que interceptan Supabase RPCs
    server.ts     ← MSW server (arranca en vitest.setup.ts)
    fixtures/     ← datos mock (ofertas, pipeline-status, skills, etc.)
```

**Mocks de Supabase:** MSW intercepta llamadas HTTP a `https://test.supabase.co` y responde con fixtures. Los handlers mockean RPCs (`get_pipeline_status`, `reconciliar_sistemas`, etc.) y tablas PostgREST (`ofertas_dashboard`, `ofertas_skills`, etc.).

**Config:** `vitest.config.ts` — environment happy-dom, setup con MSW, include `__tests__/**/*.test.{ts,tsx}`, alias `@/` → raíz del proyecto.

```bash
# Ejecutar tests frontend
cd fase3_dashboard/mol-dashboard
npx vitest run                    # todos (933+ tests)
npx vitest run --reporter=verbose # con detalle
npx vitest --ui                   # UI interactiva
npx vitest run __tests__/component/centro-control.test.tsx  # uno solo
```

**IMPORTANTE:** Al agregar componentes nuevos al dashboard, agregar tests en `__tests__/component/`. Seguir el patrón existente: mockear datos en `mocks/fixtures/`, agregar handlers en `mocks/handlers.ts` si hace falta, renderizar con `render()` de testing-library, verificar con `screen.getByText()` / `screen.queryByText()`.

---

## Configuración

### Configs NLP (Postprocessor)

| Archivo | Propósito |
|---------|-----------|
| `config/nlp_preprocessing.json` | Parsing ubicación |
| `config/nlp_inference_rules.json` | Inferencia área/seniority/modalidad |
| `config/nlp_validation.json` | Validación tipos |
| `config/nlp_extraction_patterns.json` | Regex experiencia |
| `config/nlp_normalization.json` | CABA → Capital Federal |

### Configs Matching

| Archivo | Propósito |
|---------|-----------|
| `config/matching_config.json` | Pesos, umbrales, penalizaciones |
| `config/matching_rules_business.json` | Reglas de negocio (ver conteos en learnings.yaml) |
| `config/area_funcional_esco_map.json` | Mapeo área → ISCO |
| `config/sector_isco_compatibilidad.json` | Compatibilidad sector-ISCO |

### Diccionarios

| Archivo | Propósito |
|---------|-----------|
| `config/skills_database.json` | ~320 skills técnicas |
| `config/oficios_arg.json` | ~170 oficios argentinos |

---

## Comandos Clave

```bash
# === SCRAPING ===
python run_scheduler.py --test

# === NLP ===
python database/process_nlp_from_db_v11.py --ids 123,456

# === MATCHING ===
pytest tests/matching/test_gold_set_manual.py -v

# === VALIDACIÓN ===
python scripts/validar_ofertas.py --status
python scripts/compare_runs.py --list
python scripts/compare_runs.py --latest
python scripts/validar_ofertas.py --ids 123,456 --estado validado

# === EXPORT ===
python scripts/exports/export_validation_excel.py --etapa completo --ids X
```

---

## Guía Rápida: Mapeo Error → Config

| Tipo de Error | Config a Editar |
|---------------|-----------------|
| Provincia/Localidad mal | `config/nlp_preprocessing.json` |
| Seniority incorrecto | `config/nlp_inference_rules.json` |
| Modalidad incorrecta | `config/nlp_inference_rules.json` |
| Área funcional incorrecta | `config/nlp_inference_rules.json` |
| ISCO incorrecto para título X | `config/matching_rules_business.json` |

→ **Tabla completa:** `docs/guides/OPTIMIZACION.md`

---

## Regla de Versionado

**OBLIGATORIO:** Cuando se crea una nueva versión:

1. Crear nueva versión (ej: `v11.py`)
2. Archivar anterior en `database/archive_old_versions/`
3. Verificar que nada importe el archivo archivado
4. Actualizar CLAUDE.md

**NUNCA** dejar dos versiones activas en el mismo directorio.

---

## Ubicación de Scripts

| Si el script es para... | Va en... |
|-------------------------|----------|
| Gold Set NLP | `scripts/nlp/gold_set/` |
| Gold Set Matching | `scripts/matching/gold_set/` |
| Backup/migrate BD | `scripts/db/` |
| Exportar (S3, Excel) | `scripts/exports/` |
| Linear | `scripts/` (raíz) |

**NUNCA** crear `test_*.py` fuera de `tests/`.

---

## Estructura del Proyecto (Resumen)

```
MOL/
├── 01_sources/          # Scraping (bumeran/, zonajobs/, etc.)
├── database/            # BD, NLP processors, matching
│   ├── prompts/         # Prompts LLM
│   ├── patterns/        # Regex patterns
│   └── archive_old_versions/
├── config/              # JSONs de configuración
├── tests/               # Tests pytest
├── scripts/             # Utilidades
│   ├── db/              # BD
│   ├── nlp/gold_set/    # Optimización NLP
│   ├── matching/        # Optimización Matching
│   └── exports/         # Exportaciones
├── docs/                # Documentación
│   ├── guides/          # RUN_TRACKING, VALIDACION, OPTIMIZACION
│   └── reference/       # PIPELINE
├── fase3_dashboard/     # Fase 3: Dashboard y presentación
│   ├── nextjs/          # Dashboard Next.js (desarrollo)
│   └── docs/            # Docs específicos Fase 3
├── Visual--/            # Dashboard R Shiny (legacy)
└── run_scheduler.py     # Entry point scraping
```

---

## Modelos LLM/ML

| Modelo | Uso |
|--------|-----|
| **Qwen2.5:7b** | NLP: extracción semántica |
| **BGE-M3** | Matching: embeddings |
| **ChromaDB** | Skills lookup |

**Requisitos:**
- Ollama en `localhost:11434` con `qwen2.5:7b`
- ChromaDB con vectores en `database/esco_vectors/`

---

## Reglas de Desarrollo

1. **Scraping:** SIEMPRE `run_scheduler.py`, NUNCA scrapers directo
2. **Tests:** Todo cambio NLP/Matching debe pasar Gold Set
3. **Umbrales:** NLP >= 90%, Matching >= 95%
4. **Linear:** Usar cache (`scripts/linear_*.py`), NUNCA MCP directo

---

## Flujo de Branches

```
main                    ← Producción (solo via PR)
  └── develop           ← Integración (pasó Gold Set)
        ├── feature/optimization-nlp
        └── feature/optimization-matching
```

**NUNCA** push directo a `main`. **SIEMPRE** Gold Set antes de merge.

---

## Sesiones Paralelas de Claude Code

**Problema:** Dos sesiones de Claude en el mismo directorio se pisan los branches.
**Solución:** Cada sesión usa su propio **worktree** (misma BD git, distinta carpeta).

### Al iniciar una sesión nueva en paralelo

```bash
# 1. ANTES de abrir Claude, crear worktree para el branch
./scripts/worktree-session.sh create feature/nombre-del-branch

# 2. Abrir Claude Code en esa carpeta
cd /mnt/d/OEDE/Webscrapping-nombre-del-branch
claude

# 3. Si necesitás branch nuevo (se crea desde main)
./scripts/worktree-session.sh new feature/mi-nuevo-feature
```

### Al terminar la sesión

```bash
./scripts/worktree-session.sh remove feature/nombre-del-branch
```

### Reglas

- **NUNCA** hacer `git checkout` a otro branch si hay otra sesión activa
- Cada sesión trabaja en su carpeta, su branch, sin interferir
- Los commits de cualquier worktree son visibles desde todos (mismo repo)
- `./scripts/worktree-session.sh list` para ver sesiones activas

### Estructura de carpetas

```
/mnt/d/OEDE/Webscrapping/                      ← Sesión principal
/mnt/d/OEDE/Webscrapping-admin-arquitectura/    ← Sesión paralela
/mnt/d/OEDE/Webscrapping-otro-feature/          ← Otra sesión paralela
```

---

## Colaboracion Multi-Desarrollador

Este proyecto es trabajado por **multiples personas en distintas fases**.

**LEER:** `docs/guides/COLABORACION.md` para reglas de sync, division de trabajo y referencias a documentacion oficial de Claude Code.

---

## AI Platform Local

Plataforma en `D:\AI_Platform`.

```python
import httpx
GATEWAY = "http://localhost:8080"
```

| Endpoint | Descripción |
|----------|-------------|
| `POST /v1/chat/completions` | LLM |
| `POST /v1/embeddings` | Embeddings |

Docs: http://localhost:8080/docs

---

> **Última actualización:** 2026-01-16

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gbreard) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
