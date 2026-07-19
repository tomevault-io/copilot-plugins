## cc-to-grok-bridge

> This repo is a **thin bridge**: Claude Code stays the source of truth for rules, skill bodies, and hook **scripts**. Grok Build is a guest that reuses them.

# AI assistant notes — cc-to-grok-bridge

This repo is a **thin bridge**: Claude Code stays the source of truth for rules, skill bodies, and hook **scripts**. Grok Build is a guest that reuses them.

## Do

- Prefer existing scripts under `scripts/` over inventing parallel tooling.
- Keep defaults generic: workspace via `$CC_GROK_WORKSPACE` or `--workspace`, never hardcode a personal machine path.
- After install, remind the user to restart Grok from the **workspace root**.
- **Never** copy secrets from Claude Code config files into this repo or into git-tracked project config.
- For MCP/OAuth: **you (the AI) run the install/login commands**; the user only approves browser consent or provides a secret from their password manager when asked once. Do **not** default to “paste this long block into your terminal”.

## Do not

- Modify `~/.claude/hooks/*.sh` bodies to “fix” Grok — use the adapter.
- Write Grok-only rules into `CLAUDE.md` or CC memory sources.
- Commit `.env`, real MCP tokens, or personal vault paths.
- Name private/personal MCP product codenames in public docs (describe **traits**: API-key HTTP, OAuth Google, Notion stdio, etc.).
- Push to any company GitHub org; personal public repos only when the user asks.

## Compatibility (summary)

| Domain | Compatibility | Notes |
|--------|---------------|--------|
| System Prompt | High | Same workspace `CLAUDE.md` (compat) |
| skill | High | Same `~/.claude/commands` (symlinks OK); minor frontmatter/trigger diffs |
| Memory | High | Pull + rules pointer + 3-zone layout; product search optional |
| Hooks | Medium | Adapter for payload + deny; no full CC-style ask UI |
| MCP | Medium | Manual/AI-assisted per server; claude.ai cloud connectors not portable |
| Plugins | Low | No auto-port of CC `enabledPlugins`; see docs/06_plugins.md |

Real-world note: bringing a CC setup into **Grok Build** is usually much smoother than the Antigravity/Gemini bridge path (hooks hard-block + simpler memory).

## Memory (for AI)

1. Run `python3 scripts/memory_sync.py --workspace <ws>` after CC memory changes.  
2. Confirm `<ws>/.grok/rules/cc-memory-pointer.md` exists.  
3. Do **not** tell the user that bridge “requires” `[memory] enabled=true` to load the pointer — rules load without it.  
4. Optional: if they want product `memory_search`, help enable via `/memory on` or config.  
5. Push only with explicit user request: `memory_push.py` on `general/` only.

## Plugins（給 AI）

CC auto-enabled plugins (marketplace + SessionStart always-on) **do not** transfer to Grok.  
Users install plugins **to avoid manual slash** — do **not** recommend “just run /skill each session” as a migration path.

1. Run `grok plugin list`. Empty while CC has plugins enabled is expected.  
2. For each always-on capability:
   - **A** — Real Grok plugin packaging exists (merged upstream or user-approved ref) → `grok plugin install <source> --trust` after user OK; enable + restart session.  
   - **B** — No Grok packaging yet → instruction always-on in `<workspace>/.grok/rules/<name>.md` (auto-load, no slash).  
3. Never assume `claude plugin install` ≡ `grok plugin install`.  
4. Do not auto-install unmerged/dirty forks with `--trust` unless the user explicitly accepts that risk; default to B while waiting for upstream.  
5. Details: [docs/06_plugins.md](docs/06_plugins.md).

## MCP 部署手冊（給 AI 手把手執行）

詳細型態表見 [docs/03_mcp.md](docs/03_mcp.md)。以下是你應**直接執行**的流程。

### 共通前置

1. Confirm `grok` is on PATH: `grok --version` or `which grok`.  
2. `grok mcp list` — baseline.  
3. Never read `~/.claude.json` / CC project MCP blobs for secret values.  
4. Prefer **user scope** (`~/.grok/config.toml`) for anything with credentials.  
5. After add: `grok mcp doctor [name]`；ask user to **restart Grok session**.

### A) API key / Bearer HTTP remote MCP

```text
User provides: server display name, URL, and secret (from password manager) once in chat (or env var name already set).
You run (example shape — replace placeholders):

  grok mcp add --transport http <name> <url> --header "Authorization: Bearer <token>"

Or stdio with env:

  grok mcp add <name> -e API_KEY=<token> -- npx -y <package> ...

Then: grok mcp list && grok mcp doctor <name>
```

Do not print the full token back in summaries.

### B) Notion（獨立 MCP，不搬 claude.ai connector）

1. Explain: claude.ai Notion connector is **not portable**.  
2. Pick a maintained Notion MCP (stdio/HTTP) with clear docs.  
3. Ask user once for integration token (Notion developer portal).  
4. **You** run `grok mcp add …` with token in user scope / env.  
5. Verify with doctor + a read-only tool call in a new session.

### C) Google（Drive / Gmail / Calendar 等）

1. Explain: claude.ai Google connectors are **not portable**.  
2. Prefer MCP packages that open **browser OAuth / device flow**.  
3. **You** start the login command; user only clicks Allow in the browser.  
4. **Forbidden as default UX**: multi-line `export …` paste homework for the user.  
5. If only API-key Google APIs exist for a niche tool: treat as type A; store key in user env, not git.  
6. Verify doctor + one harmless list/read call.

### D) OAuth 通用原則

- AI drives CLI (`grok mcp add`, package `auth`/`login`, etc.).  
- User action = browser consent or one-time secret from password manager.  
- On failure: lengthen `startup_timeout_sec` for cold `npx`, re-run doctor, check scope (user vs project).

### E) 本地 CLI（非 MCP）

If the capability is already a CLI + skill under `~/.claude/commands`, do not force MCP. Ensure CLI login state exists; Grok can use the skill.

## Verify（hook／adapter 改完必跑 — 全數，不是煙測）

```bash
export CC_GROK_WORKSPACE=~/path/to/your-workspace
# optional real clasp sandbox:
# export CC_GROK_CLASP_SANDBOX=~/path/to/clasp-run-sandbox

python3 scripts/bridge_doctor.py --workspace "$CC_GROK_WORKSPACE"
python3 scripts/hook_acceptance.py --with-clasp
```

- doctor：`fails=0`  
- acceptance：`=== N/N PASS ===` 且 exit 0  

未過不准宣稱 DONE，不准 push。

## Key docs

- [architecture.md](architecture.md) — invariants  
- [docs/01_絲滑啟動.md](docs/01_絲滑啟動.md) — daily SOP  
- [docs/02_memory.md](docs/02_memory.md) — memory  
- [docs/03_mcp.md](docs/03_mcp.md) — MCP types + Notion/Google  
- [docs/06_plugins.md](docs/06_plugins.md) — plugins gap + migration paths  
- [README.md](README.md) — compatibility matrix for humans  

---
> Source: [yyu0310/cc-to-grok-bridge](https://github.com/yyu0310/cc-to-grok-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
