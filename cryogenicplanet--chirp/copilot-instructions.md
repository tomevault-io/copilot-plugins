## chirp

> - Keep it simple. Build the smallest working slice that meets the current requirement. Do not add speculative abstractions, configuration, packages, fallback paths, or frameworks for hypothetical future needs.

# Working in chirp

- Keep it simple. Build the smallest working slice that meets the current requirement. Do not add speculative abstractions, configuration, packages, fallback paths, or frameworks for hypothetical future needs.
- Prefer direct code and existing Effect services. Extract an abstraction only for a concrete shared need or a safety boundary; do not wrap every function in a service or invent a repository layer over SQL.
- Avoid process-global and module-level mutable state, including mutable containers behind `const`. Own runtime state inside an explicit service instance or scope (for example, `Ref`), and test resources inside each test lifecycle. Keep shared constants readonly; document any unavoidable exception. Ordinary function-local mutation is fine.
- Exception: `packages/cloud/src/auth-runtime.ts` stores its managed request runtimes (authentication and dashboard) on `globalThis` because Next.js compiles the custom server and route handler as separate module instances. `packages/cloud/src/server.ts` is the sole lifecycle owner and awaits disposal after draining requests; no other cloud state may use this escape hatch.
- Keep optional product features in extensions. Simplicity must not weaken bootloader recovery, authentication, or data durability guarantees.
- Test in proportion to failure cost, not coverage targets. Bootloader hardening needs strong failure, concurrency, restart, and real-process integration tests, especially for lost writes, broken edits, auth boundaries, and restore.
- Test server transactions, outbox delivery, cursors, and authorization with focused behavior tests. Keep UI testing minimal: a few critical user-flow smoke checks and visual inspection; no blanket component snapshots or tests of styling, trivial glue, or framework behavior.
- Use subagents for bounded build, discovery, and review work. Parallelize independent work with explicit file ownership; use isolated worktrees for concurrent writers. In one checkout, use one writer at a time and parallel read-only scouts/reviewers.
- The lead agent owns integration and acceptance. Keep shared interfaces small, review meaningful changes with a fresh agent, and remove unnecessary complexity before calling a slice done.
- Only the owner's own words are canonical. `docs/product-intent.md` is the founding statement; never edit it. `docs/decisions.md` records later rulings; add to it only a quotation or a recorded selection, never prose. Everything else is explanation, not requirement. Read each file in full before editing it, and the package docs/README.md before package work.
- Mechanism is documented where it lives: routes at `GET /api`, schemas in the migration ladders, headers in `packages/protocol/src/headers.ts`, and why boot and the server are shaped as they are in `packages/{boot,server}/docs/constraints.md`.
- Distinguish implemented behavior from planned documentation. Check the current code before claiming a feature is available.
- Keep design documents concise and purpose-specific. The intent, decisions, and research split under docs/cloud/ is intentional; do not collapse or delete it to satisfy a documentation-location rule. Record decision provenance honestly; missing quotations are a provenance gap, not a reason to erase the record. Routine contributor instructions belong here or beside the code, operator guidance in README.md or docs/deploy.md, and running-board guidance in packages/server/pages/.
- Run `bun run check` after code changes. Tests are separate; run relevant tests when adding behavior.
- Keep direct dependencies exact; use Bun workspaces, never Turbo.
- Use Effect v4 and its platform services for runtime I/O. Wire layers in main.ts or server.ts.
- Use strict, erasable TypeScript. No any, unchecked casts, ts-ignore, namespace, enum, or parameter properties. Runtime dynamic imports are allowed only in `packages/server/src/kernel/ext.ts` and `packages/server/src/kernel/migrations.ts`; ordinary dependencies use static imports.
- Keep tests in packages/<name>/test/ mirroring src/. No barrel exports except package entry points.
- server/src/main.ts launches boot; server/src/server.ts is the child entry. Keep these separate to prevent recursive spawning.
- Pages are content in packages/server/pages/, not a package. Add a protocol package only when shared schemas are needed.
- Services use Context.Service with a layer export in the same file. Every HttpApi endpoint needs a description.
- repos/ holds upstream sources to learn style from, never to import: `effect/` for how to write good Effect, and `pi-mono/` for how to build minimal, extendable abstractions. Starting points and revisions are in repos/README.md.
- Flag disagreements with product intent or a documented constraint rather than silently changing behavior.
- Multiple agents can share this checkout. Stage explicit paths; never reset, clean, stash, or discard unrelated changes. Commit only when asked.

# Releasing

- The board image (boot and the seed app, one version) is published to `ghcr.io/cryogenicplanet/chirp`. Only the `Release canary` workflow builds it: once `master`'s CI passes it publishes `:sha-<commit>` and moves `:canary`. Merging to `master` is the canary release.
- Promote to `:latest` by pushing a `vMAJOR.MINOR.PATCH` tag on a commit already on `master`. `Release latest` retags that commit's canary build and never rebuilds, so the released image is the one that ran as canary. It refuses a commit without a canary build.
- Never build or push a board image by hand, and never point Cloud or a deployment at an image built from a branch. That is how an image ships without a fix its branch never contained.
- Cloud deploys itself from `master` through `Deploy Cloud`. Releasing a board image and upgrading existing boards are separate: a board keeps the digest it was created with.

---
> Source: [CryogenicPlanet/chirp](https://github.com/CryogenicPlanet/chirp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
