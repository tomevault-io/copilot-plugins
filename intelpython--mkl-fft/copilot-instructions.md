## mkl-fft

> CI/CD workflows and repo automation.

# AGENTS.md — .github/

CI/CD workflows and repo automation.

## Workflows (source of truth)
- `conda-package.yml` — Intel channel conda build/test pipeline
- `conda-package-cf.yml` — conda-forge-oriented build/test pipeline
- `build-with-clang.yml` — clang compatibility checks
- `build_pip.yaml` — wheel build pipeline
- `pre-commit.yml` — lint/format checks
- `openssf-scorecard.yml` — security scanning

## Policy
- Treat workflow YAML as canonical for platform/Python matrices.
- Avoid doc claims about CI coverage unless present in workflow config.

---
> Source: [IntelPython/mkl_fft](https://github.com/IntelPython/mkl_fft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
