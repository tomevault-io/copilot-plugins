## calyflow-app

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

<!-- BEGIN:knowledge-base -->
# Read the knowledge base first

At the **start of every session**, review all articles in `knowledge-base/*` —
they capture how the app works (`requirements.md`) and the engineering conventions
(`standards.md`). They're the fastest way to get oriented before reading code, and
keep features consistent across sessions. Keep them up to date when you learn
something durable.
<!-- END:knowledge-base -->

<!-- BEGIN:mobile-ui -->
# Every UI change must account for mobile

Calyflow is used heavily on phones, and we've shipped several regressions where a change looked right on desktop but broke on small screens. **Any change that touches UI — layout, spacing, components, styling, new views — must be considered, and ideally tested, on mobile screen sizes**, not just desktop.

- Assume mobile-first: verify the change works at narrow widths (≈375px, the common phone viewport) as well as on desktop.
- Watch for the usual regressions: horizontal overflow/scroll, content clipped or hidden behind fixed/sticky elements, tap targets too small or overlapping, text and buttons that no longer wrap or fit, and desktop-only hover interactions with no touch equivalent.
- Prefer testing it for real: use the `run-app` skill and check the affected screen at a mobile viewport (browser responsive/device mode). If you genuinely can't test it, say so and call out the mobile risk explicitly in the PR.
<!-- END:mobile-ui -->

<!-- BEGIN:worktree-path-discipline -->
# When working in a git worktree, edit inside it

If the session is in a worktree (cwd contains `/.claude/worktrees/<name>/`), the original repo checkout still exists at the repo root with the same file tree. **Every Read/Edit/Write must target a path under the worktree**, i.e. one containing `/.claude/worktrees/<name>/`.

`Explore`/`Plan` subagents, `Grep`, and `Glob` report paths rooted at the **original repo** (e.g. `/Users/you/Projects/repo/lib/foo.ts`). Don't edit those verbatim — translate to the worktree (`…/repo/.claude/worktrees/<name>/lib/foo.ts`) or use worktree-relative paths. After your first edit, confirm `git status` shows it in the worktree and the repo root stays clean. Misplaced edits land on `main` and the worktree branch stays empty.
<!-- END:worktree-path-discipline -->

<!-- BEGIN:pr-closes-issues -->
# Close issues from PRs with a keyword

When a PR resolves a GitHub issue, put a **closing keyword** in the PR body (or a commit message): `Closes #123`, `Fixes #123`, or `Resolves #123`. GitHub then auto-closes the issue when the PR merges to `main`. Plain references like "Implements #123" or "Addresses #123" only *link* the issue — they do **not** close it. Use one keyword per issue the PR resolves.
<!-- END:pr-closes-issues -->

<!-- BEGIN:pr-automerge -->
# PRs should auto-merge

Once a PR is open, enable auto-merge so it lands as soon as checks pass — don't wait around to merge it by hand. Default to squash + delete the branch: `gh pr merge --squash --auto --delete-branch`. `main` has **required status checks** (`build`, `gitleaks`, `dependency-review`), so auto-merge holds the PR until CI is green before it lands — enabling it right after you open the PR is safe, it will not merge early. (Merging to `main` then triggers the prod deploy pipeline.)

Always pass `--delete-branch`. A **squash** merge rewrites your commits into one new commit on `main`, so the branch's original commits keep different SHAs — GitHub's `main...your-branch` compare then shows them as "ahead" and the branch looks like it has unpublished code even though the content is fully merged. Deleting the branch on merge removes that confusion (and the stale copy of any migration file — see below). To confirm nothing is actually unmerged, diff the *content*, not commits: `git diff origin/main origin/<branch> -- <path>` per file (empty = identical); a non-empty `main...branch` compare after a squash is cosmetic.
<!-- END:pr-automerge -->

<!-- BEGIN:migration-numbering -->
# Number migrations against the LATEST main, and keep them idempotent

Migration files are `supabase/migrations/NNNN_name.sql`, and the deploy records each `NNNN` as a **unique** version in prod's `schema_migrations`. Two open branches that each grab the same next number both pass CI in isolation, then the **second one to reach prod fails** the deploy with `duplicate key value violates unique constraint "schema_migrations_pkey" (version)=(NNNN)` — and because the migration step aborts, it blocks the *entire* prod deploy, not just that feature. This has bitten us repeatedly.

Before you add **and again right before you merge** a migration:

- Pick the number from the **latest `origin/main`**, never your branch's base: `git fetch origin main && git ls-tree --name-only origin/main -- supabase/migrations | sort | tail -3`. Use `max + 1`.
- If `origin/main` gained a migration at your number while your PR was open (another migration PR merged first), **renumber yours to the new `max + 1` before merging**. A pure `git mv` is enough — no code references the filename.
- Make every migration **idempotent** — `add column if not exists`, `create table if not exists`, `drop ... if exists`, guarded `insert`/backfills — so renumbering, re-running, or a retried deploy is always safe.

When two migration PRs are in flight at once, whoever merges second owns the renumber. If a deploy is already red with the duplicate-key error, the fix is: renumber the losing migration to `max + 1` on `main` (small PR) and let the deploy re-run.
<!-- END:migration-numbering -->


<!-- BEGIN:security-review -->
# Run a security review before opening a PR

Before opening any PR, run the `/security-review` skill on the branch. It audits
the pending diff for vulnerabilities (injection, auth bypass, secret leakage,
XSS, unsafe input handling). Treat findings like failing tests: fix them, or
explicitly justify them in the PR description before merging. This is
**mandatory** for changes touching public/unauthenticated surfaces, auth,
tokens, webhooks, or LLM prompts handling untrusted input — and cheap enough to
run on everything else.
<!-- END:security-review -->

---
> Source: [Calyflow/calyflow-app](https://github.com/Calyflow/calyflow-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
