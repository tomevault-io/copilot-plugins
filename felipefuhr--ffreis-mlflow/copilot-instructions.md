## ffreis-mlflow

> **This repo:** `ffreis-mlflow` — builds the MLflow tracking and artifacts server

# Agent Context

**This repo:** `ffreis-mlflow` — builds the MLflow tracking and artifacts server
container image. Deployment manifests live in `stock/ffreis-infrastructure`.

## Non-obvious facts

- **This repo builds the image; deployment is in a different repo.** Changes to the
  image here must be coordinated with `stock/ffreis-infrastructure` to update the
  manifest image tag.

- **Runs as non-root (uid 2000).** Do not change this — it is a security requirement.

- **Exposes port 8787** (non-default for MLflow). All manifests and client configs
  assume this port.

- **`uv.lock` is locked for reproducibility.** If you update `pyproject.toml`, also
  update `uv.lock` (`uv lock`). CI does not auto-sync.

- **Trivy scans for HIGH/CRITICAL CVEs** in CI. New base image versions may introduce
  findings that block the build.

## Structure

```
container/mlflow/
  Dockerfile          ← multi-stage build
  pyproject.toml
  uv.lock
```

## Build/run

```bash
make build       # produces mlflow image
make clean-images
```

## Keeping this file current

- **If you discover a fact not reflected here:** add it before finishing your task.
- **If something here is wrong or outdated:** correct it in the same commit as the code change.
- **If you rename a file, command, or concept referenced here:** update the reference.

---
> Source: [FelipeFuhr/ffreis-mlflow](https://github.com/FelipeFuhr/ffreis-mlflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
