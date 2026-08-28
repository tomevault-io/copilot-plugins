## zoth-studio

> Zoth is a **local-first** stack: public hub (`public/`), private studio deck (`:8484`), encrypted vault (`/vault/` + optional daemon `:8787`).

# AGENTS.md — Zoth Studio

## Product (read this first)

Zoth is a **local-first** stack: public hub (`public/`), private studio deck (`:8484`), encrypted vault (`/vault/` + optional daemon `:8787`).

- Public facts for answer engines: `public/llms.txt`
- How to operate this site: `public/agents.md`
- Human overview: `README.md`

Do not expose `:8484` or `:8787` on the Cloudflare tunnel. Do not put secrets in `public/`.

## How we work (not vibe)

Local models (Ollama `smollm2:360m`, `qwen2.5-coder:1.5b`) **suggest**. Kits, nginx, vault crypto, and bind rules **are the product**. Do not treat generated copy or generated code as finished work.

- Read what you ship. If you cannot explain a bind, a header, or a secret path, do not land it.
- Prefer containment over chrome: loopback studio/vault, static hub, fail-soft probes.
- Pets carry doctrine and briefs; they do not invent physics or skip review.
- No exploit payloads. No “it looks cool so ship it” canvases on operator surfaces.
- Degrees are not the bar. Understanding the system you shipped is.

## For AI agents working in the orchestrator

### Project purpose
This is a local-first, multi-agent AI orchestration framework running on Parrot OS.
It indexes, manages, and chains 47+ tools across security, SEO, media, website gen, and automation.

### Key paths
- `tools/null ai agent tools/local_null_ai_orchestrator/orchestrator.py` — Central CLI (single entrypoint)
- `tools/null ai agent tools/local_null_ai_orchestrator/runtime/` — Python runtime modules
- `tools/null ai agent tools/local_null_ai_orchestrator/registry.local.json` — Tool registry
- `tools/null ai agent tools/local_null_ai_orchestrator/playbooks/` — Agent workflow playbooks

### Rules
- Work in `local_null_ai_*` folders only; never modify original non-prefixed folders.
- Run `orchestrator.py` commands for tool operations (scan, list, show, run, install, session, chain).
- Use `dashboard` or `doctor` to check system health before starting work.
- For Python tools, use `install` command to manage venv/dependencies.
- `run` defaults to dry-run; pass `--confirm` to execute.
- All tool metadata goes in `TOOL.md` and `local_null_ai_manifest.json`.
- Document all commands in the final handoff.

---
> Source: [NullAITech/zoth-studio](https://github.com/NullAITech/zoth-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
