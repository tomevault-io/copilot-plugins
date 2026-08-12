## agent-policy-broadcast

> Build a small, dependency-free tool that distributes one trusted policy to many AI-agent workspaces safely and audibly.

# AGENTS.md

## Purpose

Build a small, dependency-free tool that distributes one trusted policy to many AI-agent workspaces safely and audibly.

## Rules

- Default to read-only planning; writes require the explicit `apply` command.
- Never overwrite full local instruction files. Own only stable marker blocks.
- Refuse paths outside `allowed_roots` and refuse symlink write targets.
- Run adapter commands as argv with `shell=False`.
- Back up before mutation, use atomic writes, emit receipts, and keep rollback tested.
- Do not add secret distribution, remote policy fetching, Git mutation, force-push, merge or deployment.
- Keep the core Python-standard-library-only until a demonstrated need justifies a dependency.
- New adapters require fixtures for malformed output, duplicate workspaces and path-boundary failures.

## Required checks

```bash
PYTHONPATH=src python3 -m unittest discover -s tests -v
python3 -m py_compile src/agent_policy_broadcast/*.py
git diff --check
```

---
> Source: [q652983601/agent-policy-broadcast](https://github.com/q652983601/agent-policy-broadcast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
