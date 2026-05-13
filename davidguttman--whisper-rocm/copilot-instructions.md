## whisper-rocm

> - Source and diagnostic scripts (`test_ct2_full.py`, `run_whisper.sh`) sit in the repo root; keep new tooling alongside them unless it is container-specific.

# Repository Guidelines

## Project Structure & Module Organization
- Source and diagnostic scripts (`test_ct2_full.py`, `run_whisper.sh`) sit in the repo root; keep new tooling alongside them unless it is container-specific.
- `docker_rocm/` holds container assets; name new patches `*.patch` and base images `Dockerfile.<target>` to match existing patterns.
- Model caches (`whisper-*-ct2/`) and install outputs (`ctranslate2-install/`) stay untracked; update `.gitignore` before introducing new artifact directories.

## Build, Test, and Development Commands
- Create the environment with `uv venv --python 3.12 .venv` and activate via `source .venv/bin/activate`.
- Install dependencies using `uv pip install -r requirements.txt` plus `uv pip install torch --index-url https://download.pytorch.org/whl/rocm6.2` for GPU wheels.
- Run the helper `./run_whisper.sh jfk.wav tiny` to convert models and launch an end-to-end transcription.
- Smoke tests: `python test_ct2_full.py jfk.wav large-v3` for GPU throughput, `python test_ct2_gpu.py` for device checks, and `./run_rocm_test.sh` for the Docker path.

## Coding Style & Naming Conventions
- Follow PEP 8: 4-space indentation, snake_case functions, and module-level constants in UPPER_CASE.
- Prefer docstrings and f-strings for logging; reuse the structured print blocks already in `test_ct2_full.py`.
- Shell helpers should stay Bash-compatible with `set -e`, uppercase exports, and explicit path guards (`$CT2_INSTALL_PREFIX`, `$ROCBLAS_TENSILE_LIBPATH`).

## Testing Guidelines
- Before GPU tests export `ROCBLAS_TENSILE_LIBPATH=/opt/rocm/lib/rocblas/library` and prepend `$CT2_INSTALL_PREFIX/lib` to `LD_LIBRARY_PATH`.
- Name new checkers `test_<focus>.py` so they can run via `python file.py audio.wav model`.
- Capture timing and real-time factor metrics in test output to preserve performance baselines.
- Validate Docker changes with `./run_rocm_test.sh` to confirm host device mounts.

## Commit & Pull Request Guidelines
- Keep commit subjects imperative and concise (e.g., “Add ROCm large-v3 benchmark”); limit the first line to ~70 characters.
- Document environment or driver requirements in the body when they affect reviewers’ setup.
- Pull requests should include the commands used for verification, any notable performance deltas, and references to linked issues.
- Re-run at least one GPU smoke test before requesting review and report the hardware target (gfx1151 vs CPU).

---
> Source: [davidguttman/whisper-rocm](https://github.com/davidguttman/whisper-rocm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
