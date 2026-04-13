## motivate-kids

> > This file applies to the **motivate-kids** repository.

# CLAUDE.md — Project Instructions

> This file applies to the **motivate-kids** repository.
> Global preferences live in `~/CLAUDE.md`.

---

## Branching & Release Strategy

**`main` is the production branch.** It is deployed to Vercel on every push.

- **Never commit directly to `main`.**
- All work — features, fixes, chores — must happen on a dedicated branch.
- Branch naming convention:
  - `feat/<short-description>` — new features
  - `fix/<short-description>` — bug fixes
  - `chore/<short-description>` — dependency updates, config, tooling
  - `refactor/<short-description>` — code restructuring with no behaviour change
  - `docs/<short-description>` — documentation only

### Workflow for Every Change

#### Phase 1: Analyze & Design (before writing any code)

When receiving a feature request or bug report:

1. **Deeply analyze the request** — don't take it at face value. Ask:
   - What is the user actually trying to accomplish?
   - What surfaces/pages are affected?
   - What existing code and patterns are involved?
   - What edge cases, failure modes, or UX pitfalls exist?
   - How does this interact with the rest of the system?
2. **Audit the live state** — read the current code for all affected surfaces. Identify gaps between what exists and what the ideal UX should be.
3. **Design the solution thoroughly** — define the exact changes needed per file, the user flow, and any data/type changes.
4. **Update `prd.md`** — add a structured Build Queue entry with acceptance criteria, technical notes, and test cases. Show the user and confirm before building.

#### Phase 2: Build (one feature at a time)

1. Branch off `main`: `git checkout -b feat/<name> main`
2. Implement **one feature only** following the one-feature-at-a-time rule (see `~/CLAUDE.md`)
3. Run lint + type-check + build — all must pass
4. Open a PR targeting `main`
5. Squash-merge the PR; delete the feature branch after merge
6. **Verify the feature works** before moving to the next one
7. Only then start the next feature — never batch multiple features into one PR

---

## Versioning & Changelog

This project follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`):

| Bump | When |
|------|------|
| `PATCH` (0.2.**x**) | Bug fixes, small UX tweaks, copy changes |
| `MINOR` (0.**x**.0) | New features, new pages, new user-facing capabilities |
| `MAJOR` (**x**.0.0) | Breaking changes, full rewrites, platform migrations |

### On every release

1. Update `version` in `package.json`
2. Add a new entry at the top of `CHANGELOG.md` (newest first)
3. Commit: `chore: release vX.Y.Z`
4. Tag: `git tag vX.Y.Z && git push origin vX.Y.Z`

### CHANGELOG.md format

Follow [Keep a Changelog](https://keepachangelog.com/en/1.0.0/):

```markdown
## [X.Y.Z] — YYYY-MM-DD
### Added
- ...
### Fixed
- ...
### Changed
- ...
### Removed
- ...
```

---

## Tech Stack Quick Reference

- **Framework:** Next.js 14 App Router (TypeScript)
- **Styling:** Tailwind CSS (warm theme defined in `tailwind.config.ts`)
- **State:** React Context + `localStorage` via `FamilyContext`
- **Auth:** Supabase Auth (email + OTP), app data is client-side only
- **Deployment:** Vercel (auto-deploys on push to `main`)
- **Testing:** Jest + React Testing Library

---

## Key Files

| Path | Purpose |
|------|---------|
| `app/parent/settings/page.tsx` | Tabbed Settings page (Members, Profiles, Badges, History) |
| `context/FamilyContext.tsx` | Global state + all data mutations |
| `lib/store.ts` | localStorage read/write helpers |
| `types/index.ts` | All domain types |
| `lib/i18n.ts` | Translations (en / zh) |
| `prd.md` | Product requirements & build queue |
| `CHANGELOG.md` | Version history |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Motivation-Labs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Motivation-Labs)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
