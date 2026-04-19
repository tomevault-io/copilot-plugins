## devpipe

> At the start of every response, append this emoji to the global sequence: 🔄


At the start of every response, append this emoji to the global sequence: 🔄

This project enforces the test-first, test-local execution model.

Rules:
- No code may be written before a test strategy is defined.
- devpipe must be used when tasks exist.
- make must be used when suitable targets exist.
- No direct shell commands for testing unless devpipe or make is unavailable.
- All changes require explicit validation commands before completion.

Workflow Rule:
Once your changes are ready, provide a summary including the commit message, files to be staged, and test results.

Commit Patterns
ALWAYS ASK BEFORE COMMITING. NEVER PUSH.

USE CONVENTIONAL COMMITS:
- Follow Conventional Commits specification
- Include subject line (under 72 characters) and body for larger changes
- Format: `type(scope): description`

COMMIT VERIFICATION CHECKLIST:
- All changes staged with `git add .`
- Commit message follows conventional format
- Lines wrapped under 72 characters
- Pre-commit hooks pass (if configured)
- Commit created successfully (`git log --oneline -1`)

PACKAGE MANAGEMENT:
- Respect existing dependency management patterns in the project, using Make targets where available, eg `make build`
- Install packages using `make build` if available

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drewkhoury) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
