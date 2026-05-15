## vibe-to-prd

> Drop a product idea, get a PRD. AI agent that turns vibes into specs.

# Vibe-to-PRD

Drop a product idea, get a PRD. AI agent that turns vibes into specs.

## Structure

```
vibe-to-prd/
├── app/
│   ├── main.py              # AgentOS entry point (tracing + scheduler enabled)
│   └── config.yaml          # Quick prompts for web UI
├── architect/
│   ├── agent.py             # Architect agent (gpt-5.4, UserFeedbackTools + FileTools)
│   ├── instructions.py      # Prompt (phases, PRD format, memory, rules)
│   └── __main__.py          # python -m architect
├── db/
│   └── __init__.py          # PostgreSQL connection (singleton)
├── prds/                    # Generated PRDs saved here (gitignored)
├── scripts/
│   ├── format.sh            # ruff format
│   ├── validate.sh          # ruff lint + mypy
│   ├── venv_setup.sh        # venv setup
│   ├── generate_requirements.sh  # uv pip compile
│   ├── railway_up.sh        # First-time Railway provisioning
│   ├── railway_redeploy.sh  # Redeploy to Railway
│   └── railway_env.sh       # Sync .env.production to Railway
├── compose.yaml
├── Dockerfile
└── pyproject.toml
```

## Agent: Architect

- **Tools:** UserFeedbackTools (structured questions), FileTools (save PRDs to `prds/`)
- **Memory:** Agentic memory enabled — learns user preferences across sessions (platform, audience, product style)
- **Flow:** Seed (user drops idea) → Discovery (4-6 questions via ask_user) → Generate (full PRD + save to file)
- **PRD sections:** Overview, Target User, Core Features, User Journey, Technical Considerations, Scope & Timeline, Success Metrics, Monetization, Open Questions

## Running

```bash
docker compose up -d --build
# Web UI: os.agno.com → Add OS → Local → http://localhost:8000
# CLI:    python -m architect
```

## Local Dev

```bash
./scripts/venv_setup.sh && source .venv/bin/activate
docker compose up -d db
python -m architect
```

## Commands

```bash
./scripts/format.sh                    # Format
./scripts/validate.sh                  # Lint + type check
./scripts/generate_requirements.sh     # Regenerate requirements.txt
./scripts/railway_up.sh               # First-time Railway deploy
./scripts/railway_redeploy.sh         # Redeploy to Railway
./scripts/railway_env.sh              # Sync .env.production to Railway
```

---
> Source: [agno-agi/vibe-to-prd](https://github.com/agno-agi/vibe-to-prd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
