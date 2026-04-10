## scientific-statgen-playbook

> This repository hosts two plugins for downstream statistical genetics workflows.

## scientific-statgen-playbook (Plugin Format)

This repository hosts two plugins for downstream statistical genetics workflows.

Scope note: this repository hosts two plugins:
1. `statgen-analysis` — GWAS analysis, fine-mapping, heritability, TWAS
2. `statgen-validation` — biological validation via OpenTargets, GWAS Catalog, Ensembl

Dependency contract:
1. `statgen-analysis` is self-contained for downstream analysis workflows.
2. `statgen-validation` is self-contained for biological validation queries.
3. Both plugins complement each other: run analysis first, then validate findings.
4. For upstream model design and structured development workflows, install `scientific-plan-execute` from the [scientific-software-playbook](https://github.com/mancusolab/scientific-software-playbook).

## Plugin Assets (Source Of Truth)

### Skills (`statgen-analysis`)
- `statgen-analysis`: `plugins/statgen-analysis/skills/statgen-analysis/SKILL.md`

### Skills (`statgen-validation`)
- `biological-validation`: `plugins/statgen-validation/skills/biological-validation/SKILL.md`

### Assets (`statgen-analysis`)
- Scripts: `plugins/statgen-analysis/scripts/`
- Reference docs: `plugins/statgen-analysis/reference/`
- Visualization: `plugins/statgen-analysis/visualization/`
- Examples: `plugins/statgen-analysis/examples/`
- Tests: `plugins/statgen-analysis/tests/`

### Assets (`statgen-validation`)
- Scripts: `plugins/statgen-validation/scripts/`
- Reference docs: `plugins/statgen-validation/reference/`
- Tests: `plugins/statgen-validation/tests/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CamelliaRui)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CamelliaRui)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
