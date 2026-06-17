## git-commit

> Generate and apply Angular-style Conventional Commits for all staged changes in main repo and submodules.


# Create Commit Messages for All Repositories

MUST analyze staged changes across all repositories and generate Angular-style commits individually.

## Terminology

- **Main Repo**: The top-level Git repository where the user initiated the command.
- **Sub Repo**: Any nested Git repository tracked via `.gitmodules`.

## Core Principles
- MUST operate using the cognitive loop: Think > Try > Summarize > Record.
- [Think] MUST build a holistic mental router map of the ENTIRE SKILL before starting. Use elimination to discard impossible approaches to minimize failure costs. Perform a forward prediction to avoid unresolvable states.
- [Try] MUST verify current environment boundaries BEFORE taking action. Execute steps sequentially without combining commands.
- [Summarize] MUST verify actual outcomes; hallucinating success is forbidden. If an attempt fails, MUST NOT prematurely abort. Instead, diagnose the root cause, zoom out to the router map, and backtrack to a safe state to try an alternative.
- [Record] MUST explicitly state variables and context to carry over when passing control to the next state or file.
- MUST treat these rules as strict guardrails. Autonomous optimization within these boundaries is expected.

## Execution Workflow

1. **[Think] Scope Discovery**: MUST identify all repository boundaries. Execute `git rev-parse --show-toplevel` to define the Main Repo absolute path.
2. **[Think] Sub Repo Identification**: IF `.gitmodules` exists in the Main Repo root, MUST execute `SUBMODULES.md` passing the Main Repo path to identify and process Sub Repos.
3. **[Record] Handoff to Main Repo**: MUST execute `MAIN_REPO.md`, explicitly carrying over any completed Sub Repo updates context to process the Main Repo.

---
> Source: [dyphn1/git-commit](https://github.com/dyphn1/git-commit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
