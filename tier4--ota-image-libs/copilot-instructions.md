## ota-image-libs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`ota-image-libs` is a Python library and CLI toolkit for creating, reading, verifying, and deploying OTA image version 1.

An OTA image of OTA image specification version 1 is a unique representation of an input system rootfs image, consists of:

- A blob storage of all the unique resources of the rootfs, distinguished by SHA256.
- A resource_table as the manifest of the blob storage.
- A file_table that records all the file entries of the rootfs.

## Commands for dev

This project uses [uv](https://docs.astral.sh/uv/) for project management, dependency management, and virtualenv management.

**Testing:**

```bash
uv run pytest                                                                       # Run all tests
uv run pytest tests/test_artifact_reader.py                                         # Run a specific test file
uv run coverage run -m pytest && uv run coverage combine && uv run coverage report  # With coverage
```

**Linting & Formatting (ruff):**

```bash
uv run ruff check src/ tests/         # Lint
uv run ruff check --fix src/ tests/   # Auto-fix lint errors
uv run ruff format src/ tests/        # Format
```

**Type Checking:**

```bash
uv run pyright src/
```

**Build:**

```bash
uv build            # both source dist and wheel package
uv build --sdist    # only source dist
uv build --wheel    # only wheel package
```

**Pre-commit:**

```bash
uv run pre-commit install           # install hooks (once after cloning)
uv run pre-commit run               # run on changed files only
uv run pre-commit run --all-files   # run all hooks manually
```

Hooks run on every commit: YAML/TOML validation, end-of-file fixer, trailing whitespace, ruff lint (with auto-fix), ruff format, and markdownlint (with auto-fix, config in `.markdownlint.yaml`).

## Architecture

### Project Layout

The repository contains two top-level packages under `src/`:

- **`ota_image_libs/`** — Core shared library for reading, writing, signing, and verifying OTA images.
- **`ota_image_tools/`** — Handy utility tools to work with OTA images.

### OTA image V1 support (`ota_image_libs/v1/`)

All schemas required for OTA image specification v1 are as follows:

| Module | Purpose |
| --- | --- |
| `artifact/` | Provides `packer` for bundling OTA image into OTA image artifact(requires Python ≥3.11), and `reader` for reading OTA image artifact |
| `image_index/` | `ImageIndex` — top-level OCI image index |
| `image_manifest/` | `ImageManifest` — per-image layer descriptors |
| `image_config/` | `ImageConfig`, `SysConfig` — image and system configuration |
| `file_table/` | SQLite-backed file metadata for each file in an OTA image |
| `resource_table/` | SQLite-backed blob/resource metadata |
| `index_jwt/` | JWT signing/verification schema and utilities |
| `otaclient_package/` | OTAClient release package format |

Within each module, besides schemas, utils for operating the metadata are also available.

### Blob Storage Optimization (`ota_image_libs/_resource_filter/`)

`_resource_filter` defines the `filter_applied` field schema used in the resource table to describe how a logical resource is stored in the blob store of the OTA image.
Each filter is serialized as `<code>:<msgpack-options>` bytes and registered via a filter registry.

| Filter | Code | Purpose |
| --- | --- | --- |
| `BundleFilter` | `b` | Resource is a slice of a larger bundle blob — stores `bundle_resource_id`, `offset`, `len` |
| `CompressFilter` | `c` | Resource is stored compressed — stores `resource_id` and `compression_alg` (e.g. `zstd`) |
| `SliceFilter` | `s` | Resource is reconstructed from an ordered list of sub-resource IDs — stores `slices` |

`FilterConfig` is the abstract base class; concrete types self-register via `register_filter()` and are resolved at parse time by `get_filter_type()`.

### Common Internal Shared Libs and Utils (`ota_image_libs/common/`, `ota_image_libs/_crypto/`)

- **`common/model_spec.py`** — Helper base classes and utils for using pydantic v2.
- **`common/metafile_base.py`** — Helper base classes for defining and parsing OTA metadata files using pydantic v2.
- **`common/db_utils.py`** — Shared helpers on top of `simple-sqlite3-orm` for using sqlite3.
- **`common/msgpack_utils.py`** — MessagePack serialization helpers.
- **`common/io.py`** — I/O utilities (e.g. zstandard/zstd compression wrappers).
- **`_crypto/jwt_utils.py`** — ES256 JWT signing and verification.
- **`_crypto/x509_utils.py`** — X.509 certificate chain validation.

### CLI Commands: ota-image-tools (`ota_image_tools/cmds/`)

Handy CLI for working with OTA image.

`list-image`, `inspect-index`, `inspect-blob`, `lookup-image`, `deploy-image`, `verify-sign`, `verify-resources`. All commands accept `-d`/`--debug` for debug logging.

## OTA Image Specification Version 1 Specification (`spec/`)

The `spec/` directory contains the authoritative OTA image version 1 specification.
Consult these when working on schemas or adding new features rather than reverse-engineering intent from code:

- `image_spec.md` — Overall OTA image specification overview
- `image_index.md`, `image_manifest.md`, `image_config.md`, `sys_config.md` — Per-component schemas
- `file_table.md`, `resource_table.md` — SQLite database schemas
- `annotations.md` — Standard annotation key definitions
- `index_jwt.md` — JWT signing specification
- `otaclient_package.md` — OTAClient release package format

## CI/CD

Three GitHub Actions workflows live under `.github/workflows/`:

**`test.yaml` — Test CI**

- Triggers on PRs to `main`/`v*`, pushes to `main`/`v*` (only when `src/`, `tests/`, or the workflow file changes), and manual dispatch.
- Runs `uv run coverage run -m pytest` across a matrix of Python 3.8–3.13 (`fail-fast: false`).
- On Python 3.13 only, runs a SonarCloud scan using the generated `coverage.xml`.

**`release.yml` — Release CI**

- Triggers on GitHub release (published) and manual dispatch.
- Builds the wheel using the minimum supported Python version (read from `.python-version`), uploads it as an artifact, calculates checksums, and attaches all `dist/*` files to the GitHub release.

**`gen_requirements_txt.yaml` — Requirements sync**

- For syncing uv.lock from pyproject.toml, and one-way exporting `requirements.txt` from uv.lock for snyk scan.
- Triggers on PRs to `main` when `pyproject.toml` changes, and manual dispatch.
- Re-generates `requirements.txt` from `pyproject.toml` via `.github/tools/gen_requirements_txt.py` and auto-commits the update to the PR branch if changed.

## Code Style

- **Line length**: 88
- **Target version**: `py38` (ruff `target-version` and pyright `pythonVersion`)
- **Linting rules**: pyflakes (`F`), flake8-quotes (`Q`), isort (`I`), flake8-bugbear (`B`), flake8-builtins (`A`), flake8-import-conventions (`ICN`), and `E4`/`E7`/`E9`
- **Ignored rules**: `E266` (multiple `#`), `E203` (whitespace before `:`), `E701` (multiple statements per line), `S101` (use of `assert`)
- **Docstring convention**: Google style (`tool.ruff.lint.pydocstyle`)
- **Type checking**: Pyright in `standard` mode targeting Python 3.8; tests and scripts are excluded from type checking

---
> Source: [tier4/ota-image-libs](https://github.com/tier4/ota-image-libs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
