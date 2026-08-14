## ds2api

> These rules apply to all agent-made changes in this repository.

# AGENTS.md

These rules apply to all agent-made changes in this repository.

## PR Gate

- Before opening or updating a PR, run the same local gates as `.github/workflows/quality-gates.yml`.
- Required commands:
  - `./scripts/lint.sh`
  - `./tests/scripts/check-refactor-line-gate.sh`
  - `./tests/scripts/run-unit-all.sh`
  - `npm run build --prefix webui`
- Running on Windows:
  - The shell scripts require Git Bash (Git for Windows).
  - If Git Bash is installed but `bash` is not on `PATH`, invoke the scripts from PowerShell with the full path to `bash.exe`, for example:
    ```powershell
    & "C:\Program Files\Git\bin\bash.exe" -c "./scripts/lint.sh"
    & "C:\Program Files\Git\bin\bash.exe" -c "./tests/scripts/check-refactor-line-gate.sh"
    & "C:\Program Files\Git\bin\bash.exe" -c "./tests/scripts/run-unit-all.sh"
    npm run build --prefix webui
    ```

## Go Lint Rules

- Run `gofmt -w` on every changed Go file before commit or push.
- Do not ignore error returns from I/O-style cleanup calls such as `Close`, `Flush`, `Sync`, or similar methods.
- If a cleanup error cannot be returned, log it explicitly.

## Change Scope

- Keep changes additive and tightly scoped to the requested feature or bugfix.
- Do not mix unrelated refactors into feature PRs unless they are required to make the change pass gates.

## Protocol Adapter Boundary

- Do not let OpenAI Chat, OpenAI Responses, Claude, Gemini, or other interface protocol formatting own shared business behavior.
- Normalize protocol-specific request shapes into the project standard request/turn model first, run shared business logic in one place, then render back to the target protocol at the boundary.
- Business logic that must stay globally consistent includes empty-output retry, thinking/reasoning handling, tool-call detection and policy, usage accounting, current-input-file injection, history persistence, file/reference handling, and completion payload assembly.
- If a behavior must differ by protocol, keep the difference as an explicit adapter/rendering concern and document why it cannot live in the shared normalized path.
- When adding a new feature, also verify that the Vercel path is wired into it.

## Documentation Sync

- When business logic or user-visible behavior changes, update the corresponding documentation in the same change.
- `docs/prompt-compatibility.md` is the source-of-truth document for the “API -> pure-text web-chat context” compatibility flow.
- If a change affects message normalization, tool prompt injection, prompt-visible tool history, file/reference handling, history split, or completion payload assembly, update `docs/prompt-compatibility.md` in the same change.

---
> Source: [ouqiting/ds2api](https://github.com/ouqiting/ds2api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
