## project-rultes

> These rules define **non-negotiable architectural principles** for this Next.js codebase. They are written for experienced developers and architects. They assume fluency in React, modern JavaScript/TypeScript, and web platform fundamentals.


---

## alwaysApply: true

# Next.js Architecture Rules (Senior / Architect Level)

These rules define **non-negotiable architectural principles** for this Next.js codebase. They are written for experienced developers and architects. They assume fluency in React, modern JavaScript/TypeScript, and web platform fundamentals.

The goal is **long-term scalability, correctness, and operational clarity**, not developer convenience or tutorial-style patterns.

---

## 1. Architectural Philosophy

- This is a **product-grade system**, not a demo or experiment.
- Prefer **boring, explicit, and predictable** designs over clever abstractions.
- Optimize for:

  - Change isolation
  - Testability
  - Observability
  - Failure containment

Avoid patterns that trade short-term velocity for long-term entropy.

---

## 2. App Router Is the Default

- Use the **Next.js App Router** exclusively unless explicitly justified otherwise.
- Pages Router is considered legacy and must not be introduced.

Key implications:

- Server Components are the default
- Client Components are opt-in and treated as a boundary

---

## 3. Server vs Client Boundary Discipline

### Server Components

- Contain:

  - Data fetching
  - Authorization decisions
  - Feature gating
  - Layout composition

- Must remain **pure and side-effect free** (no browser APIs).

### Client Components

- Used only for:

  - User interaction
  - Browser-only APIs
  - Local ephemeral state

- Must be **explicitly marked** with `"use client"`
- Should never fetch core domain data directly

> If a component does not require browser APIs, it must not be a Client Component.

---

## 4. Data Access Architecture

- Data fetching is a **server concern**.
- Never call databases, internal services, or secrets from Client Components.

Recommended structure:

- `src/domain/` → business logic, entities, invariants
- `src/infrastructure/` → DB, external APIs, adapters
- `src/application/` → orchestration, use-cases
- `src/app/` → delivery layer (routing, layouts, RSC composition)

React components must not contain domain rules.

---

## 5. API Routes Are Integration Boundaries

- Route handlers (`app/api/**/route.ts`) are:

  - Integration points
  - Not business logic containers

Rules:

- Validate input explicitly
- Call application-layer use cases
- Return normalized error contracts

Never embed business rules directly in route handlers.

---

## 6. State Management Strategy

- Prefer **server state** over client state.
- URL state is preferred over global client stores.

Client state libraries (Zustand, Redux, etc.):

- Require architectural justification
- Must be scoped, not global by default

If state must survive reloads, question the architecture.

---

## 7. Side Effects and Mutations

- All mutations must be:

  - Explicit
  - Auditable
  - Idempotent where possible

Use:

- Server Actions for mutations
- Clear command-style naming (`createOrder`, `cancelSubscription`)

Avoid implicit side effects hidden inside components.

---

## 8. Error Handling and Failure Modes

- Errors are part of system behavior, not edge cases.

Rules:

- Use `error.tsx` and `not-found.tsx` intentionally
- Never swallow errors
- Distinguish:

  - User errors
  - System errors
  - Integration failures

Design for partial failure.

---

## 9. Performance Is Architectural, Not Cosmetic

- Rendering strategy (SSR, SSG, streaming) must be intentional.
- Do not rely on memoization as a primary optimization.

Rules:

- Measure before optimizing
- Avoid unnecessary client-side hydration
- Treat bundle size as a first-class constraint

---

## 10. Testing Philosophy

- Test behavior, not implementation.
- Unit tests target domain and application layers.
- Integration tests validate Next.js routing and data boundaries.

Do not snapshot-test React trees without strong justification.

---

## 11. Dependency Governance

- Every new dependency is an architectural decision.

Rules:

- Prefer platform and framework primitives
- Avoid overlapping libraries
- Justify any state, data, or form library

Smaller dependency graphs are easier to reason about and secure.

---

## 12. Code Review Bar

Every change must answer:

- What boundary does this respect or introduce?
- What future change does this make easier?
- What failure mode does this handle explicitly?

If these answers are unclear, the change is not ready.

---

## Final Principle

> Architecture exists to make **the right things easy and the wrong things hard**.

If this codebase feels permissive, unclear, or magical, the architecture has failed.

---
> Source: [JaimeenMakavana/agent-vis](https://github.com/JaimeenMakavana/agent-vis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
