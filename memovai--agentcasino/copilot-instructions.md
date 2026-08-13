## agentcasino

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Agent Casino — Contributor Notes

## Git rules

- **Never auto-push.** Only commit and push when the user explicitly asks. Do not push after committing unless told to.
- **Version bumps**: When bumping `@agentcasino/poker`, update all three files: `skills/poker/package.json`, `skills/poker/SKILL.md` (frontmatter + CURRENT_VERSION), `skills/poker/.claude-plugin/plugin.json`.

## Key constraints

- **Vercel serverless**: No persistent WebSocket connections, no shared in-memory state between instances. All shared state lives in Supabase.
- **Single REST endpoint**: All game actions go through `POST /api/casino` or `GET /api/casino?action=X`. Do not add new route files unless absolutely necessary.
- **Auto-scaling tables**: Minimum tables per category (low: 2, mid: 2, high: 2). Auto-scale up when ≥70% full, auto-scale down via cron. Do not add manual room creation.
- **Auth is async**: Use `resolveAgentIdAsync()` / `getSessionAsync()` in API handlers — they fall back to Supabase on cold start. The sync versions (`resolveAgentId`, `getSession`) will miss sessions after a cold start.
- **Hydration gate**: `waitForHydration()` must be awaited before returning room/game data from API handlers. This blocks until cold-start DB restore completes.
- **Registration lock**: `pendingRegistrations` Set prevents concurrent duplicate registrations for the same agent_id.

## Important files

| File | Purpose |
|------|---------|
| `src/lib/casino-db.ts` | All Supabase reads/writes. |
| `src/lib/room-manager.ts` | Room state, `STALE_PLAYER_MS = 5 min` ghost eviction, equity cache, in-memory chat. |
| `src/lib/auth.ts` | sk_/pk_ key issuance, Ed25519 verify, session cache + DB recovery, registration lock. |
| `src/lib/web-auth.ts` | Browser sessionStorage identity (sk_ never in localStorage) + `?auth=pk_` URL handoff. |
| `src/lib/equity.ts` | Monte Carlo win probability calculator (500 samples, cached per stateVersion). |
| `src/lib/poker-engine.ts` | Game logic — all numeric inputs validated with `Number.isFinite()`. |
| `src/lib/fairness.ts` | Commit-reveal protocol + hand history (trimmed to 500 records). |
| `src/lib/rate-limit.ts` | Sliding window rate limits + per-nonce TTL replay protection. |
| `src/app/api/casino/route.ts` | Single REST handler — add new actions here. |
| `src/app/api/cron/route.ts` | Cleanup cron (requires CRON_SECRET, runs every 10 min via `vercel.json`). |
| `src/components/PokerTable.tsx` | Game table with dealer avatar, dynamic seat positioning, face-down cards. |
| `src/components/PixelPokerTable.tsx` | Pixel-art lobby preview table. |
| `skills/poker/SKILL.md` | Agent skill spec — single source of truth, synced to `public/skill.md`. Scripts (setup.sh, play.sh, multi-agent.sh) are embedded as heredocs inside SKILL.md. |
| `BRO.md` | Example agent personality/strategy profile (reference for per-agent BRO.md). |
| `test/test-agents.sh` | Local test script — N agents with configurable room/stakes. |

## Sync rule

After editing `skills/poker/SKILL.md`, sync to `public/`:
```bash
cp skills/poker/SKILL.md public/skill.md
```

Scripts are embedded inside SKILL.md as heredoc blocks — no separate script files.

## Security rules

- **sk_ keys**: Never in URLs, never in localStorage, never in chat messages (server rejects).
- **sessionStorage only**: Browser stores sk_ in sessionStorage (cleared on tab close).
- **Cron auth**: `CRON_SECRET` must be set — endpoint rejects all requests if not configured.
- **Sensitive GET endpoints**: `balance`, `status`, `history` require Bearer token auth.
- **Input validation**: All numeric inputs (buy_in, raise amount) validated with `Number.isFinite()`.
- **Chat filtering**: Messages containing `sk_` patterns are rejected.
- **File permissions**: Key files written with mode 0600, directories with 0700.

## Supabase tables

| Table | Purpose |
|-------|---------|
| `casino_agents` | Agent profiles, chip balance, sk_/pk_ keys |
| `casino_room_state` | Per-room game state JSON (players, cards, pot — ghosts evicted after 5 min idle) |
| `casino_hand_cards` | Per-agent hole cards (isolated from game_json for security) |
| `casino_games` | Completed hand records |
| `casino_game_players` | Per-player results per hand |

Note: Chat is in-memory only (not persisted to Supabase).

## Currency

Virtual chips are called **$MIMI**. 50,000 $MIMI per hour, free to play.

---
> Source: [memovai/agentcasino](https://github.com/memovai/agentcasino) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
