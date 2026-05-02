## hoxxes-briefing

> Build and maintain the Hoxxes Briefing SPA + Hono monorepo. Favor correctness,

# AGENTS

## Mission

Build and maintain the Hoxxes Briefing SPA + Hono monorepo. Favor correctness,
clarity, and launch readiness over speculative features.

## Sources Of Truth

- System shape: [docs/architecture.md](docs/architecture.md)
- Product and UX intent: [docs/product.md](docs/product.md)
- Deep Rock Galactic vocabulary: [docs/domain.md](docs/domain.md)
- Web UI implementation rules: [apps/web/docs/ui-system.md](apps/web/docs/ui-system.md)

## Local Context

- Check nested `AGENTS.md` files before changing `apps/`, `packages/`, or
  `crates/`.
- Keep agent guidance short. Put human documentation in `README.md` or `docs/`.
- Do not create new LLM-specific planning docs under `docs/` unless explicitly requested.

## Safe Commands

The following commands are safe to run when relevant:

- `pnpm check`
- `pnpm test`
- `pnpm build`
- `cargo test --workspace`
- `cargo bench -p drg_mission_gen_wasm` when bench work is explicitly requested

Long-running commands may be used only when needed for manual verification:

- `pnpm dev:web`
- `pnpm dev:api`

## Verification

- UI/runtime changes: run `pnpm test` and `pnpm build`.
- Rust boundary changes: run `cargo test --workspace`.
- Cross-stack changes: run `pnpm check`.

## Repo Rules

- Keep public docs and planning artifacts in English unless a task explicitly
  states otherwise.
- When docs and code disagree, prefer verified code and update docs in the same
  change when feasible.
- Prefer small, behavior-focused tests over broad snapshots.
- Use Conventional Commits for commit messages.

## Engineering Standards

### YAGNI (You Aren't Gonna Need It)

Detection patterns:

- Unused parameters "for future use"
- Abstract classes with single implementation
- Configuration options never used
- Speculative generalization

Guidance:

- Build for current requirements
- Refactor when needs emerge
- Delete dead code when it is proven unused and within the task scope.
- Prefer simple over flexible

### KISS (Keep It Simple, Stupid)

Detection patterns:

- Cyclomatic complexity > 10 as a review smell, not an automatic rewrite trigger
- Nested callbacks/promises 4+ deep
- Generic solutions for specific problems
- Framework overkill for simple tasks

Refactoring:

- Flatten control flow
- Extract named functions
- Use early returns
- Choose boring technology

---
> Source: [khmm12/hoxxes-briefing](https://github.com/khmm12/hoxxes-briefing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
