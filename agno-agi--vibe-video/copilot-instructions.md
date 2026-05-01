## vibe-video

> A conversational multi-agent team that produces Hyperframes motion-graphics

# Vibe Video

A conversational multi-agent team that produces Hyperframes motion-graphics
videos from natural-language prompts. Part of the "mini demo agent" family
alongside [vibe-to-prd](https://github.com/agno-agi/vibe-to-prd).

The canonical specification is `docs/SPEC.md`. All other documentation
derives from it — keep the two in sync.

## Architecture

- Team definition: `vibe_video/team.py` (Vibe Video team leader, Coordinate mode)
- Member agents: `vibe_video/agents/animator.py` (Animator),
  `vibe_video/agents/code_explorer.py` (CodeExplorer),
  `vibe_video/agents/researcher.py` (Researcher, optional)
- Shared settings: `vibe_video/settings.py` (DB, REPOS_DIR, RENDERS_DIR, model)
- API server: `app/main.py` (FastAPI + AgentOS web UI)
- Custom tools: `vibe_video/tools/git.py` (GitTools),
  `vibe_video/tools/hyperframes.py` (HyperframesTools)
- Hyperframes authoring guide: compact, hand-written, inlined directly
  in `vibe_video/agents/animator.py` (no external skill assets)
- Database: PostgreSQL + pgvector (session memory + agentic memory)

## Team Structure
```
Vibe Video (Team, Coordinate, claude-opus-4-7)
├── Animator   — writes Hyperframes HTML, renders to MP4, iterates
├── CodeExplorer   — clones arbitrary repos on demand, read-only
├── Researcher — web search via Parallel (optional; gated on PARALLEL_API_KEY)
└── [leader responds directly for greetings/simple questions]
```

- **Leader** — routes, delegates, synthesizes. No tools of its own.
- **Animator** — HyperframesTools, FileTools, ReasoningTools.
- **CodeExplorer** — CodingTools (read-only), GitTools (clone + read), ReasoningTools.
- **Researcher** — ParallelTools, ReasoningTools. Instantiated only when
  `PARALLEL_API_KEY` is set.

## Key Concepts

- **Coordinate mode:** leader picks the right specialist, delegates a
  structured brief, synthesizes. Respond directly only for greetings
  and "what can you do?".
- **Hyperframes:** HTML + GSAP local-render framework from HeyGen. The
  Animator writes `index.html` + optional `compositions/*.html` into a
  per-session workspace under `RENDERS_DIR`, lints, renders to MP4 via
  the CLI, and iterates on failure (up to `MAX_RENDER_RETRIES`).
- **On-demand cloning:** CodeExplorer has no pre-configured repo list.
  `REPOS_DIR` is an ephemeral scratch dir; `clone_repo(url)` is
  idempotent (clones or fast-forwards).
- **Skill-in-prompt:** the Hyperframes and GSAP skills from
  `heygen-com/hyperframes` are fetched at build time and inlined into
  Animator's system prompt.
- **Agentic memory:** captures user style preferences (e.g. "3Blue1Brown
  pacing", narration on/off) across sessions. No learnings KB.
- **Scheduler:** `scheduler=True` in AgentOS. No pre-registered jobs —
  the feature is surfaced so users can schedule their own recurring
  runs against the team from the web UI.

## Structure
```
vibe-video/
├── app/
│   ├── main.py          # AgentOS web UI
│   └── config.yaml      # Quick prompts
├── vibe_video/
│   ├── team.py          # Vibe Video team (leader)
│   ├── agents/
│   │   ├── animator.py
│   │   ├── code_explorer.py
│   │   └── researcher.py
│   ├── settings.py      # Shared DB/paths/model
│   ├── tools/
│   │   ├── git.py          # Read-only git + clone_repo
│   │   └── hyperframes.py  # Hyperframes CLI wrapper
├── db/
│   ├── session.py
│   └── url.py
├── scripts/                # validate.sh, format.sh, entrypoint.sh, ...
├── docs/
│   └── SPEC.md          # Canonical specification
├── compose.yaml
├── Dockerfile
├── pyproject.toml
└── requirements.txt
```

## Running
```bash
docker compose up -d --build
```

Connect via the AgentOS web UI at `http://localhost:8000` or the CLI
(`python -m vibe_video`).

## Setup Flow

1. Clone repo.
2. Configure `.env`:
   - `ANTHROPIC_API_KEY` (required)
   - `PARALLEL_API_KEY` (optional; enables Researcher)
   - `GITHUB_ACCESS_TOKEN` (optional; enables private repo clone)
3. `docker compose up -d --build` (builds the image — first build takes
   a few minutes because it installs Node + Chromium + Hyperframes).
4. Open the AgentOS web UI, or run `python -m vibe_video` in a container
   shell.

## Local Development
```bash
./scripts/venv_setup.sh && source .venv/bin/activate
docker compose up -d vibe-video-db
python -m vibe_video              # CLI mode
```

Local rendering requires Node 22, FFmpeg, and a Chromium binary on PATH.
On macOS: `brew install node ffmpeg chromium`.

## Commands
```bash
./scripts/venv_setup.sh && source .venv/bin/activate
./scripts/format.sh                 # Format code
./scripts/validate.sh               # Lint + type check
python -m vibe_video                # CLI mode
```

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `ANTHROPIC_API_KEY` | Yes | Anthropic API key (leader + all agents) |
| `DB_*` | No | Postgres config (defaults to localhost) |
| `PARALLEL_API_KEY` | No | Enables Researcher — web search via Parallel |
| `GITHUB_ACCESS_TOKEN` | No | Enables CodeExplorer to clone private repos |
| `REPOS_DIR` | No | Ephemeral clone scratch (default: `/repos` named volume in Docker; `~/.cache/vibe-video/repos` locally) |
| `RENDERS_DIR` | No | Render workspace + MP4 outputs (default: `./renders` — bind-mounted in Docker, same path locally; gitignored) |
| `MAX_RENDER_RETRIES` | No | Animator's render retry budget (default: 3) |
| `HYPERFRAMES_BIN` | No | Override for `npx hyperframes` |
| `JWT_VERIFICATION_KEY` | Prod | RBAC public key from os.agno.com |

---
> Source: [agno-agi/vibe-video](https://github.com/agno-agi/vibe-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
