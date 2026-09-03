## dcu-vllm-docker

> Keep the root directory minimal and introduce structure deliberately:

# Repository Guidelines

## Project Structure & Module Organization
Keep the root directory minimal and introduce structure deliberately:

- `docker/` for Dockerfiles, entrypoints, and container assets
- `docs/` for design notes and operational runbooks
- `scripts/` for reusable verification and export helpers
- `tests/` for automated coverage

If you add a new top-level directory, document its purpose in the pull request.

Current key entry points:

- `docker/vllm-offline/Dockerfile` for the offline DTK25 vLLM runtime image
- `docs/offline-vllm-image.md` for build, run, and delivery steps
- `Makefile` for the supported build, test, and export commands

## Build, Test, and Development Commands
Expose a small, stable command surface and prefer the existing `Makefile` entry points:

- `make build` to build images or artifacts
- `make test` to run the full test suite
- `make test-image IMAGE=<tag>` to run the offline image smoke checks
- `make save IMAGE=<tag>` to export the image tarball for offline delivery

Avoid one-off scripts in the repository root when a `Makefile` or `scripts/` entry would be clearer.

## Coding Style & Naming Conventions
Use 4 spaces for Python and shell-adjacent config examples; use language-default formatting tools once a stack is chosen. Name files and directories by purpose, not by date or author. Prefer lowercase, hyphenated names for Docker-related files and snake_case for Python modules. Keep environment-specific values out of committed files; use `.env.example` for non-secret placeholders.

## Testing Guidelines
Add tests alongside the first production code change. Mirror the code layout under `tests/` and use descriptive names such as `test_image_build.py` or `test_startup.sh`. New features should ship with at least one happy-path test and one failure-path test. If automation is not yet available, include manual verification steps in the PR.

## Commit & Pull Request Guidelines
No local git history is available in this checkout, so follow Conventional Commits until a repository-specific pattern emerges, for example `feat: add base vLLM Dockerfile` or `fix: handle missing ROCm device mapping`. PRs should explain the change, list verification steps, link related issues, and include logs or screenshots for behavior that affects runtime setup or container startup.

## Security & Configuration Tips
Do not commit secrets, tokens, or machine-specific IPs. Keep large model paths, accelerator settings, and registry credentials configurable through environment variables or documented local overrides.

## DCU Runtime Notes
For DTK25 DCU production runs, container startup must include `--network=host`, `--ipc=host`, `--device=/dev/kfd`, `--device=/dev/dri`, `--group-add=video`, `--cap-add=SYS_PTRACE`, and `--security-opt seccomp=unconfined`.

If the host provides vendor user-space at `/usr/local/hyhal`, mount it into the container as `-v /usr/local/hyhal:/opt/hyhal:ro`. This is required on the current production host stack; without it, `torch.cuda.is_available()` stays `False`, `rocminfo` shows only the CPU agent, and `hy-smi` reports `Open mkfd failed`.

Before starting `vllm serve`, validate the container runtime with `print-runtime`, `/opt/dtk-25.04.2/hip/bin/rocminfo`, and `/opt/dtk-25.04.2/.hyhal/bin/hy-smi`. Only proceed when PyTorch reports the GPU as available and the vendor tools can see the GPU agent without `mkfd` errors.

---
> Source: [WuChaoli/dcu-vllm-docker](https://github.com/WuChaoli/dcu-vllm-docker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
