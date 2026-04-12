## syncvibe

> This project uses SyncVibe for team coordination. All shared state lives in `.syncvibe/`.

## SyncVibe Collaboration

This project uses SyncVibe for team coordination. All shared state lives in `.syncvibe/`.

### Before starting work
- Read `.syncvibe/chat-log.jsonl` (last 20 lines) for recent team discussions.

### Chat
- Chat is append-only JSONL in `.syncvibe/chat-log.jsonl`. One JSON object per line.
- To send a message: append a line with `{"id":"<uuid>","user_id":"...","user_name":"...","user_color":"...","content":"...","message_type":"user","thread_id":null,"session_id":"...","timestamp":"..."}`.
- If SyncVibe MCP server is available, use `read_chat` for smart filtered/incremental reads.

### Cross-Repo Awareness
- SyncVibe has 3 components: **CLI** (Rust, this repo), **Relay** (Cloudflare Workers, syncvibe-relay), **Web** (React/Vite, syncvibe-web)
- When modifying features, consider the impact on all 3 components (e.g. protocol changes affect CLI + relay, auth changes affect all 3)

### Agent Message Flow
- Agent messages use `user_id` with `agent-` prefix (e.g. `agent-claude`)
- TUI `reload_data()` detects `agent-` messages from disk and broadcasts via WebSocket
- Relay preserves agent identity: only connections with matching `agentId` can send as `agent-{id}`; human messages are always stamped with authenticated identity
- Remote @mentions trigger local agent via tmux send-keys (30s debounce)
- Dedup prevents broadcast loops: `msg_id_set` insert before broadcast, relay excludes sender

### UI Consistency
- Any UI changes must match the existing visual style and design language
- Follow established patterns for colors, spacing, typography, and component structure

### Release Checklist
Before every version release:
1. **README.md** — Update to reflect any new features, commands, or changes
2. **ARCHITECTURE.md** — Update if architecture, protocols, or components changed
3. **Release notes** — Write clear changelog with bullet points for each release (never leave it as just a "Full Changelog" link)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Curious1008)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Curious1008)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
