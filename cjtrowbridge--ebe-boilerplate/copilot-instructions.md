## ebe-boilerplate

> This repository is a forkable Ebitengine application boilerplate centered on one primary workflow and one deliberately unfinished follow-on workflow:

# Ebitengine Build Pipeline Boilerplate — Agent Instructions

## Repository purpose

This repository is a forkable Ebitengine application boilerplate centered on one primary workflow and one deliberately unfinished follow-on workflow:

1. **Primary — local build pipeline.** `scripts/build.py` (or `make build` on Unix) performs a no-flag build for every target the current host can support. This is the normal pipeline and the main product of the repository.
2. **Secondary — deploy pipeline.** Deployment is intentionally a setup point for the forker. Release destinations, signing, credentials, packaging, and CI providers are not configured here; do not invent or activate them without explicit operator direction.

`agentic-pipelines/` and the Ollama configuration files are optional scaffolding for people who want to add LLM-assisted governance or automation. They are not required to build, verify, review, or use this boilerplate.

## Agent boundaries

- Use this session's review and deterministic local checks for ordinary review work.
- Do not contact a configured Ollama or other inference endpoint unless the operator explicitly requests that operation.
- Only `scripts/build.py` and its direct callers perform builds. `pipeline.py run` is optional governance scaffolding and never performs a host build.
- Keep deploy configuration empty until the operator selects a target, credentials mechanism, signing policy, and release destination.

All agent task routing and framework invariants for optional Agentic Pipelines work are at [agentic-pipelines/AGENTS.md](agentic-pipelines/AGENTS.md). Use them only when operating or changing that optional subsystem. The framework is mounted at `./agentic-pipelines`.

## Host build contract

- **No-flag build entrypoint:** `make build` (Unix) or `scripts\build.py` (cross-platform). No target flags, environment variables, or config files are required.
- **Artifact layout:** `releases/{goos}/{goarch}/app/latest[.exe]`; Windows gets `.exe`, and a future Android target will use `.apk`.
- **Target feedback:** the build preflight lists every matrix target, its intended output path, whether it is possible locally, and why.
- **Go version:** Go 1.26.4 with Ebitengine v2.9.9, pinned in `go.mod`.

## Optional Ollama scaffolding

`api.sample.yaml` is an example local-Ollama configuration. A user may copy it to ignored `api.yaml` and explicitly run the bootstrap scripts plus `pipeline.yaml` if they want the optional governance demo. It is never a prerequisite for `scripts/build.py`, `make build`, or deployment setup.

## Reference-only content

`third_party/apparat/` is a local reference checkout only. It and nested submodules must remain ignored and untracked; they are never imported, built, or executed.

---
> Source: [cjtrowbridge/ebe-boilerplate](https://github.com/cjtrowbridge/ebe-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
