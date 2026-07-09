## doca

> OpenClaw Dashboard — a single Node.js/Express web app (`server.js` + `modules/*`) that serves a static frontend (`public/`) and a JSON/SSE API for managing an external "OpenClaw" Docker/AI stack. There is one service; no database.

# AGENTS.md

## Cursor Cloud specific instructions

### What this is
OpenClaw Dashboard — a single Node.js/Express web app (`server.js` + `modules/*`) that serves a static frontend (`public/`) and a JSON/SSE API for managing an external "OpenClaw" Docker/AI stack. There is one service; no database.

### Running
- Dev: `npm run dev` (uses `node --watch server.js` for hot reload). Prod-style: `npm start`.
- Listens on `0.0.0.0:4242` (override with `PORT`).
- The server serves **HTTPS with a self-signed cert** (auto-generated into `.certs/`), falling back to HTTP only if cert generation fails. Use `curl -k` and, in a browser, click through the "Your connection is not private" warning (Advanced → Proceed).

### Lint / test / build
- There is **no build step** (plain JS, static assets) and **no lint or test scripts** — `package.json` only defines `start` and `dev`. Do not expect `npm test`/`npm run lint` to exist.

### Environment gotchas (not bugs)
- The dashboard manages an *external* Docker Compose stack and various AI CLIs. Those tools (Docker, Ollama, nvidia-smi, huggingface-cli, etc.) are **not installed by default**. Panels that shell out to them (e.g. "All Containers" showing `docker: not found`, GPU stats, model managers) will show errors/empty state. This is expected and does not indicate the app is broken — installing Docker/etc. is optional and only needed to exercise those specific panels.
- Some sidebar stats (CPU temp, GPU) read host sensors that are unavailable in the VM and render as `-`.
- Default paths (`COMPOSE_DIR`, `CONFIG_PATH`, `SKILLS_DIR`, `WORKSPACE_DIR`, `SNAPSHOT_DIR`) derive from `~` and may not exist; override via env vars (see README "Environment Variables") when testing those features.
- Runtime prefs are written to `.dashboard-prefs.json` and certs to `.certs/` (both gitignored).

---
> Source: [Zalban95/DOCA](https://github.com/Zalban95/DOCA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
