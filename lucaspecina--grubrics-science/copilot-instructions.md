## grubrics-science

> Entrena Qwen3-8B con RL (GRPO) para generar rúbricas de evaluación médica y científica. La señal de reward es *functional alignment*: correlación de Spearman entre los rankings del Judge (GPT via Azure) y los gold_scores de médicos/expertos humanos.

# GRubrics

Entrena Qwen3-8B con RL (GRPO) para generar rúbricas de evaluación médica y científica. La señal de reward es *functional alignment*: correlación de Spearman entre los rankings del Judge (GPT via Azure) y los gold_scores de médicos/expertos humanos.

## Stack

- **Modelo**: Qwen3-8B + LoRA (rank 64)
- **RL framework**: veRL (GRPO) | **SFT**: TRL + LoRA
- **Rollout**: vLLM (H100)
- **Judge**: GPT-4.1 via Azure OpenAI, scoring binario a-la-HealthBench (1 call/criterion, pass/fail) — CHG-021
- **Tracking**: wandb | **Env**: `conda activate RL`

## Workflow de desarrollo

- **Desarrollo local**: Windows — editar código, leer logs, planear experimentos
- **Ejecución**: H100 remota (Linux, Azure ML) — training, precompute, baselines
- **SSH directo**: Claude puede ejecutar comandos via `ssh azure-ml "comando"` (key auth, sin password)
- **Env de training**: `conda activate RL` (siempre, en la H100)
- **Dinámica**: editar local → push → Claude ejecuta en H100 via SSH (o el usuario reporta)
- **Nunca asumir** que un comando se puede ejecutar localmente — preguntar siempre si hay duda
- **Setup de la VM**: ver `docs/h100-setup.md` para reproducir el entorno desde cero

## Los tres actores

- **GRubrics** (Qwen3-8B + LoRA) — se entrena, genera rúbricas
- **Judge** (GPT fijo) — evalúa respuestas con la rúbrica generada
- **Answer Policy** (GPT fijo) — generó las respuestas pre-computadas (offline, en `data/cache/`)

## Convenciones del repo

- Configs en `configs/` — GRPO: `verl_grpo.yaml`, SFT: `sft_healthbench.yaml`
- Presets de datos: `configs/training_presets.yaml` (`open_only` default, `verifiable_only`, `curriculum`, `full_mix`)
- Checkpoints: `checkpoints/grubrics-transfer/`
- Cache precompute: `data/cache/*.jsonl` — **NO borrar**, cada run cuesta $
- Vars de entorno: `AZURE_API_BASE`, `AZURE_API_KEY`, `AZURE_API_VERSION`, `RUBRIC_JUDGE_MODEL`, `REWARD_LAMBDA_*`
- Tests: `pytest tests/ -v` (181 tests, todos deben pasar antes de commitear)

## Dónde está cada cosa

- `grubrics_science/rewards/grubrics_reward.py` — reward function unificada (async)
- `grubrics_science/judge/judge.py` — Judge async con rate limiting, retry, cache
- `grubrics_science/data/adapters/` — 7 adapters (healthbench, medqa, medmcqa, gsm8k, math, frontierscience)
- `run_sft.py` / `run_grpo.py` — launchers principales
- `notebooks/analyze_rubrics.ipynb` — análisis post-training
- `scripts/` — download_datasets, run_baselines, validate_judge, analyze_precompute
- `scripts/validate_e2e_pipeline.py` — validación E2E completa: SFT→GRPO→Resume (~35 min en H100)
- `docs/h100-setup.md` — guía para reproducir el entorno de la H100 desde cero

## Guardrails de evaluación y testing

Reglas para evitar falsos negativos cuando se evalúan modelos, judges, o componentes del pipeline:

1. **Timeout generoso siempre**: usar `--timeout 300` (mínimo) en cualquier script que llame a la API. Los modelos de reasoning (gpt-5-mini, o1, etc.) pueden tardar >120s. Los modelos estándar (gpt-4.1, gpt-4o) también pueden ser lentos en prompts largos. **Nunca usar el default sin verificar cuál es.**

2. **Guardar output siempre**: usar `--output` para guardar resultados detallados. Sin output no se puede diagnosticar si un resultado malo fue por el modelo o por un error de infraestructura (timeout, parse failure, rate limit).

3. **Verificar scores sospechosos**: accuracy=0.000 o kappa=0.000 son señales de rotura, no de modelo malo. Antes de concluir que un modelo "no sirve", revisar:
   - ¿Cuántas entries fueron evaluadas vs skipped?
   - ¿Cuántas tuvieron parse failure (score=0.0 por default)?
   - ¿Los raw responses del modelo tienen sentido?

4. **Parse failures silenciosos**: `judge.py:_parse_response` devuelve `[0.0]` cuando el JSON no parsea. Esto NO se reporta como error — la entry cuenta como evaluada con score 0. Revisar warnings del logger.

5. **Reproducir antes de concluir**: si un resultado contradice lo esperado (ej: modelo que funciona para todos menos para nosotros), repetir con parámetros controlados antes de documentar como hallazgo.

**Ref**: CHG-020 (timeout fix), EXP-JUDGE-001 (resultados GPT-4.x posiblemente artefacto).

## Comportamientos conocidos de veRL

Estos no son bugs sino comportamientos del framework que hay que tener en cuenta:

- **veRL JSON columns**: parche auto-aplicado al cargar datos en `rl_dataset.py`
- **Judge cache en RL**: siempre `max_cache_size=0` durante training (RAM unbounded si no)
- **veRL auto-resume + total_training_steps absoluto**: veRL detecta checkpoints en `default_local_dir` y resume automáticamente. `total_training_steps` es absoluto (no relativo al checkpoint). Borrar el directorio de checkpoints antes de un run from scratch con pocos steps.
- **veRL checkpoints**: guarda 3 formatos en cada step: FSDP shard (`model_world_size_*.pt`), HuggingFace (solo config+tokenizer en `huggingface/`), LoRA adapter (`lora_adapter/`). Resume usa el FSDP shard, no HF format.
- **Checkpoint save time**: ~150-185s por step (~80% del step time con batch=4). No es un bug — usar `save_freq` alto en producción.
- **veRL config `reward:`**: `custom_reward_function` debe estar bajo `reward:` key, no top-level.
- **TRL compatibility**: veRL 0.7.1 requiere TRL ≤0.15.2 (0.29+ rompe imports).
- **SFT `remove_unused_columns`**: TRL 0.15.2 necesita `remove_unused_columns=true` para evitar error con columnas no-tensor.

Para bugs y blockers activos ver `TODO.md`.

## Pipeline validado (E2E)

El pipeline completo fue validado el 2026-03-19 en H100:

1. **SFT** (`run_sft.py`): Qwen3-8B + LoRA → merge → checkpoint HF merged
2. **GRPO** (`run_grpo.py`): carga SFT checkpoint → LoRA fresco → training con FSDP+vLLM
3. **Resume**: veRL auto-detecta `latest_checkpointed_iteration.txt` → carga FSDP shard → continúa

Script de validación: `python scripts/validate_e2e_pipeline.py` (~35 min)

Para reproducir el entorno desde cero: `docs/h100-setup.md`

## Docs de referencia

- `WIKI.md` — **LA referencia del proyecto**: qué es, para quién, cómo funciona el experimento, datos, stack, resultados, riesgos y hoja de ruta. Conceptual→técnica, accesible a cualquiera. **Mantener SIEMPRE actualizada**: cualquier cambio de diseño, resultado nuevo o decisión de rumbo se refleja acá en la misma sesión (es el documento que el usuario guarda en sus notas y comparte)
- `TODO.md` — source of truth de pendientes (IDs: `TODO-NNN`)
- `CHANGELOG.md` — decisiones de diseño y cambios significativos (IDs: `CHG-NNN`)
- `docs/theoretical-foundations.md` — **cadena argumental del paper: claims con evidencia, qué funciona/qué no, contraargumentos** (mantener al incorporar evidencia nueva)
- `docs/adversarial-evaluation-reframing.md` — **reframing candidato 2026-07: evaluación adversarial (atacante/defensor co-evolucionados)** — PROPUESTA pendiente de verificación (TODO-017); leer antes de cualquier decisión de rumbo
- `docs/phase0-plan.md` — plan operativo de la Fase 0 (etapas A-D)
- `docs/performance-profile.md` — **referencia viva de profiling, bottlenecks y optimizaciones** (mantener actualizado)
- `docs/data-guide.md` — **guía de datos, splits y flujos** (leer antes de tocar datos o training)
- `docs/experiment-log.md` — bitácora cronológica de runs y resultados (IDs: `EXP-xxx`)
- `docs/research.md` — framing del paper, preguntas de investigación, landscape de la literatura
- `docs/related-work.md` — revisión de literatura detallada
- `docs/h100-setup.md` — **guía de setup de la VM H100 desde cero** (driver, conda, paquetes, validación)

## Mantenimiento de documentación y skills — CRÍTICO

**Esta sección es obligatoria. Cada vez que en la conversación aparece información nueva
que debería quedar documentada, actualizá el archivo correspondiente SIN esperar que
el usuario lo pida.** Proponer las actualizaciones proactivamente es parte fundamental
del workflow. No hacerlo degrada la calidad del proyecto entre sesiones.

### Archivos y cuándo actualizar

| Archivo | Qué contiene | Cuándo actualizar |
|---------|-------------|-------------------|
| `TODO.md` | Pendientes con IDs `TODO-NNN` | Nuevo bug, blocker, run pendiente, extensión |
| `CHANGELOG.md` | Decisiones y cambios con IDs `CHG-NNN` | Decisión de diseño, cambio de approach, por qué se descartó algo |
| `docs/experiment-log.md` | Resultados de runs con IDs `EXP-xxx` | Resultado o aprendizaje de un experimento |
| `CLAUDE.md` | Convenciones del repo y workflow | Nueva convención, cambio de stack o workflow |
| `WIKI.md` | **LA referencia viva del proyecto** (conceptual→técnica) | **SIEMPRE**: cambio de diseño, resultado nuevo, decisión de rumbo, cambio de datos/stack/costos — en la misma sesión en que ocurre |
| `docs/research.md` | Framing del paper y preguntas de investigación | Avance o respuesta a una pregunta de investigación |
| `.claude/skills/*/SKILL.md` | Guías operativas | Cambio en workflow operativo (debug, precompute, run, eval, dataset, h100) |

### Estructura del TODO.md

`TODO.md` tiene 4 niveles jerárquicos. **Los estratégicos se resuelven primero** porque informan todo lo demás:

1. **Investigaciones estratégicas** (TODO-001..003) — preguntas de arquitectura que informan decisiones concretas. Se resuelven investigando, no ejecutando. Al resolverse, actualizar los milestones y runs que dependen de ellas.
2. **Pipeline milestones** (TODO-004..005) — hitos concretos del pipeline. Dependen de las investigaciones.
3. **Runs core** (TODO-006..010) — experimentos a ejecutar. Dependen de los milestones.
4. **Extensiones** (TODO-011) — post-core, no bloquean nada.

**Estados y transiciones**:
- 🟡 pendiente → 🟢 en curso (cuando se empieza a trabajar activamente)
- 🟢 en curso → ✅ hecho (cuando se completa, agregar fecha y resultado)
- 🔴 bloqueado → 🟡 pendiente (cuando se desbloquea, porque el blocker se resolvió)
- Al resolver un TODO, revisar si otros TODOs que dependían de él cambian de 🔴 a 🟡.

**Dependencias**: se expresan con "Bloqueado por: TODO-NNN" y "Bloquea: TODO-NNN". Cuando un blocker se resuelve, actualizar los dependientes.

**Nuevo contenido**: asignar el siguiente ID en la sección correspondiente. Preferir absorber en un TODO existente antes de crear uno nuevo — mantener la lista corta y estratégica, no granular.

### Sistema de cross-references

Los documentos se conectan mediante IDs con formato `{PREFIX}-{NNN}`:

| Prefijo | Archivo | Ejemplo |
|---------|---------|---------|
| `TODO-NNN` | `TODO.md` | TODO-001, TODO-004 |
| `CHG-NNN` | `CHANGELOG.md` | CHG-010, CHG-014 |
| `EXP-xxx` | `docs/experiment-log.md` | EXP-001, EXP-DEBUG-A, VAL-003 |

**Cómo referenciar**: usar el ID inline en cualquier doc. Ejemplo:

```
En TODO.md:   "Depende de: TODO-001. Refs: CHG-011, EXP-DEBUG-A"
En CHANGELOG:  "Refs: TODO-005, EXP-DEBUG-A"
En experiment-log: "Refs: CHG-012, TODO-004"
```

### Reglas

1. **Proactividad**: si algo nuevo surge en la conversación y es relevante para alguno
   de los archivos de arriba, actualizalo o proponé actualizarlo. NO esperar a que lo pidan.
2. **Proporcionalidad**: no todo merece actualización en todos lados. Fixes menores a herramientas
   auxiliares (notebook, scripts de visualización, etc.) no requieren actualizar toda la documentación.
   Reservar actualizaciones multi-archivo para cambios que afecten el pipeline principal, decisiones
   de diseño, o resultados de experimentos.
3. **Leer antes de escribir**: antes de editar cualquier doc, leerlo para no pisar contenido existente.
   No leer preventivamente al inicio de cada sesión — solo cuando vayas a escribir.
4. **Contradicciones**: si la conversación contradice lo documentado, preguntar:
   "¿Querés que actualice [archivo] con esto, o es solo para esta sesión?"
5. **Skills**: los archivos en `.claude/skills/*/SKILL.md` son guías operativas. Si un workflow cambia
   (nuevo paso, fix, problema descubierto, cambio de approach), actualizar el skill correspondiente.
   Skills disponibles: `debug-grpo`, `eval-results`, `new-dataset`, `precompute`, `run-experiment`, `h100-workflow`.
6. **Cross-refs**: al actualizar un doc, agregar refs a IDs relevantes de otros docs.
   Un problema nuevo puede requerir: `TODO.md` (pendiente), `CHANGELOG.md` (decisión), skill (guía operativa),
   `experiment-log.md` (resultado).
7. **Propagación de estado**: cuando un TODO cambia de estado (especialmente a ✅), revisar:
   - ¿Hay TODOs bloqueados por este que ahora se desbloquean?
   - ¿Hay que crear un CHG en CHANGELOG.md para documentar la decisión/cambio?
   - ¿Algún skill se ve afectado?

---
> Source: [lucaspecina/grubrics-science](https://github.com/lucaspecina/grubrics-science) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
