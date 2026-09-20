## heimdall-agent

> TypeScript control plane, Pi solver, observer, operator desk, and workspace playbooks.

# Heimdall

TypeScript control plane, Pi solver, observer, operator desk, and workspace playbooks.

This tree is for **authorized CTF / lab use only**. Do not point it at systems you do not own or have written permission to test.

## Layout

| Path | Role |
|---|---|
| `src/control` | daemon, scheduler, console API, job runner |
| `src/agent` | Pi / Claude SDK solver harness |
| `src/observer` | map / CONFIRMED / LOCK / ANGLES |
| `src/taps` | LLM and HTTP taps |
| `console/` | operator desk (`bin/heimdall console` → `:8700`) |
| `bin/` | thin command entries |
| `resources/` | prompts, workspace templates, shared skills, wordlists, CVEs and toolkit manifests |
| `scripts/` | build, install and runtime scripts |
| `deploy/` | Docker, Compose and container entrypoints |
| `config.example.yaml` | copy to `config.yaml` (gitignored) |
| `tools.env.example.yaml` | copy to `tools.env.yaml` (gitignored) |

## How to run

```bash
npm ci
npm --prefix console ci
cp config.example.yaml config.yaml
cp tools.env.example.yaml tools.env.yaml
# fill LLM keys in tools.env.yaml — never commit them
npm test
npm run build
bin/heimdall console
```

Desk: http://127.0.0.1:8700/  
Quick start: [`README.md`](README.md#quick-start)

## Do not

- Commit keys, `.env`, `tools.env.yaml`, `config.yaml`, ovpn, tokens, or `deploy/**/dist/*.tar.gz`
- Copy `data/`, `tools/c2/`, `deploy/hosted/`, or `deploy/inimage/` into a public tree
- Point the solver LLM URL at the desk (`:8700`) or `host.docker.internal`
- Use this against unauthorized targets

---
> Source: [QiantangCredit/heimdall-agent](https://github.com/QiantangCredit/heimdall-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
