## virtual-env

> Always use an isolated project environment, never the system runtime


# Isolated environments

Never install packages or run app tooling with the machine’s default Python or Node.

- **Python:** create `.venv` if missing (`python3 -m venv .venv`) and use `.venv/bin/python` / `.venv/bin/pip`. Never system `pip`.
- **Node (this repo):** Node 20 per `.nvmrc` / Dockerfile. Use nvm/fnm/volta, Docker `node:20-bookworm-slim`, or a project-local `.node/` (gitignored). Do not use global Node (e.g. Homebrew v25) to change `package-lock.json`.
- Prefer `npm ci` after lockfile changes. Do not `npm install -g` unless the user asked.

---
> Source: [Gourav-praneeth/GradeLens](https://github.com/Gourav-praneeth/GradeLens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
