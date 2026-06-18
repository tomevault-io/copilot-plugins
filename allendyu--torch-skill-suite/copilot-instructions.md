## torch-skill-suite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Torch Skill Suite** is a Claude Code skill suite for automating PyTorch deep learning engineering workflows. It consists of six specialized skills that follow a sequential, contract-driven workflow:

1. `torch-data` — Data engineering (Dataset/DataLoader, preprocessing, data validation)
2. `torch-model` — Model construction (backbone selection, head design, model scaffolding)
3. `torch-train` — Training loop generation (optimizer, scheduler, checkpointing, logging)
4. `torch-eval-tune` — Validation and tuning (metrics, error analysis, hyperparameter suggestions)
5. `torch-infer-deploy` — Inference and deployment (TorchScript/ONNX export, FastAPI service)
6. `torch-engineering` — Engineering extension and standardization (project structure, testing, CI)

## Architecture

### Skill-Based Design
Each skill is a self-contained Claude Code skill located in `torch-skill-suite/.claude/skills/`. Skills communicate via shared **contract files** (YAML) that define interfaces between stages. The recommended flow is sequential (1→6), but skills can be used independently if contracts are available.

### Contract-Driven Workflow
The suite uses four core contract schemas (JSON Schema) to ensure consistency across skills:
- `shared/schemas/project_spec.schema.json` — Global task definition
- `shared/schemas/data_contract.schema.json` — Data specification (input shape, splits, preprocessing)
- `shared/schemas/model_contract.schema.json` — Model specification (backbone, head, loss compatibility)
- `shared/schemas/deploy_contract.schema.json` — Deployment specification (export format, service type)

Canonical example contracts are in `shared/contracts/` and use the `*.example.yaml` suffix; each file is a single schema-valid contract that can be used directly as a scaffold input. Scenario examples and recipes are in `shared/examples/contracts/`. Runtime skill outputs normally use unsuffixed names such as `data_contract.yaml`, `model_contract.yaml`, and `deploy_contract.yaml`. Each skill consumes contracts from previous stages and produces contracts for downstream stages.

### Skill Boundaries
Each skill's responsibilities and boundaries are documented in its `SKILL.md` file (e.g., `torch-skill-suite/.claude/skills/torch-data/SKILL.md`). Key principles:
- `torch-data` owns data preparation, not model design.
- `torch-model` owns model structure, not training policy.
- `torch-train` owns training loops, not deployment.
- Skills hand off via explicit contracts to avoid ambiguity.

## Common Development Tasks

### Validating Contracts
Run validation commands from the package directory (`torch-skill-suite/`). The canonical examples use the `*.example.yaml` suffix:
```bash
cd torch-skill-suite
python .claude/skills/torch-data/scripts/validate_contract.py --contract shared/contracts/data_contract.example.yaml
python .claude/skills/torch-model/scripts/validate_contract.py --contract shared/contracts/model_contract.example.yaml
python .claude/skills/torch-infer-deploy/scripts/validate_contract.py --contract shared/contracts/deploy_contract.example.yaml
```

### Running Tests
Run the test suite from the package directory because `pytest.ini` lives there:
```bash
cd torch-skill-suite
python -m pytest
```

Current health check: `199 passed` on 2026-05-08. The suite currently emits PyTorch deprecation warnings around `torch.jit.trace`, `torch.jit.save`, and `torch.jit.load`; these do not fail tests, but deployment export code should track the future `torch.export` migration path.

### Inspecting Datasets
The inspection script attempts to infer dataset format:
```bash
python torch-skill-suite/.claude/skills/torch-data/scripts/inspect_dataset.py --path /path/to/dataset --data_type image --task_type classification
```

### Running Skills
Skills are automatically triggered when user requests match their descriptions (see system-reminder). To manually invoke a skill, use the `/skill` command with the skill name (e.g., `/skill torch-data`).

## Important Directories

- `torch-skill-suite/.claude/skills/` — Skill definitions (SKILL.md files, scripts, templates, tests)
- `torch-skill-suite/shared/schemas/` — JSON Schema files for contracts
- `torch-skill-suite/shared/contracts/` — Canonical schema-valid `*.example.yaml` contract scaffolds
- `torch-skill-suite/shared/examples/contracts/` — Scenario contract examples (per modality/task)
- `torch-skill-suite/shared/python/torch_skill_shared/` — Shared Python utilities (`yaml_utils`, `model_builder`)
- `torch-skill-suite/shared/route_map.yaml` — Authoritative (data_type, task_type) → model route table with priority and support status
- `torch-skill-suite/docs/` — Architecture, workflow, and MVP roadmap documentation

## Current State

- **Supported routes**: Five end-to-end routes are fully implemented across `torch-data` → `torch-model` → `torch-train` → `torch-eval-tune` → `torch-infer-deploy`: `image_classification` (P0), `text_classification`, `image_segmentation`, `tabular_classification`, `tabular_regression` (P1). Additional P2/P3 routes (detection, time series, audio, video, multimodal, generation) have data-side support and route definitions but no model/template implementations yet — see `shared/route_map.yaml` for the authoritative status matrix.
- **`torch-engineering` status**: Skill description and boundaries are documented in `SKILL.md`, but no scripts, templates, or tests are implemented yet. The other five skills each ship scripts and tests.
- **Validation**: The package test suite passes (`199 passed` as of 2026-05-08) when run from `torch-skill-suite/`; canonical contract examples and the `project_spec`/`data`/`model`/`deploy` schemas all validate cleanly.
- **Documentation**: `docs/` describes architecture, workflow, and MVP status; per-skill roadmaps live in each skill's `mvp_plan.md`; `torch_skill_suite_plan.md` in the repository root is the original comprehensive design.
- **Known warnings**: Deployment-side tests emit PyTorch deprecation warnings for `torch.jit.trace`, `torch.jit.save`, and `torch.jit.load`; tests still pass, but the deployment path should track migration to `torch.export`.

## Key Design Principles

1. **Minimal reliable pipelines** over speculative abstractions.
2. **Explicit interfaces** via contract files.
3. **Observations over assumptions** — skills should inspect existing project structure before generating code.
4. **Downstream compatibility** — each skill’s output should be directly usable by the next skill.
5. **Template-based generation** where possible, with Claude filling parameters and adapting structure.

## Notes for Future Claude Code Instances

- Always check for existing contract files (`data_contract.yaml`, `model_contract.yaml`, etc.) before generating code.
- Prefer extending existing project structure over wholesale replacement.
- When a skill is triggered, read its SKILL.md to understand boundaries and collaboration patterns.
- The shared schemas are authoritative for contract validation; use them to ensure interoperability.
- The ultimate goal is a fully automated PyTorch workflow where users can start with raw data and end with a deployed, engineered project via sequential skill invocations.

---
> Source: [allendyu/torch-skill-suite](https://github.com/allendyu/torch-skill-suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
