## geeksharearchive

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

## GitHub submission workflow

When the user asks to commit or push changes to GitHub, work from the local `main` checkout and push directly to `origin/main`. Do not create a task branch or Pull Request unless the user explicitly requests one.

Before starting work, fetch `origin` and fast-forward the local `main` branch with `git pull --ff-only origin main`. Before pushing, confirm that `main` still contains the latest `origin/main`; if the histories diverge or a conflict occurs, stop and resolve it safely instead of force-pushing. Run the relevant checks before every push, stage only files that belong to the current task, and never force-push or delete `main`.

In the Codex desktop app, prefer the Local checkout for this repository. If a task starts in a managed worktree, hand it off to Local before committing instead of creating a branch in the worktree.

## Deployment requests

Whenever the user asks to deploy, publish, or go live, treat the request as authorization for the complete production-release workflow below and perform it in this order:

1. Run the relevant checks and production build, fixing in-scope failures until they pass.
2. Deploy the current changes to Cloudflare with the repository's production configuration and verify the live deployment.
3. Commit only the changes belonging to the current task and push that commit to the intended GitHub branch.
4. Report the release as stable only after the checks, Cloudflare deployment, live verification, and GitHub push have all succeeded.

A deployment request is not complete if either Cloudflare deployment or GitHub push fails. Continue resolving in-scope failures when possible and report any real blocker instead of describing an unverified release as stable. "Stable" describes a verified release; do not create or move a Git tag named `stable` unless the user explicitly requests that tag.

---
> Source: [igeekshare/geekshareArchive](https://github.com/igeekshare/geekshareArchive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
