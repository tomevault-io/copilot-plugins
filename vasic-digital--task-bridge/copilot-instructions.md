## task-bridge

> All rules in `constitution/CLAUDE.md` (and the `constitution/Constitution.md`

## INHERITED FROM constitution/CLAUDE.md

All rules in `constitution/CLAUDE.md` (and the `constitution/Constitution.md`
it references) apply unconditionally. The project-specific rules below extend
them — they do NOT weaken any universal clause. When this file disagrees with
the constitution submodule, the constitution wins.

> The constitution submodule is the canonical root (§11.4.35). When this repo is
> consumed as a submodule, the parent project's `constitution/` provides it. The
> universal anti-bluff covenant (§11.4), no-guessing mandate (§11.4.6),
> credentials-handling mandate (§11.4.10), decoupling mandate (§11.4.28),
> data-safety (§9), and host-session safety (§12) all apply here.

# task_bridge — project rules (consumer extensions)

`task_bridge` is a **generic, project-agnostic, fully-decoupled** bidirectional
task/board sync engine (Go). The ONLY project-specific rules it carries are the
rules that keep it decoupled.

## DECOUPLING CONTRACT (constitution §11.4.28) — load-bearing

1. **ZERO project specifics in this repo.** No hardcoded credentials, board IDs,
   folder IDs, list IDs, package names, hostnames, regions, app names, or asset
   names. A `grep -rni '<consumer-identifiers>'` over this tree MUST be empty
   (the consumer substitutes its own project/asset/region identifiers).
2. **Everything injected at runtime** through `pkg/config.Config`. The consumer
   reads its own `.env` / secret store / board URLs and passes them in. The
   engine reaches the consumer ONLY through that struct.
3. **Fully reusable + modular + completely testable** by ANY project. ClickUp is
   the first remote member; the `pkg/client`, `pkg/mapper`, `pkg/resolver`
   interfaces keep it swappable (Jira/Linear later) without touching the engine.

## CREDENTIALS (§11.4.10)

- The API token is injected (`Config.APIToken`), **never logged**, **never
  persisted** by the engine, **never written** to the DB or any doc.
- `.env` / `.env.*` / `*.env` are gitignored (§11.4.30); only `.env.example`
  (placeholders) is tracked.

## DATA SAFETY (§9 / §11.4.122)

- Default `DeleteBehavior` is **never-auto-delete-remote**: the engine NEVER
  issues a remote `DELETE` on its own. A local `Deleted` marks the item Deleted
  in the docs/DB and (optionally) a remote `Deleted` status, but destroys no
  remote data unless the consumer explicitly opts into `AllowRemoteDelete`.

## ANTI-BLUFF (§11.4 / §11.4.27)

- Dry-run is the default; real remote writes require explicit `--apply`.
- Scaffold stubs return explicit not-implemented errors — they NEVER fake
  success. Fakes/mocks live ONLY in unit tests; integration/e2e/chaos/stress
  tests exercise the real engine against a mock ClickUp (no real writes).
- Every gate ships with a paired §1.1 mutation; every PASS cites captured
  evidence (§11.4.5 / §11.4.69).

## STACK

Go (matches the `workable-items` + `docs_chain` stack), wrapping the MIT
`raksul/go-clickup` transport. One binary + one daemon; the LLM is glue/trigger
only (§11.4.141).

## COMMIT / PUSH

Pushed to **GitHub + GitLab** under `vasic-digital` (§2.1 multi-upstream,
absolute no-force-push §11.4.113). `upstreams/` recipes + `install_upstreams`
(§11.4.36) configure the remotes.

---
> Source: [vasic-digital/task_bridge](https://github.com/vasic-digital/task_bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
