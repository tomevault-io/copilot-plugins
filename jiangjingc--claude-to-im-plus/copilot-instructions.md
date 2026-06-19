## claude-to-im-plus

> |


# Claude-to-IM Plus Bridge Skill

You are managing the Claude-to-IM Plus bridge.
User data is stored at `~/.claude-to-im/`.

The skill directory (SKILL_DIR) is at `~/.claude/skills/claude-to-im`.
In Codex installs it may instead be `~/.codex/skills/claude-to-im`.
If neither path exists, fall back to Glob with pattern `**/skills/**/claude-to-im/SKILL.md` or `**/skills/**/claude-to-im-plus/SKILL.md` and derive the root from the result.

## Command parsing

Parse the user's intent from `$ARGUMENTS` into one of these subcommands:

| User says (examples) | Subcommand |
|---|---|
| `setup`, `configure`, `配置`, `我想在飞书上用 Claude`, `帮我连接 Telegram`, `帮我接微信`, `帮我接钉钉` | setup |
| `start`, `start bridge`, `启动`, `启动桥接` | start |
| `stop`, `stop bridge`, `停止`, `停止桥接` | stop |
| `status`, `bridge status`, `状态`, `运行状态`, `怎么看桥接的运行状态` | status |
| `logs`, `logs 200`, `查看日志`, `查看日志 200` | logs |
| `handoff weixin`, `handoff dingtalk`, `把当前会话切到微信`, `把当前会话切到钉钉`, `把当前 Codex 会话切到微信`, `把当前 Claude 会话切到钉钉` | handoff |
| `reconfigure`, `修改配置`, `帮我改一下 token`, `换个 bot` | reconfigure |
| `doctor`, `diagnose`, `诊断`, `挂了`, `没反应了`, `bot 没反应`, `出问题了` | doctor |

**Disambiguation: `status` vs `doctor`** — Use `status` when the user just wants to check if the bridge is running (informational). Use `doctor` when the user reports a problem or suspects something is broken (diagnostic). When in doubt and the user describes a symptom (e.g., "没反应了", "挂了"), prefer `doctor`.

Extract optional numeric argument for `logs` (default 50).

Before asking users for any platform credentials, read `SKILL_DIR/references/setup-guides.md` internally so you know where to find each credential. Do NOT dump the full guide to the user upfront — only mention the specific next step they need to do (e.g., "Go to https://open.feishu.cn → your app → Credentials to find the App ID"). If the user says they don't know how, then show the relevant section of the guide.

## Runtime detection

Before executing any subcommand, detect which environment you are running in:

1. **Claude Code** — `AskUserQuestion` tool is available. Use it for interactive setup wizards.
2. **Codex / other** — `AskUserQuestion` is NOT available. Fall back to non-interactive guidance: explain the steps, show `SKILL_DIR/config.env.example`, and ask the user to create `~/.claude-to-im/config.env` manually.

You can test this by checking if AskUserQuestion is in your available tools list.

## Config check (applies to `start`, `stop`, `status`, `logs`, `reconfigure`, `doctor`)

Before running any subcommand other than `setup`, check if `~/.claude-to-im/config.env` exists:

- **If it does NOT exist:**
  - In Claude Code: tell the user "No configuration found" and automatically start the `setup` wizard using AskUserQuestion.
  - In Codex: tell the user "No configuration found. Please create `~/.claude-to-im/config.env` based on the example:" then show the contents of `SKILL_DIR/config.env.example` and stop. Don't attempt to start the daemon — without config.env the process will crash on startup and leave behind a stale PID file that blocks future starts.
- **If it exists:** proceed with the requested subcommand.

## Subcommands

### `setup`

Run an interactive setup wizard. This subcommand requires `AskUserQuestion`. If it is not available (Codex environment), instead show the contents of `SKILL_DIR/config.env.example` with field-by-field explanations and instruct the user to create the config file manually.

When AskUserQuestion IS available, collect input **one field at a time**. After each answer, confirm the value back to the user (masking secrets to last 4 chars only) before moving to the next question.

**Step 1 — Choose channels**

Ask which channels to enable (telegram, discord, feishu, qq, weixin, dingtalk). Accept comma-separated input. Briefly describe each:
- **telegram** — Best for personal use. Streaming preview, inline permission buttons.
- **discord** — Good for team use. Server/channel/user-level access control.
- **feishu** (Lark) — For Feishu/Lark teams. Streaming cards, tool progress, inline permission buttons.
- **qq** — QQ C2C private chat only. No inline permission buttons, no streaming preview. Permissions use text `/perm ...` commands.
- **weixin** — WeChat QR login. Single linked account only; a new login replaces the previous one. No inline permission buttons, no streaming preview. Permissions use text `/perm ...` commands or quick `1/2/3` replies. Voice messages only use WeChat's own speech-to-text text; raw voice audio is not transcribed by the bridge.
- **dingtalk** — DingTalk Stream mode. Supports private chats and group chats; v1 only handles `@bot` or reply-to-bot messages in groups. Plain text replies only, but inbound images can be forwarded to Claude/Codex.

**Step 2 — Collect tokens per channel**

For each enabled channel, collect one credential at a time. Tell the user where to find each value in one sentence. Only show the full guide section (from `SKILL_DIR/references/setup-guides.md`) if the user asks for help or says they don't know how:

- **Telegram**: Bot Token → confirm (masked) → Chat ID (see guide for how to get it) → confirm → Allowed User IDs (optional). **Important:** At least one of Chat ID or Allowed User IDs must be set, otherwise the bot will reject all messages.
- **Discord**: Bot Token → confirm (masked) → Allowed User IDs → Allowed Channel IDs (optional) → Allowed Guild IDs (optional). **Important:** At least one of Allowed User IDs or Allowed Channel IDs must be set, otherwise the bot will reject all messages (default-deny).
- **Feishu**: App ID → confirm → App Secret → confirm (masked) → Domain (optional) → Allowed User IDs (optional). After collecting credentials, explain the two-phase setup the user must complete:
  - **Phase 1** (before starting bridge): (A) batch-add permissions, (B) enable bot capability, (C) publish first version + admin approve. This makes permissions and bot effective.
  - **Phase 2** (requires running bridge): (D) run `/claude-to-im start`, (E) configure events (`im.message.receive_v1`) and callback (`card.action.trigger`) with long connection mode, (F) publish second version + admin approve.
  - **Why two phases:** Feishu validates WebSocket connection when saving event subscription — if the bridge isn't running, saving will fail. The bridge needs published permissions to connect.
  - Keep this to a short checklist — show the full guide only if asked.
- **QQ**: Collect two required fields, then optional ones:
  1. QQ App ID (required) → confirm
  2. QQ App Secret (required) → confirm (masked)
  - Tell the user: these two values can be found at https://q.qq.com/qqbot/openclaw
  3. Allowed User OpenIDs (optional, press Enter to skip) — note: this is `user_openid`, NOT QQ number. If the user doesn't have openid yet, they can leave it empty.
  4. Image Enabled (optional, default true, press Enter to skip) — if the underlying provider doesn't support image input, set to false
  5. Max Image Size MB (optional, default 20, press Enter to skip)
  - Remind user: QQ first version only supports C2C private chat sandbox access. No group/channel support, no inline buttons, no streaming preview.
- **Weixin**: Do not ask for a static token. Instead:
  1. Tell the user this channel uses QR login, not manual credential entry.
  2. Run `cd SKILL_DIR && npm run weixin:login`
  3. The helper writes `~/.claude-to-im/runtime/weixin-login.html` and tries to open it automatically in the local browser.
  4. If auto-open fails, tell the user to open that HTML file manually and scan the QR code with WeChat.
  5. Wait for the helper to report success, then confirm that the linked account was saved locally.
  - Explain briefly: the linked Weixin account is stored in `~/.claude-to-im/data/weixin-accounts.json`. Running the helper again replaces the previously linked account.
  - Explain briefly: `CTI_WEIXIN_MEDIA_ENABLED` only controls inbound image/file/video downloads. For voice messages, the bridge only accepts the text returned by WeChat's built-in speech-to-text. If WeChat does not provide a transcript, the bridge replies with an error instead of downloading/transcribing raw audio.
- **DingTalk**:
  1. Ask for DingTalk **App Key**
  2. Ask for DingTalk **App Secret**
  3. Tell the user to enable **Bot** + **Stream mode** on the DingTalk internal app
  4. Explain briefly: group chats only process `@bot` or reply-to-bot messages, v1 replies are plain text only, and inbound images can be forwarded to Claude/Codex

**Step 3 — General settings**

Ask for runtime, default working directory, model, and mode:
- **Runtime**: `claude` (default), `codex`, `auto`
  - `claude` — uses Claude Code CLI + Claude Agent SDK (requires `claude` CLI installed)
  - `codex` — uses OpenAI Codex SDK (requires `codex` CLI; auth via `codex auth login` or `OPENAI_API_KEY`)
  - `auto` — tries Claude first, falls back to Codex if Claude CLI not found
- **Working Directory**: default `$CWD`
- **Model** (optional): Leave blank to inherit the runtime's own default model. If the user wants to override, ask them to enter a model name. Do NOT hardcode or suggest specific model names — the available models change over time.
- **Mode**: `code` (default), `plan`, `ask`

**Step 4 — Write config and validate**

1. Show a final summary table with all settings (secrets masked to last 4 chars)
2. Ask user to confirm before writing
3. Use Bash to create directory structure: `mkdir -p ~/.claude-to-im/{data,logs,runtime,data/messages}`
4. Use Write to create `~/.claude-to-im/config.env` with all settings in KEY=VALUE format
5. Use Bash to set permissions: `chmod 600 ~/.claude-to-im/config.env`
6. Validate tokens — read `SKILL_DIR/references/token-validation.md` for the exact commands and expected responses for each platform. This catches typos and wrong credentials before the user tries to start the daemon. For Weixin, a successful QR login already counts as validation.
7. Report results with a summary table. If any validation fails, explain what might be wrong and how to fix it.
8. On success, tell the user: "Setup complete! Run `/claude-to-im start` to start the bridge."

### `start`

**Pre-check:** Verify `~/.claude-to-im/config.env` exists (see "Config check" above). Without it, the daemon will crash immediately and leave a stale PID file.

Run: `bash "SKILL_DIR/scripts/daemon.sh" start`

Show the output to the user. If it fails, tell the user:
- Run `doctor` to diagnose: `/claude-to-im doctor`
- Check recent logs: `/claude-to-im logs`

### `stop`

Run: `bash "SKILL_DIR/scripts/daemon.sh" stop`

### `status`

Run: `bash "SKILL_DIR/scripts/daemon.sh" status`

### `logs`

Extract optional line count N from arguments (default 50).
Run: `bash "SKILL_DIR/scripts/daemon.sh" logs N`

### `handoff`

Use this to rebind Weixin or DingTalk so future messages continue on the **current** Codex or Claude Code session.

Public handoff commands:

- `handoff weixin`
- `handoff dingtalk`

Run:

- `bash "SKILL_DIR/scripts/handoff.sh" weixin`
- `bash "SKILL_DIR/scripts/handoff.sh" dingtalk`

Behavior:

- If `CODEX_THREAD_ID` exists, treat the current session as Codex and handoff to that thread
- Otherwise detect the current Claude Code session using `CLAUDE_SESSION_ID` → `CMUX_CLAUDE_PID` → current-cwd Claude live sessions, preferring the most recently active same-cwd session
- On success, auto-switch the global `CTI_RUNTIME` to the detected runtime (`codex` or `claude`)
- If no binding exists yet for the target channel, tell the user to send at least one message from the target chat first
- If multiple bindings exist for the target channel, do not guess; tell the user this simplified handoff only supports a single target chat per channel right now
- The helper creates a brand-new local bridge session and keeps old sessions/message files
- After writing the binding, handoff always starts the bridge so the new target chat is immediately usable
- Restarting the bridge drops pending permission requests
- Handoff affects future messages for that channel only; it does not migrate a reply that is already streaming
- In Codex runtime, bridge subprocesses disable Codex response websockets by default for proxy reliability. Set `CTI_CODEX_DISABLE_WEBSOCKETS=false` only if you need Codex's default websocket transport.
- For proxy environments, tell users to put `export HTTPS_PROXY=...`, `export HTTP_PROXY=...`, `export ALL_PROXY=...`, and optional `export NO_PROXY=...` directly in `~/.claude-to-im/config.env`. The bridge forwards uppercase and lowercase proxy variables to the daemon and Claude/Codex subprocesses; do not rely on interactive shell startup files like `~/.zshrc` for background daemons.

Removed commands:

- `handoff projects`
- `handoff threads ...`
- `handoff claude ...`

If the user asks for those, tell them they were removed and to use `handoff weixin` or `handoff dingtalk` from the current conversation instead.

**⚠️ Claude resume limitations (v1):** The resumed Claude session in the bridge does NOT inherit: `--settings`, `--permission-mode`, sandbox flags, or extra allowed directories (`--add-dir`) from the original Claude Code window. The bridge uses only what its own `config.env` provides (`CTI_DEFAULT_MODE`, `CTI_ENV_ISOLATION`, `CTI_AUTO_APPROVE`, etc.). Tell the user clearly: "The resumed session may have different tool permissions and allowed-directory access than your original Claude Code window."

### `reconfigure`

1. Read current config from `~/.claude-to-im/config.env`
2. Show current settings in a clear table format, with all secrets masked (only last 4 chars visible)
3. Use AskUserQuestion to ask what the user wants to change
4. When collecting new values, tell the user where to find the value; only show the full guide from `SKILL_DIR/references/setup-guides.md` if they ask for help
5. Update the config file atomically (write to tmp, rename)
6. Re-validate any changed tokens
7. Remind user: "Run `/claude-to-im stop` then `/claude-to-im start` to apply the changes."

If the user wants to switch Weixin accounts during `reconfigure`, run `cd SKILL_DIR && npm run weixin:login` again. Each successful scan replaces the previously linked local account.

### `doctor`

Run: `bash "SKILL_DIR/scripts/doctor.sh"`

Show results and suggest fixes for any failures. Common fixes:
- SDK cli.js missing → `cd SKILL_DIR && npm install`
- dist/daemon.mjs stale → `cd SKILL_DIR && npm run build`
- Config missing → run `setup`
- Weixin account missing / expired → `cd SKILL_DIR && npm run weixin:login`
- Weixin voice message reports missing speech-to-text → enable WeChat's own voice transcription and resend; the bridge does not transcribe raw voice audio itself

For more complex issues (messages not received, permission timeouts, high memory, stale PID files), read `SKILL_DIR/references/troubleshooting.md` for detailed diagnosis steps.

**Feishu upgrade note:** If the user upgraded from an older version of this skill and Feishu is returning permission errors (e.g. streaming cards not working, typing indicators failing, permission buttons unresponsive), the root cause is almost certainly missing permissions or callbacks in the Feishu backend. Refer the user to the "Upgrading from a previous version" section in `SKILL_DIR/references/setup-guides.md` — they need to add new scopes (`cardkit:card:write`, `cardkit:card:read`, `im:message:update`, `im:message.reactions:read`, `im:message.reactions:write_only`), add the `card.action.trigger` callback, and re-publish the app. The upgrade requires two publish cycles because adding the callback needs an active WebSocket connection (bridge must be running).

## Notes

- Always mask secrets in output (show only last 4 characters) — users often share terminal output in bug reports, so exposed tokens would be a security incident.
- Always check for config.env before starting the daemon — without it the process crashes on startup and leaves a stale PID file that blocks future starts (requiring manual cleanup).
- The daemon runs as a background Node.js process managed by platform supervisor (launchd on macOS, setsid on Linux, WinSW/NSSM on Windows).
- Config persists at `~/.claude-to-im/config.env` — survives across sessions.

---
> Source: [JiangJingC/claude-to-im-plus](https://github.com/JiangJingC/claude-to-im-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
