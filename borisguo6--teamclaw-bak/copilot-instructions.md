## teamclaw-bak

> A multi-agent orchestration platform with visual workflow (OASIS). Create and configure agents (OpenClaw/external API), orchestrate them into Teams, build new Teams with Team Creator, and design workflows via visual canvas. Supports Team conversations, OASIS Town with living GraphRAG memory, scheduled tasks, Telegram/QQ bots, TinyFish competitor monitoring, and Cloudflare Tunnel for remote access.


# TeamClaw

Use this skill to install, configure, run, operate, troubleshoot, or modify TeamClaw.

This skill is now the **entrypoint**, not the whole manual. Use **progressive disclosure**:

1. Read [docs/index.md](./docs/index.md) first.
2. Open only the doc(s) needed for the current task.
3. If you need to inspect or edit code, read [docs/repo-index.md](./docs/repo-index.md).
4. Use [README.md](./README.md) for product overview and user-facing positioning, not as the canonical operator reference.

## Task Router

Read only the relevant docs:

| Task | Read First | Then Read |
|---|---|---|
| Install / configure / start TeamClaw | This file | [docs/ports.md](./docs/ports.md) only if ports or routing matter |
| Understand what TeamClaw is | [docs/overview.md](./docs/overview.md) | [README.md](./README.md) |
| Use Team Creator or build a Team from a task description | [docs/team-creator.md](./docs/team-creator.md) | [docs/build_team.md](./docs/build_team.md), [docs/example_team.md](./docs/example_team.md) |
| Understand OASIS runtime semantics, Town Mode, GraphRAG memory, or ReportAgent | [docs/oasis-reference.md](./docs/oasis-reference.md) | [docs/runtime-reference.md](./docs/runtime-reference.md), [docs/create_workflow.md](./docs/create_workflow.md) |
| Understand runtime architecture / auth / services | [docs/runtime-reference.md](./docs/runtime-reference.md) | [docs/ports.md](./docs/ports.md), [docs/repo-index.md](./docs/repo-index.md) |
| Find CLI commands | [docs/cli.md](./docs/cli.md) | `uv run scripts/cli.py <command> --help` |
| Build or edit a Team | [docs/build_team.md](./docs/build_team.md) | [docs/example_team.md](./docs/example_team.md) |
| Create or debug a workflow YAML | [docs/create_workflow.md](./docs/create_workflow.md) | [docs/example_team.md](./docs/example_team.md) |
| Configure OpenClaw integration | [docs/openclaw-commands.md](./docs/openclaw-commands.md) | [docs/build_team.md](./docs/build_team.md) |
| Configure TinyFish competitor monitoring | [docs/tinyfish-monitor.md](./docs/tinyfish-monitor.md) | [docs/runtime-reference.md](./docs/runtime-reference.md), [docs/repo-index.md](./docs/repo-index.md) |
| Debug ports / service wiring / frontend proxy routes | [docs/ports.md](./docs/ports.md) | [docs/repo-index.md](./docs/repo-index.md) |
| Refactor / deploy / rollback | [docs/migration-playbook.md](./docs/migration-playbook.md) | [docs/backend-refactor-plan.md](./docs/backend-refactor-plan.md), [docs/repo-index.md](./docs/repo-index.md) |

## Repository Indexing

Before touching code, read [docs/repo-index.md](./docs/repo-index.md). It maps:

- runtime entrypoints (`src/mainagent.py`, `src/front.py`, `src/time.py`, `oasis/server.py`)
- install and configuration scripts (`selfskill/scripts/run.*`, `configure.py`, `scripts/setup_apikey.*`)
- frontend files (`src/static/js/main.js`, `src/static/css/style.css`, `src/templates/*.html`)
- OASIS Town / GraphRAG files (`oasis/swarm_engine.py`, `oasis/graph_memory.py`, `oasis/forum.py`)
- Team / workflow data paths under `data/user_files/...`
- OpenClaw, bots, tests, and verification commands

If the task is narrow, inspect only the indexed files for that area instead of scanning the whole repo.

## Agent Rules

### Zero-Question Install Flow

The goal is a **silent, non-interactive install**. The AI code CLI agent should NEVER repeatedly ask the user for `LLM_API_KEY`, `LLM_BASE_URL`, or `LLM_MODEL` during installation. Instead:

1. Run `setup` → `configure --init` → `start`. The `start` command:
   - Creates a default `.env` if missing
   - Starts all services even if LLM is not yet configured
   - Automatically warms an installed OpenClaw gateway and refreshes runtime `OPENCLAW_*` values
   - Does NOT silently import OpenClaw/Antigravity config — that's left to the user via the Setup Wizard
   - **Automatically starts Cloudflare Tunnel** for mobile remote access and prints the public URL
2. **Do NOT ask the user for LLM credentials before starting**. The frontend provides a first-login setup wizard where the user can configure API Key, select provider, and choose a model from a dropdown — all from the web UI.
3. After `start`, the console will print the public tunnel URL. **Always tell the user the mobile access URL**:
   - Local: `http://127.0.0.1:<PORT_FRONTEND>/mobile_group_chat`
   - Public (phone): `<PUBLIC_DOMAIN>/mobile_group_chat` — this is the URL to open on a phone browser
   - Tell the user: "在手机浏览器打开以下地址即可远程访问消息中心：`<PUBLIC_DOMAIN>/mobile_group_chat`"
4. The first-login flow handles everything:
   - Passwordless localhost login (no password needed)
   - If LLM is not configured, a setup wizard modal appears automatically
   - The wizard auto-detects local resources and shows import buttons:
     - 🦞 **从 OpenClaw 导入** — if OpenClaw is installed, one click to import API Key/Base URL/Model/Provider
     - ⚡ **使用 Antigravity 免费模型** — if Antigravity-Manager is running on port 8045, one click to configure free model access
   - The wizard also supports: manual Provider dropdown → API Key input → auto-detect models → Model dropdown
   - Password setup is also prompted after first login (non-blocking)

### General Rules

4. Do not install or configure OpenClaw unless the user explicitly asks for it.
5. Cloudflare Tunnel is started **automatically** by the `start` command for mobile remote access. No manual `start-tunnel` is needed. The `stop-tunnel` / `tunnel-status` commands remain available if the user wants to manage it manually.
6. On Windows, prefer the PowerShell flow. Use WSL only if the user prefers it or native Windows tooling is unsuitable.
7. Audio settings should follow the detected LLM provider when left blank:
   - OpenAI: `TTS_MODEL=gpt-4o-mini-tts`, `TTS_VOICE=alloy`, `STT_MODEL=whisper-1`
   - Gemini: `TTS_MODEL=gemini-2.5-flash-preview-tts`, `TTS_VOICE=charon`
8. Never auto-retry a workflow because it looks stuck. Check `topics show` first, report the current status or error, and retry only after user confirmation.
9. Never let a sub-agent start a child workflow unless explicitly instructed.
10. Before adding an OpenClaw agent into a Team, always run `openclaw sessions` and confirm the target agent already exists.
11. On Windows PowerShell, prefer `openclaw.cmd` for channel and plugin commands. `openclaw` may resolve to `openclaw.ps1`, which can fail under restrictive execution policies.
12. For the Weixin plugin on Windows, do not trust the official `npx -y @tencent-weixin/openclaw-weixin-cli@latest install` path as the only route. It may fail because the installer shells out to `which openclaw`. Fall back to manual plugin install with `openclaw.cmd`.
13. If TeamClaw LLM settings change after OpenClaw is already installed, prefer finishing the provider/model selection before pushing TeamClaw config back into OpenClaw. Partial edits intentionally do not auto-sync; use `sync-openclaw-llm` when the desired LLM config is final.

## Standard Install Flow

### Quick Start (Zero Questions)

The simplest install — no questions asked, no manual config required:

```bash
# Linux / macOS
bash selfskill/scripts/run.sh setup
bash selfskill/scripts/run.sh configure --init
bash selfskill/scripts/run.sh start
# → Open http://127.0.0.1:51209
# → First login: use passwordless localhost login
# → Setup wizard appears automatically if LLM not configured
```

```powershell
# Windows PowerShell
powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 setup
powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 configure --init
powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 start
# → Open http://127.0.0.1:51209
```

The `start` command automatically:
1. Creates `config/.env` from template if missing
2. Starts all services regardless of LLM config status
3. Warms an installed OpenClaw gateway and refreshes runtime `OPENCLAW_*` values without importing OpenClaw's LLM config

After startup, the frontend setup wizard handles LLM configuration via the web UI. The wizard detects local OpenClaw and Antigravity-Manager and offers one-click import buttons.

### Magic Prompts for AI Code CLI

After the first login, users can send these prompts to their AI code CLI agent:

- `阅读SKILL帮我安装并配置OpenClaw/AntiGravity`
- `帮我自动选择目前能用的最好的LLM模型`
- `帮OpenClaw安装微信插件并绑定`

### OpenClaw Integration (Optional)

Only install OpenClaw when the user explicitly asks for it.

Check whether OpenClaw exists:

- Linux / macOS: `command -v openclaw >/dev/null 2>&1`
- Windows PowerShell: `Get-Command openclaw -ErrorAction SilentlyContinue`

If the user explicitly wants OpenClaw integration and it is missing, use this flow:

1. Ensure `Node.js >= 22`
2. Install CLI: `npm install -g openclaw@latest --ignore-scripts`
3. Run onboarding:
   - Windows / automation: `openclaw onboard --non-interactive --accept-risk --install-daemon`
   - If you want OpenClaw to reuse an existing OpenAI key: append `--openai-api-key <LLM_API_KEY>`
   - Linux / macOS local interactive flow: `openclaw onboard --install-daemon`
4. Enable HTTP compatibility: `openclaw config set gateway.http.endpoints.chatCompletions.enabled true`
5. Restart gateway: `openclaw gateway restart`
6. Web entry points after the gateway is up:
   - OpenClaw dashboard / Control UI: `http://127.0.0.1:18789/`
   - OpenClaw OpenAI-compatible HTTP API: `http://127.0.0.1:18789/v1/chat/completions`
7. Sync TeamClaw integration:
   - Linux / macOS: `bash selfskill/scripts/run.sh check-openclaw`
   - Windows: `powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 check-openclaw`
8. If the OpenClaw dashboard shows `gateway token missing`, either:
   - paste `OPENCLAW_GATEWAY_TOKEN` into Control UI settings, or
   - for loopback-only local development, switch to no-auth:
     - `openclaw config set gateway.auth.mode none`
     - `openclaw config unset gateway.auth.token`
     - `openclaw gateway restart`
10. If TeamClaw was already running before OpenClaw was installed or reconfigured, restart TeamClaw so OASIS reloads the `openclaw` CLI.

OpenClaw troubleshooting notes:

- On Windows PowerShell, prefer `openclaw.cmd` if `openclaw.ps1` is blocked by execution policy.
- Re-running `openclaw onboard` usually keeps channel bindings, but it may rewrite gateway auth mode or the default model/provider. Re-check:
  - `openclaw.cmd gateway status`
  - `openclaw.cmd models status --json`
  - `uv run scripts/cli.py openclaw bindings --agent main`
- Switching from OpenAI to DeepSeek is not "replace the key only". Update TeamClaw `LLM_API_KEY`, `LLM_BASE_URL`, `LLM_MODEL`, and `LLM_PROVIDER` together. For OpenClaw, define a DeepSeek custom provider in `~/.openclaw/openclaw.json` under `models.providers.deepseek` (using `"api": "openai-completions"` and `"baseUrl": "https://api.deepseek.com/v1"`) and set the default model explicitly. See [docs/openclaw-commands.md § 4](./docs/openclaw-commands.md) for the full JSON snippet and gotchas. A tested stable pair is:
  - TeamClaw: `LLM_BASE_URL=https://api.deepseek.com`, `LLM_MODEL=deepseek-chat`, `LLM_PROVIDER=deepseek`
  - OpenClaw: provider `deepseek`, model `deepseek/deepseek-chat`
- **Switching to Antigravity-Manager** (local reverse proxy that gives free access to 67+ models — Claude/Gemini/GPT — for users with a Google One Pro membership, e.g. via student verification). Antigravity exposes an OpenAI-compatible API at `http://127.0.0.1:8045`. See [docs/openclaw-commands.md § 4.1](./docs/openclaw-commands.md) for the full JSON snippets. A tested stable pair is:
  - TeamClaw: `LLM_BASE_URL=http://127.0.0.1:8045`, `LLM_API_KEY=sk-antigravity`, `LLM_MODEL=gemini-3.1-pro`, `LLM_PROVIDER=antigravity`
  - OpenClaw: provider `antigravity`, model `antigravity/gemini-3.1-pro`
  - **Prerequisite**: Antigravity-Manager must be running on port 8045 before starting TeamClaw. Install:
    - Linux / macOS: `curl -fsSL https://raw.githubusercontent.com/lbjlaq/Antigravity-Manager/v4.1.31/install.sh | bash`
    - macOS launch: `open -a "Antigravity Tools"`
  - Audio settings: Antigravity-forwarded Gemini models follow Gemini audio defaults automatically; Claude models do not have built-in TTS.
  - If OpenClaw HTTP API returns 401 after switching provider, check `openclaw config get gateway.auth`. If mode is `token`, either pass the token or switch to `none` for local-only use: `openclaw config set gateway.auth.mode none && openclaw config unset gateway.auth.token && openclaw gateway restart`.
- **Switching to MiniMax** (OpenAI-compatible API at `https://api.minimaxi.com`). Models: `MiniMax-M2.7` (1M context, reasoning), `MiniMax-M2.7-highspeed`. See [docs/openclaw-commands.md § 4.2](./docs/openclaw-commands.md) for the full JSON snippets. A tested stable pair is:
  - TeamClaw: `LLM_BASE_URL=https://api.minimaxi.com`, `LLM_API_KEY=<minimax-key>`, `LLM_MODEL=MiniMax-M2.7`, `LLM_PROVIDER=minimax`
  - OpenClaw: provider `minimax`, model `minimax/MiniMax-M2.7`
  - API key format: `sk-api-...`, obtain from [MiniMax platform](https://platform.minimaxi.com/).
  - `LLM_PROVIDER=minimax` in TeamClaw maps to `openai` compatible format (via `llm_factory.py` `_PROVIDER_ALIASES`).
- `openclaw gateway status` can still print an RPC probe warning even when the dashboard and HTTP API are healthy. Confirm with:
  - `http://127.0.0.1:18789/`
  - `http://127.0.0.1:18789/v1/chat/completions`
  - `openclaw.cmd channels list --json`
- TeamClaw's `http://127.0.0.1:<PORT_AGENT>/v1/chat/completions` is authenticated. If a direct HTTP call returns `认证失败`, verify the stack with `run.ps1 status` or test the configured LLM via `uv run python -` and `src/llm_factory.py`.

### OpenClaw Weixin Channel

Use this only when the user explicitly wants Weixin / 微信 integration.

Windows-specific notes:

1. In PowerShell, use `openclaw.cmd`, not bare `openclaw`, if script execution policy blocks `openclaw.ps1`.
2. The official installer may fail on Windows PowerShell:
   - `npx -y @tencent-weixin/openclaw-weixin-cli@latest install`
   - Reason: it shells out to `which openclaw`
3. If that happens, use the manual Windows flow:

```powershell
openclaw.cmd plugins install "@tencent-weixin/openclaw-weixin"
openclaw.cmd config set plugins.entries.openclaw-weixin.enabled true
openclaw.cmd channels login --channel openclaw-weixin
openclaw.cmd channels list --json
openclaw.cmd gateway restart
```

4. `openclaw status` showing `openclaw-weixin | ON | SETUP | no token` means:
   - the plugin is installed
   - login has not completed yet
5. After QR login succeeds, bind the Weixin account to an OpenClaw agent:

```powershell
powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 bind-openclaw-channel main openclaw-weixin:<account_id>
```

Or via TeamClaw CLI:

```powershell
uv run scripts/cli.py openclaw channels
uv run scripts/cli.py openclaw bind --data '{"agent":"main","channel":"openclaw-weixin:<account_id>"}'
```

6. After binding, refresh the TeamClaw OpenClaw Channels tab or re-run:
   - `uv run scripts/cli.py openclaw bindings --agent main`

### Advanced: Manual CLI Configuration

For users who prefer CLI over the web UI, or for automation scripts:

```bash
# Linux / macOS — manually set LLM config
bash selfskill/scripts/run.sh configure LLM_API_KEY sk-xxx
bash selfskill/scripts/run.sh configure LLM_BASE_URL https://api.example.com
bash selfskill/scripts/run.sh auto-model
# Print the model list only, choose one explicitly, then:
bash selfskill/scripts/run.sh configure LLM_MODEL <model>
```

```powershell
# Windows PowerShell — manually set LLM config
powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 configure LLM_API_KEY sk-xxx
powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 configure LLM_BASE_URL https://api.example.com
powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 auto-model
# Print the model list only, choose one explicitly, then:
powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 configure LLM_MODEL <model>
```

If OpenClaw is installed on the same machine, TeamClaw now supports an explicit reverse sync back into OpenClaw:

```bash
bash selfskill/scripts/run.sh sync-openclaw-llm
```

```powershell
powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 sync-openclaw-llm
```

`configure` also auto-syncs safe LLM updates when the TeamClaw config is complete. Partial edits that only touch `LLM_API_KEY` or `LLM_BASE_URL` intentionally stop short of rewriting OpenClaw, so a half-finished provider switch does not leak into the OpenClaw default model config.

For managed terminals, CI, or agent runners that clean up child processes after the command exits, use `start-foreground` instead of `start`.

### Windows WSL Fallback

Use WSL only when the user wants it or native PowerShell is not suitable.

- Install WSL in an elevated PowerShell window: `wsl --install -d Ubuntu`
- Prefer a Linux-side copy of the repo instead of running directly from `/mnt/c/...`
- Keep WSL and native Windows installs on separate copies and separate ports

## Recommended Configuration

These keys are recommended but **not required before first start**. The frontend setup wizard will guide the user through configuration on first login:

| Key | Purpose |
|---|---|
| `LLM_API_KEY` | Provider API key |
| `LLM_BASE_URL` | OpenAI-compatible base URL |
| `LLM_MODEL` | Model name chosen explicitly or after `auto-model` |

If these are left blank, TeamClaw starts normally but LLM-dependent features (chat, OASIS discussions) will not work until configured via the web UI setup wizard.

The template intentionally leaves `LLM_MODEL` empty so a DeepSeek default does not silently leak into OpenAI / Gemini installs.

## Optional Audio Configuration

Leave audio keys blank unless the user needs overrides:

| Key | Purpose |
|---|---|
| `TTS_MODEL` | Text-to-speech model |
| `TTS_VOICE` | Voice preset |
| `STT_MODEL` | Speech-to-text model |

Blank values should follow the current LLM provider automatically.

## Startup Expectations

After `start`, these services should come up:

| Service | Port variable | Default |
|---|---|---|
| Agent | `PORT_AGENT` | `51200` |
| Scheduler | `PORT_SCHEDULER` | `51201` |
| OASIS | `PORT_OASIS` | `51202` |
| Frontend | `PORT_FRONTEND` | `51209` |

Useful checks:

- `bash selfskill/scripts/run.sh status`
- `powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 status`
- `GET http://127.0.0.1:<PORT_AGENT>/v1/models`
- open `http://127.0.0.1:<PORT_FRONTEND>`

Notes:

- On Windows, the default ports may be auto-remapped to safe values; always trust `config/.env` or `status`.
- Use `http://127.0.0.1:<PORT_FRONTEND>`, not a hardcoded frontend port.
- Local `127.0.0.1` access supports passwordless login; non-localhost access should use password login.
- TeamClaw starts even without LLM configured. The frontend setup wizard will prompt the user to configure on first login.
- `chatbot/setup.py` requires an interactive terminal (`stdin.isatty()`). When launched from a non-interactive context (agent runners, CI, background processes, or `scripts/start.sh` piped from another script), `launcher.py` automatically skips the chatbot interactive menu. You can also force this by setting `TEAMBOT_HEADLESS=1`. If you still see `EOFError: EOF when reading a line`, it means the `isatty` guard was bypassed — ensure you are using `selfskill/scripts/run.sh start` (which backgrounds `launcher.py` correctly) instead of calling `scripts/start.sh` or `scripts/launcher.py` directly in a non-interactive shell.

### Troubleshooting: Python 2 vs Python 3

On macOS, the system `python` command may point to **Python 2.7** (`/usr/local/bin/python` or `/usr/bin/python`), while TeamClaw requires **Python 3.11+**. This causes confusing errors if you try to run scripts directly:

```
SyntaxError: Non-ASCII character '\xe5' in file front.py on line 36
```

**Root cause**: Python 2 cannot handle UTF-8 Chinese characters without a `# coding` header, and TeamClaw uses Python 3.11+ syntax throughout.

**Solutions** (in order of preference):

1. **Always use the canonical startup** — this handles venv creation and activation automatically:
   ```bash
   bash selfskill/scripts/run.sh start
   ```

2. **Activate the venv first**, then use bare `python`:
   ```bash
   source .venv/bin/activate
   python scripts/launcher.py
   ```

3. **Use the venv python directly**:
   ```bash
   .venv/bin/python scripts/launcher.py
   ```

**Never do this**:
```bash
# ❌ Wrong — system python may be Python 2.7
cd src && python front.py

# ❌ Wrong — python3 may be a different version (e.g., 3.14) without project deps
python3 src/front.py
```

**Safety guards**: `launcher.py` now includes a Python version check at startup and will print a clear error if accidentally run under Python 2. The `run.sh` and `manual_run.sh` scripts also verify the Python version after venv activation.

## Common Operations

### Runtime

```bash
bash selfskill/scripts/run.sh status
bash selfskill/scripts/run.sh stop
bash selfskill/scripts/run.sh configure --show
```

```powershell
powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 status
powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 stop
powershell -ExecutionPolicy Bypass -File selfskill/scripts/run.ps1 configure --show
```

### CLI

Use CLI help first:

```bash
uv run scripts/cli.py --help
uv run scripts/cli.py teams --help
uv run scripts/cli.py workflows --help
uv run scripts/cli.py openclaw --help
```

### Workflow Monitoring

Prefer non-blocking checks:

```bash
uv run scripts/cli.py topics show --topic-id <ID>
```

Avoid using `topics watch` or `workflows conclusion` when you need a quick status snapshot.

## Team Data Layout

Team-specific data lives under:

```text
data/user_files/{user_id}/teams/{team_name}/
```

See [docs/repo-index.md](./docs/repo-index.md) and [docs/example_team.md](./docs/example_team.md) for the file-level breakdown.

## Reference Docs

- [docs/index.md](./docs/index.md) — canonical docs map
- [docs/repo-index.md](./docs/repo-index.md) — codebase and file index
- [docs/overview.md](./docs/overview.md) — product overview
- [docs/team-creator.md](./docs/team-creator.md) — Team Creator flow, jobs, bilingual UI, workflow-to-team bridge
- [docs/oasis-reference.md](./docs/oasis-reference.md) — OASIS runtime model and orchestration reference
- [docs/runtime-reference.md](./docs/runtime-reference.md) — architecture, services, auth, runtime reference
- [docs/cli.md](./docs/cli.md) — CLI reference
- [docs/build_team.md](./docs/build_team.md) — Team creation and member config
- [docs/create_workflow.md](./docs/create_workflow.md) — workflow YAML format
- [docs/example_team.md](./docs/example_team.md) — example Team files
- [docs/openclaw-commands.md](./docs/openclaw-commands.md) — OpenClaw commands and patterns
- [docs/tinyfish-monitor.md](./docs/tinyfish-monitor.md) — TinyFish monitor operations and persistence model
- [docs/ports.md](./docs/ports.md) — service map and ports
- [docs/migration-playbook.md](./docs/migration-playbook.md) — deployment / rollback
- [docs/backend-refactor-plan.md](./docs/backend-refactor-plan.md) — architecture roadmap

---
> Source: [BorisGuo6/TeamClaw-bak](https://github.com/BorisGuo6/TeamClaw-bak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
