## adforgeai

> AI ad-agency production line: **n8n** orchestrates the workflow; a **model-router**

# CLAUDE.md — AdForgeAI

AI ad-agency production line: **n8n** orchestrates the workflow; a **model-router**
is the single egress to four vLLM model services (running locally on this host's
4× A100 80GB GPUs — see `docs/models.md`). See
`README.md` (humans) and `docs/` (agents) for detail. `CONTRACTS.md` is the
canonical API/data seam — read it before touching any service boundary.

## ⚠️ Keep the docs current — every edit
When you change anything that affects another part of the system, update the
docs **in the same change** (not "later"):

- New/changed router or ffmpeg endpoint, field, or status value → update `CONTRACTS.md` **and** `docs/services.md`.
- New env var, port, or service → `docker-compose.yml`, `.env.example`, `docs/services.md`, `docs/architecture.md`.
- Model behavior/quirk discovered or model swapped → `docs/models.md`.
- A new hard-won gotcha (the kind that cost you a debug cycle) → `docs/gotchas.md`. This file is the highest-value doc; add to it liberally.
- New/changed n8n workflow → `docs/workflows.md` and `n8n/workflows/README.md`.

A change that leaves the docs stale is incomplete. If you're unsure which doc,
default to `docs/gotchas.md` + `CONTRACTS.md`.

## ⚠️ Commit & push after every edit
After any change, commit and push to `main` — don't leave work uncommitted:
```bash
git add -A && git commit -m "<concise message>" && git push
```
- `origin main` is already tracked; `git push` suffices.
- Update docs in the **same commit** as the code (see the rule above).
- End commit messages with `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.
- Never commit `.env` or other secrets (already gitignored — keep it that way).

## How this fits together (30-second version)
```
browser → gateway (nginx, :9000 — ONLY published port) ─┬─ / ............ frontend (SPA)
                                                         ├─ /api/router/ . model-router → {qwen36, qwen-image, cosmos3-nano, acestep}
                                                         ├─ /api/ffmpeg/ . ffmpeg-worker (assembly)
                                                         ├─ /api/n8n/ , /n8n/ . n8n (webhooks + editor)
                                                         └─ /adgen/ , /console/ . MinIO (assets + console)
model-router ─┬─ Postgres (project state)   model-bridge (socat relay → local model ports on 4× A100)
              └─ MinIO (all binaries)
```
The `gateway` is the single ingress: it is the only service that publishes a host
port. Only `model-router` reaches the model ports (via `host.docker.internal` →
the `model-bridge` relay). n8n, ffmpeg-worker, and the browser never touch them.

## Conventions (follow these)
- **Contract-first**: change `CONTRACTS.md` before changing a boundary; build consumers against it.
- **All model calls go through the router.** Don't add direct model-service calls elsewhere.
- **n8n moves IDs/JSON/URLs, never binaries.** Binaries live in MinIO; pass URIs.
- **Public vs internal URLs**: stored asset URLs use `localhost:9000` (browser). Inside containers you MUST rewrite to `minio:9000` — use the `internal_url()` helper (router + ffmpeg both have it). This bites every time; see `docs/gotchas.md`.
- **Lazy/minimal** (this repo is built ponytail-style): stdlib over deps, shortest working change, mark deliberate shortcuts with `# ponytail:` comments naming the upgrade path.

## Run / rebuild / test
**Docker needs `sudo` here, but `sudo` is passwordless** — prefix every docker/compose
command with `sudo` (e.g. `sudo docker compose ...`). No password prompt; safe to run non-interactively.
```bash
sudo docker compose up -d --build              # bring up the stack
sudo docker compose build <svc> && sudo docker compose up -d --force-recreate <svc>   # after editing a service
curl -s localhost:9000/api/router/healthz      # router + model reachability (via gateway)
python3 full_test.py                           # full pipeline smoke test (slow: i2v ~7min)
sudo docker compose exec model-router python probe_services.py   # rediscover model APIs after a model swap
```
After editing an n8n workflow JSON, re-import + restart (import alone doesn't reload a running n8n):
```bash
sudo docker compose run --rm n8n-init && sudo docker compose restart n8n   # re-import all + re-activate the chain
```
More in `docs/development.md`.

## Docs map
- `docs/architecture.md` — services, data flow, network topology
- `docs/services.md` — each service: ports, env, endpoints, key files
- `docs/models.md` — the 4 models, IDs, context, API shapes, qwen36 JSON quirks
- `docs/workflows.md` — n8n workflows + the agent chain (research → strategy → **casting** → storyboard → cinematography → **sound** → image/video/post → QA), webhooks, auto-record
- `docs/gotchas.md` — the non-obvious failures and their fixes (read this!)
- `docs/development.md` — run/rebuild/test/extend playbook

---
> Source: [avinash-mall/AdForgeAI](https://github.com/avinash-mall/AdForgeAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
