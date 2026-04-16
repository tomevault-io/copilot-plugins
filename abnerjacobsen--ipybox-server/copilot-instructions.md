## ipybox-server

> Initial project setup

# Project Setup Guide

Install dependencies and create virtual environment:

```bash
uv sync
```

Activate the virtual environment:

```bash
source .venv/bin/activate
```

Install pre-commit hooks:

```bash
invoke precommit-install
```

Enforce coding conventions (also enforced by pre-commit hooks):

```bash
invoke cc
```

Run tests:

```bash
invoke test
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/abnerjacobsen)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/abnerjacobsen)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
