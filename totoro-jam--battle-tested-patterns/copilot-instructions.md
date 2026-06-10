## battle-tested-patterns

> Production-proven programming patterns extracted from React, Linux, Go, Chromium and more.

# Battle Tested Patterns

## Project Overview

Production-proven programming patterns extracted from React, Linux, Go, Chromium and more.
Each pattern has precise source links, multi-language implementations, and runnable exercises.

- **Docs**: VitePress site deployed to GitHub Pages
- **Exercises**: Vitest (TS) + cargo test (Rust) + go test (Go) + pytest (Python)
- **Monorepo**: pnpm workspace (`docs/`, `exercises/`)

## Quality Red Lines

1. **Never fabricate source links** — leave a `TODO` rather than invent a URL
2. **Never claim "project X uses this pattern"** without a verifiable link
3. **Code must be runnable** — no pseudocode
4. **Multi-language implementations must be idiomatic** — not line-by-line translation
5. **Exercise tests must pass** — verify with `pnpm test:exercises` / `cargo test` / `go test ./...` / `pytest`

## Source Link Standard

```
✅ https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberFlags.js#L18-L22
✅ https://github.com/torvalds/linux/blob/master/include/uapi/linux/stat.h#L28-L35
❌ Directory-level links (not precise enough)
❌ Feature branch links (may be deleted)
❌ Links not verified with `curl -I`
```

Always target `main`/`master` branch. Convert to commit SHA permalinks before release using `tsx scripts/convert-to-sha-links.ts`.

## Pattern File Template

Every pattern in `docs/patterns/` **must** contain these sections:

1. `# Pattern: [Name]`
2. `## One Liner` — ≤ 30 English words
3. `## Core Idea` — with ASCII diagram or Excalidraw
4. `## Production Proof` — table with ≥ 2 projects, precise GitHub URLs to line numbers
5. `## Implementation` — subsections per language (TypeScript required + ≥ 1 other: Rust/Go/Python/C)
6. `## Exercises` — links to exercise files, ≥ 2 test cases, difficulty labeled
7. `## When to Use` — applicable scenarios
8. `## When NOT to Use` — limitations and alternatives
9. `## More Production Uses` — bullet list with repo links
10. `## Related Patterns` — table linking to ≥ 2 related patterns
11. `## Challenge Questions` — 3-4 scenario-based Q&A using `::: details` syntax

## Commit Convention

Use [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` — new pattern or language implementation
- `fix:` — fix incorrect content or broken link
- `docs:` — documentation improvements
- `ci:` — workflow changes
- `chore:` — tooling, config, dependencies
- `test:` — exercise/test changes

## Commands

```bash
pnpm dev          # Start VitePress dev server
pnpm build        # Build docs site
pnpm test         # Run ALL tests (exercises + docs components)
pnpm test:exercises # Run TypeScript exercises only (Vitest)
pnpm test:docs    # Run Vue component tests only
pnpm check        # Run all checks (lint + typecheck + test + verify-code + verify-mermaid)
pnpm lint         # Lint markdown files
pnpm typecheck    # TypeScript strict type check
pnpm verify-code  # Verify all code blocks in patterns compile (TS/Rust/Go/Python)
pnpm verify-mermaid # Validate Mermaid diagram syntax
pnpm verify-links # Verify all source URLs are alive (requires network)
cd exercises/rust && cargo test    # Run Rust exercises
cd exercises/go && go test ./...   # Run Go exercises
cd exercises/python && pytest      # Run Python exercises
```

## Skills

AI agent skills live in `.claude/skills/`. Invoke with `/<name>`:

- `/new-pattern` — guided workflow to create a pattern (validates topic → verifies sources → writes doc → implements → tests)
- `/verify-source` — check all production proof links for HTTP status, format, and content accuracy
- `/diagnose` — structured debugging loop: reproduce → isolate → hypothesize → fix → verify

## Git Guardrails

`.claude/settings.json` configures a PreToolUse hook that blocks destructive git operations (`push`, `reset --hard`, `clean -f`, `branch -D`). The agent can commit but must not push — push manually after review.

## SOPs

Standard Operating Procedures live in `.sop/`. Read them before:

- Adding a new pattern → `.sop/01-new-pattern.md`
- Verifying source links → `.sop/02-verify-source.md`
- Writing multi-language code → `.sop/03-multi-lang-impl.md`
- Designing exercises → `.sop/04-exercise-design.md`
- Reviewing PRs → `.sop/05-pr-review.md`
- Fixing broken links → `.sop/06-broken-link-fix.md`
- CI/CD verification → `.sop/07-ci-cd-verification.md`
- Release process → `.sop/08-release.md`
- Vue component build pitfalls → `.sop/09-vue-build-pitfalls.md`
- Interactive viz component audit → `.sop/10-viz-component-audit.md`
- Difficulty classification criteria → `.sop/11-difficulty-classification.md`
- Related Patterns bidirectionality → `.sop/12-related-patterns-audit.md`
- Content quality audit methodology → `.sop/13-content-quality-audit.md`

## Node Version

This project requires Node.js LTS (v22). See `.nvmrc`. Run `nvm use` before working.

---
> Source: [Totoro-jam/battle-tested-patterns](https://github.com/Totoro-jam/battle-tested-patterns) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
