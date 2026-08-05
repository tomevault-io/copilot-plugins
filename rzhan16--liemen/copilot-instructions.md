## liemen

> Build and stabilize 猎门 V1 in 14 calendar days with no scope expansion.

# AGENTS.md

## Project Mission
Build and stabilize 猎门 V1 in 14 calendar days with no scope expansion.

## Working Mode
- Default delivery mode: thin vertical slices.
- Every task must cross layers end-to-end where relevant (config -> logic -> UI -> persistence -> test).
- Avoid horizontal batching (for example "all UI first" or "all backend first") unless explicitly marked as dependency groundwork.
- Multi-lane setup: `raymond` (core/backend), `partner-a` (systems/content), `partner-b` (UI/AI design).
- All agents must respect ownership in `docs/operations/WORKSPLIT-RAYMOND-PARTNER.md`.

## Hard Scope Lock (Do Not Expand)
- World/lore + rank progression E -> S
- Core loop: Portal -> Auto-battle -> Loot -> Upgrade -> Push
- 4 hunters total (1 main + 3 recruitable), 4 classes
- Team size: 3 active, no support slot
- 1 zone only, 8 normal + 1 boss
- Auto battle + manual ultimate + x2 speed
- Gear: 4 slots, 3 rarities, 4 set bonuses
- Crafting: enhance only
- AFK rewards with cap + rewarded ad extra claim
- Summon: 1 banner + simple pity + beginner guarantee
- Daily quests only
- Achievements + leaderboard (power + chapter)
- Shop: starter pack/month pass/bundle UI + staged payment integration
- Account/login + local/cloud-lite save
- Minimal analytics + privacy/terms/probability pages

## Out Of Scope (V1.1)
- Friend assist
- Limited-time event system
- Cosmetic skins

## Engineering Rules
- Use TypeScript throughout client and server contracts.
- Keep config-driven gameplay values in JSON tables under `configs/`.
- Never hardcode economy numbers in gameplay scripts.
- Add tests for behavior through public interfaces, not internal implementation details.
- Prefer tracer-bullet increments with demoable outcomes each day.

## Delivery Rules
- No new feature starts after Day 8 without explicit owner approval.
- Any blocker > 4 hours triggers scope-trim proposal in daily log.
- All new systems require:
  - acceptance checklist
  - telemetry event mapping
  - persistence impact note

## Definition Of Done (Feature)
A feature is done only when:
1. Gameplay behavior works on device in WeChat DevTools.
2. Required data schema/config exists and is versioned.
3. State persistence path is validated (or explicitly not needed).
4. At least one verification step exists (test or reproducible QA steps).
5. Telemetry events are emitted for key actions.

## Agent Skills (inspired by mattpocock/skills)
### Issue tracker
Local markdown issue tracker in `.scratch/issues/` using vertical-slice issue docs. See `docs/agents/issue-tracker.md`.

### Triage labels
Canonical state labels are used inside issue frontmatter/comments. See `docs/agents/triage-labels.md`.

### Domain docs
Single-context layout with root `CONTEXT.md` and `docs/adr/`. See `docs/agents/domain.md`.

## Execution Order
Follow `docs/roadmap/MASTER-2-WEEK-PLAN.md` strictly.

---
> Source: [Rzhan16/liemen](https://github.com/Rzhan16/liemen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
