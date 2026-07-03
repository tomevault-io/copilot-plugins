## comfyui-custom-node-template

> Single publishable ComfyUI custom node pack.

# AGENTS.md

Single publishable ComfyUI custom node pack.

- Frontend runtime code lives in `frontend/`
- Backend node code lives in `backend/`
- Root `__init__.py` is the thin ComfyUI entry shim
- Use repo commands first: `pnpm typecheck`, `pnpm test`, `pnpm test:unit`, `pnpm test:e2e`
- Use `uv` for Python dependency sync and Python execution outside repo scripts

For testing details, see `docs/TESTING.md`.
For ComfyUI API changes, verify current official docs before changing architecture or advanced frontend hooks.

---
> Source: [PBandDev/comfyui-custom-node-template](https://github.com/PBandDev/comfyui-custom-node-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
