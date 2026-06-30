## actions-term-on-fail

> For development instructions, make targets, and project structure, see [CONTRIBUTING.md](../../CONTRIBUTING.md).

# Agent Instructions

For development instructions, make targets, and project structure, see [CONTRIBUTING.md](../../CONTRIBUTING.md).

**Important:** Always use `make` targets instead of running commands directly. The Makefile handles environment setup, dependencies, and proper sequencing.

## Quick Reference

| Task | Command |
|------|---------|
| Build everything | `make build` |
| Run dev server | `make run-devmode` |
| Run all linters | `make lint` |
| Run e2e tests | `make smoketest` |

## Dev Mode Authentication

- **Login URL**: `http://localhost:7373/auth/github?user=<username>`
- **OTP Code**: Use `000000` (six zeros) in dev mode

---
> Source: [lawrencegripper/actions-term-on-fail](https://github.com/lawrencegripper/actions-term-on-fail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
