## agent-classes

> The Bessemer **vertical agent-class library**: reliable, bounded-autonomy AI agents for real roles in small organizations. The third layer of the Bessemer stack — **research** (methodology) → **icm-kit** (tooling) → **agent-classes** (product).

# agent-classes — project guide

The Bessemer **vertical agent-class library**: reliable, bounded-autonomy AI agents for real roles in small organizations. The third layer of the Bessemer stack — **research** (methodology) → **icm-kit** (tooling) → **agent-classes** (product).

Each class is anchored by a **charter** (the durable spec a deployment runs against) and, once matured into a working product, a **shared framework package** the deployment imports and configures. **This repo is PUBLIC (MIT). Adapters, per-instance config, and client data live in the private instance repo — never here.**

## The classes

- **Bookkeeper** (`bookkeeper.md`, charter v0.1) — Executor × Standing, vertical-agnostic. **Active build:** the `bookkeeper/` framework package — extract the general core (ports + orchestrator + §5 boundary + config schema + Contracts A/B), then the planned skills one at a time. See the `dev-ready` issue.
- **Legal Admin** — planned.

## Architecture — the ports/adapters line (keep it clean)

A class framework is an **agnostic core** that orchestrates skills through **ports** (interfaces). **Adapters** implement the ports per client/system and live in the private instance repo — never here.

- The reusable, vertical-agnostic logic — the skill ports, the orchestrator, the §5 boundary, the contracts, the config schema — lives in the framework package.
- Anything client/system-specific (a client's DB, inbox, document store) is an **adapter** in that client's private repo.
- New deployment = new adapters + config, reuse the framework. **Never let an adapter's specifics leak into the framework.**

## The autonomy / review boundary (the spine — never weaken it)

Every class enforces a **§5 boundary**: act autonomously only on confident, hand-reversible work; route everything uncertain, consequential, or failed to human review; **inert until configured** (no instance goes live silently auto-filing). Traceability — every item links to its source, every decision is logged with confidence + rule — is the trust wedge, not "never wrong." **This boundary is the product; preserve it exactly in code.**

## Task intake — substrate, not the human

On launch you are a **dev leaf** for this repo. Your brief lives on the work substrate, not in the human's chat. The human launches you with a bare command + the fixed trigger **"begin"**; they never paste a brief, and you never report progress to them directly — it goes on the PR/issue.

1. **Sync first.** `git fetch origin && git checkout develop && git pull` before branching.
2. **Fetch your task.** `gh issue list --label dev-ready --state open` → the lead marks the next in-order issue `dev-ready`; its body is your self-contained brief. One issue per PR, in order.
3. **Bubble up on the substrate.** Open a PR against `develop`, mirroring the issue's acceptance criteria as a checklist. The lead reviews **on the PR**; the human observes, does not relay. Coordinate via PR / issue comments — never by pasting into the human's chat.
4. **Never self-merge.** The lead merges `feature → develop` on a green, AC-passing review; the human gates `develop → main`.

## Conventions

- Branches: `feature/<short-name>` off `develop`. PRs target `develop`.
- Test before commit; put the result in the PR body. One issue per PR. Full-file outputs.
- Every PR is reviewed against its issue's acceptance criteria before merge — mirror the AC as a checklist in the PR body.
- **No Claude attribution in commits.** Do **not** add a `Co-Authored-By: Claude` trailer (or any AI attribution) to commit messages.
- **Some steps need a human** — flag exactly what's needed; never fabricate creds or silently stub past a real auth/migration step.

## Public-cleanliness (non-negotiable — this repo is public MIT)

- **No client data, no client-identifying names, no secrets, no client-system-specific code — ever.** The framework is generic only.
- The framework is *extracted and generalized from* a private instance, but nothing instance-specific crosses the line: ports + orchestrator + config schema + contracts only. Client specifics stay in the private instance repo as adapters.
- If a task tempts you to copy something client-specific in, **stop and route it to review.**

## Scope discipline

Stay in the issue. Keep the framework adapter-free and vertical-agnostic; keep all client/system specifics in the private instance repo. Match surrounding style.

---
> Source: [usebessemer/agent-classes](https://github.com/usebessemer/agent-classes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
