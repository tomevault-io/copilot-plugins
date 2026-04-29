## australianrates

> Hard rule: every chat with repo changes — commit, push, confirm deploy & CI, verify www.australianrates.com


# Every Chat: Commit, Push, Confirm Deploy, Verify Production (Hard Rule)

For **every** conversation about this project that changes the repository (code, config, assets, or work that implies a deploy), you **MUST** complete the full closeout below before claiming the task is done.

**Waivers:** Only if the user **explicitly** instructs you not to commit, push, deploy, or verify production for that task.

## 1. Commit, push, and PR (default)

- **Default:** follow **`git-pr-workflow-default.mdc`**: feature branch off `main`, commit, push branch, **open a PR to `main`**, merge when PR CI is green—then production deploys from `main`.
- **Exception:** direct `git push origin main` only if the user explicitly asked for a **`main` hotfix**.
- Do not leave uncommitted or unpushed work when the task implied shipping unless waived. If a PR cannot be merged from this environment, leave it open with the **PR URL** and state what remains after the user merges.

## 2. Confirm deployment and CI succeeded

A successful `git push` is **not** sufficient proof.

- Wait for **Cloudflare Pages** (site) and/or **Workers** (API, archive) deploys to finish when this change triggers them.
- Resolve **any CI or deploy failures** (GitHub Actions on the **PR** and/or `main`, pre-push checks, Cloudflare build/deploy errors): fix on the branch, recommit, repush, and re-check until green.
- Use available tooling (e.g. `gh run watch` / Actions UI, Cloudflare dashboard, or project scripts). If something cannot be confirmed from this environment, state what was verified and what could not be.

## 3. Confirm the intended result on production

- Verify on **https://www.australianrates.com**. Do not treat localhost alone as acceptance for user-visible or API behaviour tied to Cloudflare.
- From repo root, run the checks in **AGENTS.md** for the scope of the change: at minimum **`npm run test:homepage`** when the site or user-facing behaviour is involved; **`npm run test:api`** / **`npm run test:archive`** when the corresponding worker is involved; use **`npm run verify`** or **`npm run doctor:verify`** when full sign-off is appropriate.
- If a check fails: fix, commit, push, wait for deploy, verify again. Loop until pass or the user waives.

## 4. Final response evidence

For deploy- or production-impacting work, the final message must include what was run (commands), outcomes, and that production verification was performed—or an explicit note that it was waived or could not be run.

This rule overrides treating “pushed to the remote” as task completion without confirmed deploy and production verification.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yanniedog) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
