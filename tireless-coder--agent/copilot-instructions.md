## tireless

> Tireless cloud dev computer — connect, remote exec over ssh <workspace>.tireless, lifecycle guardrails


## Tireless workspace (cloud dev computer)
- The user has a Tireless cloud dev computer (tirelesscode.com); SSH alias: `ssh <workspace>.tireless`.
- Run remote commands with native ssh: `ssh <alias> 'cd <dir> && <cmd>'` (alias = the connection card's ssh_alias, e.g. myws.eu-central.tireless) — every call is a fresh shell, so always cd-prefix. Probes: add `-o BatchMode=yes -o ConnectTimeout=10`. Jobs >2 min: `tmux new -d` then poll. Cap output with `| tail -100`.
- Prefer the tireless_* MCP tools (status, login, connect_workspace — the one-call connect pipeline — list/get workspace, actions, watch_state, share_port, open_editor, clipboard_status, doctor, marketplace_browse/marketplace_listing, recipes_catalog, install_app/install_status, workspace_health, game_port, exposure_overview) when available; full skills live in ~/.agents/skills/tireless/.
- First-time/platform logins are interactive and belong in the USER'S OWN terminal: `tireless-connect login` (platform + regions) or `tireless login <cpUrl>` (single region). Never ask for or echo tokens in chat.
- An EXPIRED per-cell Coder session is ROUTINE, not a fault: cells cap them at 8h, so expect it after any night. Tells: ssh or the CLI says `session has expired` / `You are signed out` / `not logged in`; preflight says `AUTH=expired`; verify says `CAUSE=session_expired`. Fix it yourself with `sh ~/.agents/skills/tireless/scripts/session-heal.sh` — server-minted, no browser, no paste, no token in chat — then retry and carry on without making it a topic. Never run `tireless-connect login` from your own shell to heal: with a dead platform token it opens a browser and blocks. `AUTH=error` is a network problem, not expiry — do not heal into it.
- There are TWO auth layers and only one of them is this. The PLATFORM sign-in (`tireless_status` → `signed_in`, `token_expires_at`, repaired by `tireless_login`) lasts ~1h and refreshes ITSELF — a `token_expires_at` in the past is normal and is not a finding. The per-cell Coder session is what ssh actually uses, and nothing refreshes it. So `signed_in: true` does not mean ssh works, and `tireless_login` answering "already signed in" is not a repair (newer connectors also heal the cell session and say so under `workspace_sessions`; older ones do not). When ssh fails on auth, heal the cell session FIRST and do not investigate the platform token at all.
- Never run `tireless start|stop|delete|create` and never mutate workspaces via the Coder API — the platform reconciler owns lifecycle. Restart/suspend/resume only via tireless_workspace_action; deletion is dashboard-only.
- Creating a workspace costs money: only via tireless_create_workspace, and only after the user explicitly says yes in the conversation.
- Never publicly share ports 22, 13337, 6800, 6801, 6810, 19985 — they are the workspace itself.
- Marketplace/installs: the browse and recipes catalogs are read-only — there is no purchase tool; renting is dashboard-only. install_app may reveal secret param values shown ONCE — relay them to the user immediately, never store them. game_port opens an allowlisted raw-TCP port to the whole internet — only on the user's explicit say-so; closing is always fine.
- Image paste into agents on the workspace is Ctrl+V (not Cmd+V); if broken run `tireless-clip ensure-daemon` and reconnect, or use the dashboard Paste page (images stay valid 5 minutes).
- Editors: connect_workspace first, then open_editor. Claude uses editor=claude (new local session with a prefilled Tireless connection request); VS Code uses vscode://vscode-remote/ssh-remote+<alias>/home/dev/<workspace> (cursor:// twin; append /<repo> to open a specific project).
- "Continue this on my workspace" = the continue skill: probe with ~/.agents/skills/tireless/scripts/handoff-state.sh, sync branch + uncommitted work + env files + a handoff brief with handoff-sync.sh (direct git-over-ssh, no GitHub creds needed), start the remote agent in tmux with handoff-launch.sh. Briefs live at ~/.timeless/handoffs/ (newest: latest.md). The sync also writes the workspace stanza into the project's own CLAUDE.md/AGENTS.md automatically (`PROJECT_NOTE=` in its output; `--no-project-note` opts out) — don't ask whether to add it.
- If anything fails, diagnose with the tireless_doctor tool (or ~/.agents/skills/tireless/scripts/preflight.sh) and apply only the mapped fix.

Full skills: ~/.agents/skills/tireless/ (connect, fix, workspace, marketplace, clipboard, continue).

---
> Source: [Tireless-Coder/agent](https://github.com/Tireless-Coder/agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
