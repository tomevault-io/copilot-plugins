## prompt

> Note: this repository currently contains no application source files. The guidance below is a concise, practical checklist and set of heuristics for AI coding agents to follow when the project files are present. When you see actual source files, replace placeholders with concrete paths (examples are given).

## Copilot / AI agent instructions — repository starter

Note: this repository currently contains no application source files. The guidance below is a concise, practical checklist and set of heuristics for AI coding agents to follow when the project files are present. When you see actual source files, replace placeholders with concrete paths (examples are given).

1. Quick repo reconnaissance (what to open first)

   - Look for language indicators: `package.json`, `pyproject.toml`, `requirements.txt`, `go.mod`, `Cargo.toml`, `pom.xml`, `Dockerfile`.
   - Check top-level directories: `src/`, `app/`, `packages/`, `services/`, `cmd/`, `infra/`, `tests/`.
   - Open CI files: `.github/workflows/*` and `Dockerfile` to learn build/test commands.

2. Big-picture architecture (how to infer it fast)

   - If there is a `services/` or `packages/` folder, treat this as a monorepo: map each package to its build/test script (look in each package's manifest).
   - If `cmd/` or `main.go` / `bin/` exists, the repo likely produces one or more CLIs or server binaries.
   - `web/`, `frontend/`, `client/` usually contains UI code (look for `vite`, `next`, or `react` package.json scripts).

3. Build/test/debug workflows (concrete commands to discover)

   - Preferred: read `package.json` scripts or `Makefile` to find canonical commands.
   - Common patterns to try when files are present:
     - Node: `npm ci && npm test` or `pnpm install && pnpm test` (check for `pnpm-lock.yaml`).
     - Python: create venv, `pip install -r requirements.txt`, run `pytest` (check `pyproject.toml` for `tool.poetry`).
     - Go: `go test ./...`, `go build ./...`.
   - If CI pipeline exists, mirror its steps locally — it is the authoritative sequence.

4. Project-specific conventions to look for

   - Linting and formatting: check for `eslint`, `prettier`, `black`, `gofmt` configurations and prefer those tools' defaults.
   - Tests: prefer `tests/` or `__tests__` locations; unit tests should be runnable via the repo's test script.
   - Configuration: environment-specific config often lives in `config/`, `env.example`, or `.env.example`.

5. Integration points and external services

   - Search for connectors and clients: files importing AWS/Azure/GCP SDKs, database drivers (`pg`, `mysql`, `typeorm`, `mongoose`), or `requests`/`httpx` in Python.
   - Look for `terraform/`, `bicep/`, or `arm/` directories for infra-as-code.

6. How to change code safely (PR-friendly steps)

   - Mirror CI: make the same build/test steps locally before editing.
   - Add small, focused commits and include tests for behavioural changes. If you can't run integration tests (external dependencies), mock them or add unit tests.

7. Examples of specific cues an AI should follow (when present)

   - If `src/index.ts` imports `./services/user`, prefer editing `services` layer for business logic changes rather than `src/index.ts` server wiring.
   - If `api/` contains OpenAPI or `schemas/`, prefer generating client/server code from those contracts.

8. When updating this file
   - Preserve any existing, project-specific guidance already in this file.
   - Replace the reconnaissance checklist with concrete commands discovered in CI or top-level manifests.

If anything above is unclear or you'd like me to tailor this to the real project layout, paste the repository files (or grant access to the workspace) and I'll update this with concrete file paths and exact build/test commands.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/madziaar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
