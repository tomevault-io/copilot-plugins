## kubit

> **Framework design/spec.** The skeleton is the ideal app; packages/core has types to make it compile. After spec is complete, we build the real framework.

# Kubit Framework Design Guide

## Current Phase

**Framework design/spec.** The skeleton is the ideal app; packages/core has types to make it compile. After spec is complete, we build the real framework.

## Source of Truth (in order)

1. `skeleton/*` - The ideal developer experience
2. `packages/core/index.d.ts` - The API contract (types)
3. `docs/SPEC.md` - The specification
4. `docs/SKELETON_APP.md` - What the skeleton should contain

## Critical Rules

- Skeleton defines the DX → types codify it → spec documents it
- Never add types without usage example in skeleton
- Never change API without updating: types + spec + skeleton + this file
- Keep skeleton minimal but complete (shows all patterns once)

## Primary Docs

| Doc | Purpose |
|-----|---------|
| `docs/SPEC.md` | Living specification with Decision Log |
| `docs/SKELETON_APP.md` | Skeleton structure spec |
| `docs/TEST_PLAN.md` | What to test (for future implementation) |
| `docs/BRAINDUMP.md` | Philosophy and research notes |
| `docs/rfcs/` | Design proposals |

## Validation

After any change:
- [ ] `tsc --noEmit` passes in skeleton/
- [ ] SPEC.md updated if API changed
- [ ] SKELETON_APP.md updated if structure changed
- [ ] This file updated if docs/ changed

## Quick Reference

**Language:** TypeScript with `experimentalDecorators`
**Views:** React 19 for SSR + hydration, Inertia-style protocol
**Patterns:** Class-based controllers/models/jobs, decorators for metadata, method chaining

**Module IDs:** `kubit`, `kubit:router`, `kubit:inertia`, `kubit:server`, `kubit:orm`, `kubit:db`, `kubit:queue`, `kubit:mail`, `kubit:hash`

## Skills

Proactive skills for framework design work:

| Skill | Trigger |
|-------|---------|
| `kubit-eliciting-spec` | Expanding subsystem specs |
| `kubit-designing-api` | Adding/modifying APIs |
| `kubit-expanding-skeleton` | Adding to skeleton/ |
| `kubit-proposing-change` | Non-trivial changes, RFCs |
| `kubit-validating-design` | After any change |
| `kubit-researching-prior-art` | Comparing framework approaches |

---
> Source: [aniftyco/kubit](https://github.com/aniftyco/kubit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
