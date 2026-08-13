## shotoku

> Shotoku is a local-first authorization layer for AI agents.

# AGENTS.md

# Shotoku

Shotoku is a local-first authorization layer for AI agents.

Before an agent performs an action — calling a paid API, using an MCP tool, executing code, sending an email, deploying infrastructure, or spending money — Shotoku evaluates the request, applies policy, and records the decision locally.

Built first with x402. Designed for any payment or action rail.

No custody ever.

## Product One-Liner

Shotoku lets developers require approvals, enforce limits, and record auditable decisions before AI agents act.

## Core Question

Every feature should serve one question:

> Should this agent be allowed to perform this action?

If a feature does not improve authorization, approvals, policy enforcement, explanations, or auditability, it probably does not belong in v1.

---

# Current Repo Structure

Use the current repo structure:

```txt
shotoku/
├── core/
│   └── src/
├── cli/
│   └── src/
├── docs/
├── package.json
├── pnpm-workspace.yaml
└── tsconfig.base.json
```

Do not introduce `packages/core` or `packages/cli` unless explicitly asked. This repo currently uses `core/` and `cli/` directly.

## Packages

- `core/` — authorization, policy, ledger, types, and integrations.
- `cli/` — command-line interface and Ink TUI.
- `docs/` — API docs, decisions, positioning, non-goals, quickstarts.

---

# Brand and Positioning

## What Shotoku Is

Shotoku is:

- Local-first
- Developer-first
- Open-source
- Authorization-first
- Approval-oriented
- Audit-focused
- Simple enough for agent builders and “vibe coders” to understand

## What Shotoku Is Not

Shotoku is not:

- A wallet
- A payment processor
- A payment network
- A custody provider
- A banking product
- An enterprise procurement system
- An ERP
- A generic workflow engine
- A cloud-first compliance suite

## Important Positioning Rule

Do not position Shotoku as only an “agent spending” product.

Spending is the first wedge. Authorization for agent actions is the category.

Good:

```txt
Local-first authorization layer for AI agents.
```

Bad:

```txt
Payment layer for AI agents.
```

Good:

```txt
Built first with x402. Designed for any rail.
```

Bad:

```txt
x402 company.
```

---

# Technical Principles

## Hard Rules

1. Commit every day.
2. Ship something every Friday.
3. No features during launch polish unless critical.
4. Tests before features.
5. No vibe-coded mystery code.
6. Every line should be explainable.
7. TypeScript strict mode stays on.
8. Do not use `any`.
9. Prefer interfaces for object shapes.
10. Keep modules small and focused.
11. No custody logic.
12. No private key storage.
13. No cloud requirement in the core authorization path.

## Engineering Style

- TypeScript strict.
- Prefer pure functions in `core`.
- Avoid side effects except in clearly named I/O modules.
- Separate types from implementation.
- Keep public APIs small.
- Make error messages human-readable.
- Do not expose raw stack traces to CLI users.
- Use `readonly` fields for request/response models.
- Prefer explicit imports from Node built-ins, for example `node:crypto`.

---

# Core API Shape

The core API revolves around `authorize()`.

## Request

```ts
export interface AuthorizeRequest {
  readonly actor: string;
  readonly action: AgentAction;
  readonly resource: string;
  readonly rail?: ExecutionRail;
  readonly amount?: number;
  readonly context?: Record<string, unknown>;
}
```

## Response

```ts
export interface AuthorizeResponse {
  readonly approved: boolean;
  readonly status: AuthorizationStatus;
  readonly reasons: readonly ReasonItem[];
  readonly decisionId: string;
  readonly timestamp: string;
}
```

## Reason Item

```ts
export interface ReasonItem {
  readonly type:
    | "policy_match"
    | "budget_check"
    | "limit_check"
    | "blocked"
    | "escalated";

  readonly text: string;
}
```

## Status

```ts
export type AuthorizationStatus = "approved" | "denied" | "pending_approval";
```

## Execution Rail

```ts
export type ExecutionRail = "x402" | "mcp" | "api" | "code" | "custom";
```

## Agent Action

```ts
export type AgentAction =
  | "purchase"
  | "api_call"
  | "execute_code"
  | "send_email"
  | "mcp_tool"
  | "custom";
```

---

# Desired Core File Structure

Prefer this structure as the project grows:

```txt
core/src/
├── authorize.ts
├── types.ts
├── policy.ts
├── ledger.ts
├── explain.ts
├── errors.ts
└── index.ts
```

## Responsibilities

### `types.ts`

Only domain types and interfaces.

### `authorize.ts`

Public authorization entrypoint.

### `policy.ts`

Pure policy evaluation.

### `ledger.ts`

Local decision storage and audit log.

### `explain.ts`

Human-readable explanations.

### `errors.ts`

Typed internal errors and user-safe error formatting.

### `index.ts`

Public exports only.

Keep `index.ts` boring:

```ts
export * from "./authorize.js";
export * from "./types.js";
```

---

# CLI Principles

The CLI is the screenshot surface. It should look intentional.

## Voice

- Terse.
- Precise.
- No marketing words.
- Present tense.
- Every message should explain what happened and what to do next.

## Output Symbols

Use:

```txt
✓ APPROVED
✗ DENIED
◷ PENDING APPROVAL
```

## Approved Output

```txt
✓ APPROVED  dec_abc123
  • OpenAI is allowlisted
  • Daily budget remaining: $475
  • Transaction below $50 limit
  Recorded at 14:05:22
```

## Denied Output

```txt
✗ DENIED  dec_def456
  • anthropic.com is not allowlisted
  • Requires explicit approval
  → Run: shotoku approve dec_def456
```

## Pending Output

```txt
◷ PENDING APPROVAL  dec_ghi789
  • vendor-xyz.com is not on the allowlist
  • A human must approve this decision
  → Run: shotoku approve dec_ghi789
```

---

# Common Commands

Use pnpm.

## Install

```bash
pnpm install
```

## Build

```bash
pnpm build
```

## Typecheck

```bash
pnpm typecheck
```

## Run CLI Dev

```bash
pnpm --filter shotoku-cli dev
```

## Run Core Tests

```bash
pnpm --filter @shotoku/core test
```

## Run CLI Tests

```bash
pnpm --filter shotoku-cli test
```

---

# Commit Style

Use Conventional Commits.

Examples:

```txt
chore: scaffold pnpm monorepo
docs: authorization API design
feat: policy evaluation engine
feat: immutable decision ledger
feat: end-to-end authorization
feat: shotoku authorize command
fix: user-facing error messages
docs: quickstart and API reference
```

Do not use `feat:` for documentation-only changes.

---

# Development Roadmap

Keep the day-by-day structure, but do not treat specific dates as constraints. Each “day” is a work unit.

---

# BLOCK 1 — Foundations

## Goal

`authorize()` works end-to-end. A developer can clone the repo, run one command, and get a decision with reasons.

---

## Week 1 — Scaffold + API Design

### Day 1 — Monorepo Setup

Status: mostly complete.

Tasks:

- Confirm GitHub org is `shotoku-dev`.
- Confirm repo is `shotoku-dev/shotoku`.
- Confirm pnpm monorepo works.
- Confirm folder structure:

  - `core/`
  - `cli/`
  - `docs/`

- Confirm TypeScript strict mode across packages.
- Confirm root `tsconfig.json`.
- Confirm per-package `tsconfig.json`.
- Confirm `pnpm-workspace.yaml`.
- Confirm README has the authorization-first positioning.
- Confirm CLI renders first Ink screen.

Expected commit:

```txt
chore: scaffold pnpm monorepo
```

---

### Day 2 — CI + Conventional Commits

Tasks:

- Add `.github/workflows/ci.yml`.
- CI should run:

  - `pnpm install --frozen-lockfile`
  - `pnpm build`
  - `pnpm typecheck`
  - `pnpm test`

- Add `.gitignore`.
- Ignore:

  - `node_modules`
  - `dist`
  - `*.db`
  - `.env`
  - `.DS_Store`

- Add ESLint.
- Add Prettier.
- Add commitlint and husky only if the setup stays simple.
- Do not overcomplicate repo automation.

Expected commit:

```txt
chore: CI pipeline and commit conventions
```

---

### Day 3 — Authorization Type Definitions

Tasks:

- Create or update `core/src/types.ts`.
- Define:

  - `AuthorizeRequest`
  - `AuthorizeResponse`
  - `ReasonItem`
  - `AuthorizationStatus`
  - `AgentAction`
  - `ExecutionRail`
  - `Policy`
  - `PolicyRule`
  - `LedgerEntry`
  - `LedgerSnapshot`
  - `EvaluationResult`

- Create `docs/api.md`.
- Document the API from the types.
- Keep examples short and copy-pasteable.

Expected commit:

```txt
docs: authorization API design
```

---

### Day 4 — Policy Engine, TDD

Write tests first.

Test file:

```txt
core/src/__tests__/policy.test.ts
```

Required tests:

- Known vendor, under limit → approved.
- Known vendor, over daily limit → denied with reason.
- Unknown vendor → pending approval with reason.
- No policy file → denied with reason.
- Wildcard rule matches unknown action → pending approval.

Implementation file:

```txt
core/src/policy.ts
```

Main function:

```ts
export function evaluatePolicy(
  request: AuthorizeRequest,
  policy: Policy,
  ledger: LedgerSnapshot,
): EvaluationResult;
```

Requirements:

- Pure function.
- No I/O.
- No filesystem.
- No SQLite.
- No network.
- Checks resource allowlist.
- Checks action.
- Checks per-action or per-transaction amount limit.
- Checks daily limit against ledger snapshot.
- Returns structured reasons.

Expected commit:

```txt
feat: policy evaluation engine
```

---

### Day 5 — Local Decision Ledger

Start simple. Prefer append-only JSONL first unless SQLite is explicitly required.

The ledger must be:

- Local.
- Append-only.
- Auditable.
- Easy to inspect.
- Easy to test.

Recommended first implementation:

```txt
data/decisions.jsonl
```

Each line is one decision record.

Tests:

- Insert decision → readable back.
- Append-only behavior.
- Snapshot returns correct daily total for actor and resource.
- Query by status works.
- Query by actor works.
- Query by time window works.

Optional later upgrade:

- SQLite.
- Hash-chain.

Do not block the first version on SQLite complexity.

Expected commit:

```txt
feat: local decision ledger
```

---

## Week 2 — Wire + CLI

### Day 6 — `authorize()` End-to-End

Implementation file:

```txt
core/src/authorize.ts
```

Function shape:

```ts
export async function authorize(
  request: AuthorizeRequest,
  options: {
    readonly policyPath: string;
    readonly ledgerPath: string;
  },
): Promise<AuthorizeResponse>;
```

Tasks:

- Load policy file.
- Get ledger snapshot.
- Call policy evaluator.
- Write decision to ledger.
- Return `AuthorizeResponse`.
- Add integration test with real policy file and real local ledger.
- Verify full round trip.

Expected commit:

```txt
feat: end-to-end authorization
```

---

### Day 7 — CLI Setup + `shotoku authorize`

CLI package:

```txt
cli/
```

Use `commander` or `yargs`.

Command:

```bash
shotoku authorize --actor <id> --action <action> --resource <resource> [--amount <n>]
```

Tasks:

- Parse CLI args.
- Call `authorize()` from `@shotoku/core`.
- Print approved output.
- Print denied output.
- Print pending output.
- Handle missing args gracefully.
- Handle invalid amount gracefully.
- No raw stack traces.

Expected commit:

```txt
feat: shotoku authorize command
```

---

### Day 8 — CLI `history`, `status`, and `decision`

Commands:

```bash
shotoku history
shotoku history --actor <id>
shotoku history --since <24h|7d|30d>
shotoku history --status <approved|denied|pending_approval>
shotoku status
shotoku decision <id>
```

Tasks:

- History output table with ✓ / ✗ / ◷ icons.
- `status` shows pending count and last decision.
- `decision <id>` shows full detail.
- Empty states should be clean.
- Invalid filters should show helpful errors.

Expected commit:

```txt
feat: history and status commands
```

---

### Day 9 — Init Command + Examples

Tasks:

- Implement `shotoku init`.
- `shotoku init` creates:

  - `shotoku.config.json` or `shotoku.yaml`
  - `policy.yaml`
  - `data/`

- Add `examples/policy.yaml`.
- Add `examples/quickstart.ts`.
- Test full flow from a clean folder.
- Fix rough CLI output.

Expected commit:

```txt
feat: init command and example policy
```

---

### Day 10 — Block 1 Ship

Tasks:

- CI green.
- Tests pass.
- README accurate.
- `pnpm build` passes.
- `pnpm typecheck` passes.
- Terminal screenshot showing one clean `✓ APPROVED` output.
- Post public progress update.
- Keep post factual and short.

Expected commit:

```txt
chore: Block 1 complete
```

Do not cut a release yet.

---

# BLOCK 2 — Approvals + Explanations

## Goal

Humans can approve or deny pending decisions. MCP server works. CLI explanations are strong.

---

## Week 3 — Approval Queue + Explanations

### Day 11 — Approval Queue Data Layer

Tasks:

- Add approval entries to the ledger.
- Approval is a new append-only entry.
- Never mutate the original decision.
- Add:

  - `approve(decisionId)`
  - `deny(decisionId)`

- Write tests:

  - Approve pending decision.
  - Deny pending decision.
  - Cannot approve missing decision.
  - Cannot approve already-actioned decision.
  - Approval creates a new ledger entry.

Expected commit:

```txt
feat: approval queue data layer
```

---

### Day 12 — `approve` and `deny` CLI Commands

Commands:

```bash
shotoku approve <decisionId>
shotoku deny <decisionId>
```

Output:

```txt
✓ Approved. Recorded as apr_xxx.
```

Error cases:

- Decision not found.
- Decision already actioned.
- Decision is not pending.

Expected commit:

```txt
feat: approve and deny CLI commands
```

---

### Day 13 — Status With Pending Approvals

Upgrade:

```bash
shotoku status
```

It should show:

- Pending approvals list.
- Actor.
- Resource.
- Amount.
- Age.
- Reason summary.
- Footer with approve/deny hint.

Empty state:

```txt
No pending approvals.
```

Expected commit:

```txt
feat: status command with pending approvals
```

---

### Day 14 — Rich Explanations

Add or improve:

```txt
core/src/explain.ts
```

Requirements:

- Approved: show matched rules, budget remaining, limit used.
- Denied: show failed checks and actionable hint.
- Pending: show why human approval is needed.
- `shotoku decision <id>` shows full explanation with all fields.

Expected commit:

```txt
feat: rich decision explanations
```

---

### Day 15 — Decision History Polish

Tasks:

- All `shotoku history` filters work cleanly.
- Summary line:

```txt
3 total · 1 approved · 1 denied · 1 pending
```

- `shotoku decision <id>` includes approval or denial history.
- Handle:

  - Empty history.
  - Invalid `--since` value.
  - Unknown actor.
  - Unknown status.

Expected commit:

```txt
feat: decision history queries
```

---

## Week 4 — MCP Server

### Day 16 — MCP Server Scaffold

Tasks:

- Read official MCP docs before coding.
- Decide whether to create `mcp/` package or keep MCP server inside `cli/`.
- Prefer a separate `mcp/` package only if it stays simple.
- Build basic MCP server skeleton.
- Server starts.
- Server responds to tool discovery.

Expected commit:

```txt
chore: MCP server scaffold
```

---

### Day 17 — `authorize_action` MCP Tool

Tool:

```txt
authorize_action
```

Input:

- actor
- action
- resource
- amount?
- context?

Behavior:

- Calls `authorize()` from core.
- Returns full `AuthorizeResponse`.

Expected commit:

```txt
feat: authorize_action MCP tool
```

---

### Day 18 — Decision Lookup MCP Tools

Tools:

```txt
get_decision
get_pending_approvals
```

Tasks:

- `get_decision` returns full decision detail.
- `get_pending_approvals` lists all pending decisions.
- Test both manually.

Expected commit:

```txt
feat: get_decision and get_pending_approvals MCP tools
```

---

### Day 19 — MCP Integration Guide

Tasks:

- Connect MCP server to Codex.
- Run full demo:

  - User asks if agent can purchase.
  - Codex calls `authorize_action`.
  - Shotoku returns decision.

- Verify tools in Codex MCP panel.
- Write `docs/mcp.md`.

Expected commit:

```txt
docs: MCP integration guide
```

---

### Day 20 — Block 2 Ship

Tasks:

- Screenshot `shotoku status` with pending approvals.
- Screenshot `shotoku approve dec_xxx`.
- Optional screen recording of Codex MCP panel.
- Post public progress update.

Expected commit:

```txt
chore: Block 2 complete
```

---

# BLOCK 3 — Integration + Brand Surface

## Goal

x402 demo works. TUI is polished. Landing page is live. Demo video exists.

---

## Week 5 — x402 + TUI

### Day 21 — x402 Research

Tasks:

- Read official x402 protocol docs.
- Run reference implementation.
- Understand:

  - HTTP 402 response.
  - Signing flow.
  - Settlement flow.
  - Where Shotoku authorizes before payment.

- No production code unless the flow is understood.
- Write notes in `docs/x402.md`.

Expected commit:

```txt
docs: x402 integration notes
```

---

### Day 22 — x402 Demo Integration

Implementation file:

```txt
core/src/x402.ts
```

Tasks:

- Build demo flow.
- Intercept x402 payment intent or equivalent.
- Call `authorize()` before allowing payment.
- If approved, execute demo payment path.
- Record result to ledger.
- Add `examples/x402-demo.ts`.
- Add `examples/policy-x402.yaml`.

Expected commit:

```txt
feat: x402 demo integration
```

---

### Day 23 — TUI Scaffold

Use Ink.

Suggested path:

```txt
cli/src/tui/
```

Tasks:

- Header bar.
- Main panel.
- Footer.
- Current time.
- Ledger count.
- Static mock data first.
- Keep visual design minimal and clean.

Expected commit:

```txt
feat: TUI scaffold
```

---

### Day 24 — TUI Pending Approvals Panel

Tasks:

- Wire TUI to real ledger data.
- Show pending approvals:

  - Actor.
  - Resource.
  - Amount.
  - Age.
  - Reasons.

- Use `▶` for selected item.
- Navigate with up/down.

Expected commit:

```txt
feat: TUI pending approvals panel
```

---

### Day 25 — TUI Keyboard Workflow

Shortcuts:

- `↑/↓` — navigate.
- `Enter` — approve selected.
- `d` — deny selected.
- `e` — expand full explanation.
- `h` — toggle history panel.
- `q` — quit.

Tasks:

- Wire approve/deny to real core functions.
- Auto-refresh after decision change.
- No crashes on empty state.

Expected commit:

```txt
feat: TUI keyboard approval workflow
```

---

## Week 6 — Landing Page + Video + Pitch Deck

### Day 26 — Landing Page Build

Goal:

A developer should understand Shotoku in under 10 seconds.

Hero message:

```txt
See, approve, and audit what your agents do.
```

Subtext:

```txt
Not another payment rail. Not a cloud policy engine.
Runs on your machine.
```

Tasks:

- Build shotoku.dev.
- Use simple one-page layout.
- Include install command.
- Include GitHub link.
- Include example CLI output.
- Keep design minimal.

Visual system:

- Background: `#0A0A0A`.
- Text: `#F5F5F0`.
- Borders: `#1F1F1F`.
- Green only for approved.
- Red only for denied.
- Yellow only for pending.
- No gradients.
- No glassmorphism.
- No heavy shadows.

Expected commit:

```txt
feat: landing page
```

---

### Day 27 — Landing Page Deploy

Tasks:

- Deploy to Vercel or Cloudflare Pages.
- Connect `shotoku.dev`.
- Verify load time under 1 second.
- Check mobile.
- Fix visual issues.

Expected commit:

```txt
chore: landing page deployed
```

---

### Day 28 — Demo Video

Tasks:

- Record 45-second demo.
- Terminal-focused.
- No voiceover required.
- Show:

  - `shotoku status`
  - approved decision
  - pending decision
  - approve command
  - history command

- Export:

  - GIF for README.
  - MP4 for YouTube.

Expected commit:

```txt
docs: demo video in README
```

---

### Day 29 — Pitch Deck

Tasks:

- Create Figma deck.
- 11 slides maximum.
- Use real CLI/TUI screenshots.
- Use same visual system as landing page.
- Export PDF.

Expected commit:

```txt
docs: pitch deck PDF
```

---

### Day 30 — Block 3 Ship

Tasks:

- Cut GitHub pre-release `v0.1.0-beta`.
- Screenshot TUI with pending approvals.
- Demo video link ready.
- shotoku.dev live.
- Post public progress update.

Expected commit:

```txt
chore: Block 3 complete
```

---

# BLOCK 4 — Launch

## Goal

v0.1.0 is public. Show HN is posted. Community is engaged. YC draft exists.

---

## Week 7 — Polish + Documentation

### Day 31 — Fresh Install Test

Tasks:

- Test in clean environment.
- Run:

```bash
npm install -g shotoku-cli
shotoku init
```

- Record every friction point.
- Fix friction.

Expected commit:

```txt
fix: fresh install issues
```

---

### Day 32 — Error Messages + Edge Cases

Tasks:

- Audit all CLI error paths.
- No raw stack traces for users.
- Every error says:

  - What happened.
  - How to fix it.

- Cover:

  - Missing policy file.
  - Corrupted ledger.
  - Invalid decision ID.
  - Duplicate approve.
  - Invalid amount.
  - Invalid status.

Expected commit:

```txt
fix: user-facing error messages
```

---

### Day 33 — Quickstart + API Docs

Tasks:

- Write `docs/quickstart.md`.
- Goal: install to first decision in under 5 minutes.
- Complete `docs/api.md`.
- Include type reference and examples.

Expected commit:

```txt
docs: quickstart and API reference
```

---

### Day 34 — Policies + x402 + MCP Docs

Tasks:

- Write `docs/policies.md`.
- Complete `docs/x402.md`.
- Complete `docs/mcp.md`.
- Ensure all examples are current.

Expected commit:

```txt
docs: policies, x402, and MCP guides
```

---

### Day 35 — README Final + CI Green

Tasks:

- README final version.
- Demo GIF at top.
- Install command.
- One-paragraph pitch.
- Links to docs.
- CI green.
- No debug output.
- TUI tested on common terminals.

Expected commit:

```txt
docs: README final version
```

---

## Week 8 — Launch

### Day 36 — npm Publish + Release

Tasks:

- Publish `shotoku-cli`.
- Publish `@shotoku/core`.
- Verify:

```bash
npm install -g shotoku-cli
```

- Cut GitHub Release `v0.1.0`.
- Tag `v0.1.0`.

Expected commit:

```txt
chore: release v0.1.0
```

---

### Day 37 — Show HN Day

Tasks:

- Post Show HN.
- Stay present in comments.
- Post x402 Discord launch message.
- Post Twitter/X launch thread.
- DM five targeted developers.
- Do not ship feature code today.
- Only fix critical bugs.

---

### Day 38 — Engage + Iterate

Tasks:

- Respond to every HN comment still coming in.
- Respond to Discord replies.
- Respond to Twitter replies.
- Triage feedback.
- Fix critical bugs only.

Expected commit:

```txt
fix: launch feedback issues
```

---

### Day 39 — YC Application Draft

Tasks:

- Draft YC application.
- Use real traction numbers.
- Answer:

  - What does your company do?
  - What problem are you solving?
  - Why now?
  - What traction do you have?

- Do not submit without review.

---

### Day 40 — Block 4 Complete

Tasks:

- Post launch retrospective.
- Post community update.
- Update GitHub Discussions.
- Final YC review pass.

Expected commit:

```txt
chore: Block 4 complete
```

---

# Launch Messaging

## Core Pitch

```txt
Shotoku is a local-first authorization layer for AI agents.

Agents can spend money, call APIs, execute code, and use tools autonomously. Shotoku gives developers a simple way to see, approve, deny, explain, and audit what agents do — running entirely on their machine.
```

## Show HN Title

```txt
Show HN: Shotoku – Local-first authorization layer for AI agents
```

## Short Social Post

```txt
Shipped Shotoku: a local-first authorization layer for AI agents.

Agents act. Shotoku checks first.

approve()
deny()
explain()
audit()

Runs on your machine. No cloud. No custody.
```

---

# Success Metrics

At launch, aim for:

- 50+ GitHub stars.
- 20+ npm installs.
- 10+ meaningful community replies.
- 5+ targeted developer conversations.
- 1+ person who actually tries it and tells you what breaks.

Real feedback matters more than vanity metrics.

---

# Final Rule

Build the smallest useful version of Shotoku.

Do not chase abstractions.

Do not build enterprise features.

Do not turn the product into a wallet.

Do not let x402 become the whole identity.

Build the local authorization layer first.

---
> Source: [shotoku-dev/shotoku](https://github.com/shotoku-dev/shotoku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
