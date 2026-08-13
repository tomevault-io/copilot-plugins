## constants-policy

> Constants-only policy - all copy, labels, tunables, and error messages live in shared/constants.ts


# Constants policy (mandatory before any UI/copy/tunable change)

## Read this first

Before adding or changing any user-facing string, label, error message, timeout, path segment, MIME type, or other magic value:

1. Open **`shared/constants.ts`** (the single source of truth for the whole repo).
2. Reuse an existing export, or add a new named export with JSDoc.
3. Import via `@shared/constants` **or** the package re-export `@/lib/constants` (landing and extension both re-export the shared module).
4. Never leave a new literal in call sites.

## Canonical home

| Role | Path |
|------|------|
| **Source of truth** | `shared/constants.ts` |
| Theme type helpers | `shared/themeTypes.ts` |
| Landing re-export | `src/lib/constants.ts` → `export * from "@shared/constants"` |
| Extension re-export | `extension/src/lib/constants.ts` → `export * from "@shared/constants"` |

Do not create `copy.ts`, `strings.ts`, `messages.ts`, or a second constants module. Do not put product constants only in `src/lib/` or `extension/src/lib/` - always edit `shared/constants.ts`.

## Allowed exceptions (not `shared/constants.ts`)

| Content | Home |
|---------|------|
| FAQ / llms.txt body copy | `src/lib/ai-content.json` |
| SEO builders (robots, sitemap, JSON-LD helpers) | `src/lib/seo.ts` (+ `scripts/generate-sitemap.mjs` for static emit) |
| Build-only Vite / manifest knobs | Colocate in the config file with a short comment |
| CSS layout tokens | CSS variables in stylesheets (not TS constants) |

## Must go in `shared/constants.ts`

- UI labels, placeholders, button/CTA text, empty states, dialog titles
- Error / permission / unsupported-browser messages shown to users
- Debounce intervals, limits, widths, playback speeds, bar counts
- Storage key names, OPFS directory names, MIME types, file prefixes
- Theme picker labels/defaults, marketing paths/URLs

## Forbidden

```typescript
// ❌ Hardcoded in a component
<button>Show more</button>
throw new Error("Microphone access was denied…");
setTimeout(save, 250);

// ✅ From shared constants (via package re-export)
import { SIDEBAR_RECENT_SHOW_MORE_LABEL, EDITOR_SAVE_DEBOUNCE_MS, MICROPHONE_DENIED_MESSAGE } from "@/lib/constants";
// or: import { … } from "@shared/constants";
```

## Agent checklist (every change)

- [ ] Grepped `shared/constants.ts` for an existing name before adding a duplicate
- [ ] New exports have JSDoc describing purpose/units
- [ ] Call sites import constants - no new user-facing string literals in TSX/TS (except test fixtures)
- [ ] Did not edit only the thin re-export files in `src/lib/constants.ts` / `extension/src/lib/constants.ts`

---
> Source: [aryancodes-tech/my-memos](https://github.com/aryancodes-tech/my-memos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
