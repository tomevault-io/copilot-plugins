## clientedashtfg

> Guia permanente de trabajo para `DashClientModular4`.

# AGENTS.md

Guia permanente de trabajo para `DashClientModular4`.

## Estado actual

Phase 5 esta cerrada. `neural_abr_lite` esta integrado como guarded neural scorer controller, pero no hay benchmark, ranking ni afirmacion de mejora de QoE.

La fase activa es:

```text
Phase 6C-H1 - Live materialization robustness hotfix
```

Phase 6A2 congela el protocolo experimental final. Phase 6B implementa readiness/audit code: validacion de manifiesto, preflight estructural y cierre del gap `canonical_content_fingerprint`. Phase 6C automatiza source registry, adquisicion publica, extraccion, normalizacion, manifiestos, validacion, auditoria y freeze externo. Phase 6C-H1 endurece materializacion real con defaults primary-only, logs live, progreso acotado, timeouts, resume, skip-existing y clean-derived. Todavia no hay benchmark, ranking, graficas desde datos reales, CSVs de resultados, ganador ni afirmacion de mejora de QoE.

La ruta activa de documentacion de validacion es:

```text
docs/science/06_validation/
```

## Reglas permanentes

1. Mantener el cliente limpio, modular y testeable.
2. No tocar runtime, `player.py`, controladores, `core/trace_replay` ni `core/evaluation` sin una especificacion explicita.
3. No ejecutar benchmark, generar graficas desde datos reales, crear rankings, declarar ganadores ni afirmar mejora de QoE en Phase 6C.
4. No reentrenar NeuralABR-Lite ni cambiar `qoe_linear_v1` / `reward_n` en Phase 6C.
5. No meter datasets, modelos, runs, logs, PDFs, CSVs, JSONL, zips, videos, segmentos DASH, manifests reales, receipts, normalized traces ni otros artifacts generados en Git.
6. Mantener workspaces externos bajo `C:\Users\danie\Documents\TFG\_datasets`, `_models`, `_runs`, `_scripts`, `_literature`, `_audits` y `_archive`.
7. No pedir a Codex implementar directamente desde PDFs brutos. Usar primero indices, source cards, specs y documentos canonicos.
8. Usar `docs/INDEX.md`, `docs/science/PHASE_INDEX.md` y `docs/science/CANONICAL_DOCUMENTS.md` como entrada antes de abrir cientos de Markdown historicos.
9. Usar staging explicito. No usar `git add .`.
10. Todo cambio importante debe tener commit propio.

## Separacion tecnica

Mantener separados:

- parser MPD
- descarga de segmentos
- buffer
- motor de reproduccion
- control ABR
- logging
- evaluacion
- documentacion cientifica
- workspaces locales externos

## Evidence guardrail

Phase 6 debe bloquear cualquier traza vista en Phase 4 si se usa para evaluar `neural_abr_lite` frente a baselines. El bloqueo debe cubrir `trace_id`, `leakage_group`, `checksum_sha256` y `canonical_content_fingerprint` cuando este presente. Phase 4 teacher agreement/OOD queda como diagnostico, no como prueba fuerte de generalizacion.

En Phase 6C, `ready_for_phase6c` y el freeze externo no significan `ready_for_benchmark`; `ready_for_benchmark=false` y `benchmark_authorized=false` deben permanecer falsos. Phase 6C opera sobre external roots fuera del repo. Los IDs finales solo quedan congelados cuando exista el artifact externo `phase6_trace_manifest_final.json` despues de adquisicion, normalizacion, validacion, auditoria y freeze.

El primer run real debe usar `--sources primary` para Raca 4G LTE y Raca 5G. Lumos es opcional, Ghent/HSDPA son diagnosticos salvo seleccion explicita y Lancaster permanece excluido.

---
> Source: [DNLPRTL/ClienteDashTFG](https://github.com/DNLPRTL/ClienteDashTFG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
