## pure-lisp-kernel-lab

> 1. Read `.agents/index.md`, then only the assigned scope contract and referenced inputs.

# Agent contract

1. Read `.agents/index.md`, then only the assigned scope contract and referenced inputs.
2. Treat `.agents/roadmap.json` as the only task and status authority.
3. The coordinator dispatches every conflict-free ready wave; a worker executes exactly one assigned scope and does not delegate.
4. Edit only `owns`; treat `readPaths` as immutable. Stop on overlap or an undeclared dependency.
5. Prefer deterministic unary pure functions and isolate filesystem, process, clock, Git, and network effects at named boundaries.
6. Never emit `(safety 0)`. Every specialized domain needs explicit boundary tests and a retained reference implementation or fallback.
7. Run every scope verification gate before handoff. Performance claims require fresh semantic and measured SBCL evidence.
8. Never expose secrets, personal paths, credentials, generated keys, toolchains, binaries, or unrelated source artifacts.
9. Do not perform web search unless the user explicitly asks for it or confirms it.
10. Only the coordinator may edit `.agents/*.json`, `.gitignore`, this file, stage, commit, tag, push, or create a release.
11. Return `scopeId`, outputs, verification evidence, changed paths, risks, and contract questions.

## PROJECT_VARIABLES

- project: pure-lisp-kernel-lab
- roadmap: .agents/roadmap.json
- handoff: .agents/handoff.json
- changelog: .agents/changelog.json
- dispatch: python3 "${CODEX_HOME}/skills/orchestrate-project-agents/scripts/dispatch_wave.py" .
- roadmap-validator: python3 scripts/roadmap-check.py
- trace: CODEX_TRACE=0
- verify: python3 scripts/test-all.py

---
> Source: [bibi-chat/pure-lisp-kernel-lab](https://github.com/bibi-chat/pure-lisp-kernel-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
