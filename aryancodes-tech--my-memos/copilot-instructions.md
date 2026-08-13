## ai-operating-model

> How AI agents should operate on this repo - analysis, verification, and communication standards


# AI Operating Model

## Before any modification

1. **Classify** the task (extension / landing / storage / editor / CI / landing-seo)
2. **Read** nearest existing implementation - same directory, same layer
3. **Trace data flow** - where state is read, written, persisted
4. **State invariants** you must preserve (see `storage-invariants.mdc`)
5. **Constants** - any new copy, label, error message, or tunable → `shared/constants.ts` only (see `constants-policy.mdc`)
6. **Capabilities** - do not document or market schema-only / unwired APIs as features (`AGENTS.md` §2.5)

## Analysis output (for non-trivial tasks)

Summarize before coding:

- Which surface(s) affected
- Files likely to change
- Whether `shared/constants.ts` will gain/reuse exports
- Storage/settings impact
- Tests to add or run

## Verification is mandatory

- **Run commands** - do not only suggest them
- **Report results** - pass/fail with relevant output
- UI changes → `npm run dev` or `npm run dev:web` when feasible
- Logic changes → `npm run test` for affected modules
- Landing SEO → `npm run test -- tests/landing/lib/seo.test.ts` + `npm run generate:seo` + curl `/llms.txt`
- Pre-PR → `npm run ci`

## Code quality bar

```typescript
// ✅ Extension empty check
import { len } from "@/lib/text";
if (len(page.parent_id ?? "") === 0) { ... }

// ❌ Avoid for strings
if (!page.parent_id) { ... }
```

```typescript
// ✅ New tunable / copy - define in constants.ts, then import
/** Debounce for editor saves, in milliseconds. */
export const EDITOR_SAVE_DEBOUNCE_MS = 250;
// in component:
setTimeout(save, EDITOR_SAVE_DEBOUNCE_MS);

// ❌ Magic number or hardcoded user string in component
setTimeout(save, 250);
<button>Show more</button>
```

## Communication

- Use code citations: ```startLine:endLine:filepath
- Proportional responses - simple fix ≠ architecture essay
- State blockers explicitly with evidence (logs, test output)
- Do not claim fixed without verification

## FDE-relevant practices demonstrated in this repo

- Multi-surface delivery from one extension codebase
- Platform abstraction without duplicating product logic
- Local-first storage with explicit invariants and tests
- AI-maintained agent docs (`AGENTS.md`, `SKILLS.md`, scoped rules)
- CI parity between local (`npm run ci`) and GitHub Actions

## Escalate to user

- Data-destructive migrations
- New backend / auth / cloud sync scope
- Repeated CI failure after two distinct fix attempts
- Ambiguous UX for landing scroll or major architecture rewrites

---
> Source: [aryancodes-tech/my-memos](https://github.com/aryancodes-tech/my-memos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
