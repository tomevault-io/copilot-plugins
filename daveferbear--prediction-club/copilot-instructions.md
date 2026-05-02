## prediction-club

> This repo favors **simple, deterministic code** with **explicit state transitions**, **minimal useEffect**, and **clear separation of concerns**. Avoid controller components, implicit workflows, and boolean-heavy state.

# agents.md — Engineering Patterns & Conventions

This repo favors **simple, deterministic code** with **explicit state transitions**, **minimal useEffect**, and **clear separation of concerns**. Avoid controller components, implicit workflows, and boolean-heavy state.

The goal is code that is:

- easy to reason about
- hard to break accidentally
- straightforward to refactor

---

## Core Principles

- Pages render UI; they do not orchestrate workflows
- Prefer explicit state machines over ad-hoc booleans
- Avoid `useEffect` unless reacting to external state
- Separate background data refresh from user actions
- Encode invariants in types, not conditionals
- Prefer SWR for client-side data fetching

---

## 1) Pages Are Views, Not Workflow Engines

Do not implement setup flows, polling, or orchestration directly in page components.

Prefer:

- page.tsx: render + wire actions
- custom hooks: orchestration, state machines, side effects
- feature views: pure UI

Pattern:

```ts
const setup = useProfileSetup();
return <ProfileSetupView setup={setup} />;
```

---

## 2) Use a State Machine (Reducer) Instead of Many Booleans

Avoid accumulating flags like:
safeReady, approvalsReady, credsSaved, funded, checked, etc.

Prefer:

- a discriminated union for user-driven state
- derived checks for progress/UI

```ts
type SetupState =
  | { tag: 'idle' }
  | { tag: 'connecting' }
  | { tag: 'signingIn' }
  | { tag: 'derivingCreds' }
  | { tag: 'deployingSafe' }
  | { tag: 'approving' }
  | { tag: 'savingCreds' }
  | { tag: 'error'; message: string };
```

---

## 3) Data Fetching: Use SWR

We standardize on **SWR** for client-side data fetching.

**Rules**

- Use SWR for reads, polling, and background refresh
- Do not use raw fetch/axios directly in components
- Do not use React Query in this repo
- Wrap fetchers in small hooks (`useBalances`, `useSafeStatus`, etc.)

**Patterns**

- Use `enabled` via conditional keys
- Use `refreshInterval` for polling
- Use `mutate` for optimistic or post-action refresh

```ts
const { data, error, isLoading } = useSWR(enabled ? ['/api/resource', id] : null, fetcher, {
  refreshInterval: 15000,
});
```

SWR replaces ad-hoc polling, retry timers, and effect-based refresh loops.

---

## 4) Avoid useEffect for Orchestration

useEffect should not act as:

- a workflow scheduler
- a polling loop
- a retry engine
- a background state machine

Do not:

- manage timers, refs, or rate limits inside effects
- mix multiple responsibilities in one effect

Prefer:

- SWR with conditional keys and refreshInterval
- small single-purpose hooks

---

## 5) Separate Background Queries from User Actions

Background refresh must not hijack UI state.

Rules:

- Background queries expose data, isLoading, error
- User actions manage their own busy and error
- Background failures never set global error state

UI:

- action errors = blocking
- background errors = non-blocking warnings

---

## 6) Prefer Explicit Guards + Commands Over ensureX()

Avoid functions that:

- check prerequisites
- trigger side effects
- sometimes return false, sometimes await
- mutate UI state implicitly

Prefer:

- pure guards: needsConnect, needsChain, needsAuth
- explicit commands: connectWallet, switchChain, signIn
- buttons disabled until prerequisites are met where possible

---

## 7) Address Handling

- Parse addresses once at boundaries
- Internally use Address | null
- Do not throw from render or effects

Pattern:

```ts
parseAddress(value): Address | null
assertAddress(value): Address
```

---

## 8) Isolate Adapters (viem / ethers / etc.)

Do not cast or bridge providers inside UI code.

Rules:

- adapters live in lib/
- UI consumes clean abstractions only
- provider internals are never referenced in components

---

## 9) Status Must Be Honest and Narrow

Avoid a single overloaded status mutated by:

- user actions
- background checks

Prefer:

- reducer state.tag for user actions
- per-action busy flags if needed
- background queries expose loading independently

---

## 10) Split Big Logic into Small Hooks

Instead of one large effect that:

- checks deployment
- checks approvals
- fetches balances
- retries
- persists state

Use:

- useCredsStatus
- useSafeDeployed
- useBalances
- useApprovals

Each hook should be:

- single-purpose
- independently testable
- safe to reuse

---

## 11) UI Component Layering (shadcn + CVA)

We use **standard shadcn/ui primitives with CVA**. Keep a strict separation.

ui/ — primitives only:

- shadcn components
- CVA variants
- presentation-only props
- no domain concepts
- no hooks or logic

components/ — everything else:

- composed UI
- one-off or feature-specific components
- domain/workflow-aware components
- components that use hooks or logic

Rule of thumb:
If a component imports multiple primitives, or contains conditional logic, or references app/domain concepts — it does not belong in ui/.

Goal: ui/ stays boring and stable; app UI lives above it.

---

## 12) Avoid Imperative UI in Handlers

- Avoid window.prompt, alert, etc.
- Prefer controlled dialogs and components
- Handlers should be thin and predictable

---

## 13) Error Handling

- Distinguish action errors vs background errors
- Prefer typed errors where possible
- Silent failures only for explicitly best-effort operations
- Use gated logging helpers, not scattered console.log

---

## 14) Recommended Folder Structure

- app/\*\*/page.tsx — minimal wiring
- components/<feature>/\* — composed UI
- hooks/use<Thing>.ts — orchestration or SWR hooks
- lib/\*\* — adapters, clients, utilities
- ui/\*\* — shadcn primitives only

---

## 15) PR Review Checklist

A change is good if it:

- removes or shrinks useEffect
- replaces manual fetching with SWR
- reduces boolean state
- separates background refresh from user actions
- moves orchestration into hooks/reducers
- keeps ui/ primitive-only
- makes adding/reordering steps safe

---

## 16) Database Migration Policy (Strict)

When changing Prisma schema or SQL migrations:

- Do not run destructive migration commands automatically.
- Do not run `prisma migrate reset` unless the user explicitly asks.
- Do not run `prisma db push --force-reset` unless the user explicitly asks.
- Do not run `prisma migrate dev` against shared/remote databases unless the user explicitly asks.

Default workflow for agents:

1. Update `packages/db/prisma/schema.prisma`.
2. Generate or author the migration SQL diff in `packages/db/prisma/migrations/<timestamp>_<name>/migration.sql`.
3. Hand migration execution back to the user with exact commands.

Example diff-generation command (do not apply automatically):

```bash
cd packages/db
npx prisma migrate diff \
  --from-migrations prisma/migrations \
  --to-schema-datamodel prisma/schema.prisma \
  --script > prisma/migrations/<timestamp>_<name>/migration.sql
```

Required handoff behavior:

- Assume the user runs migrations.
- Provide a copy-paste command block for the user to apply migration(s).
- If migration history is drifted, provide a non-destructive recovery path (`migrate resolve`) instead of reset.

In short: agents prepare schema + migration artifacts; users execute DB migrations.

---

## Notes for Agents

When refactoring or adding flows:

1. Extract orchestration into a hook.
2. Replace polling effects with SWR.
3. Replace booleans with a state machine + derived checks.
4. Keep pages declarative.
5. Keep ui/ boring.

---
> Source: [DaveFerBear/prediction-club](https://github.com/DaveFerBear/prediction-club) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
