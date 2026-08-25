## hdb-analyser

> Commit, push, deploy, verify, and fix failures without pausing to ask. Loop until done.

# CLAUDE.md — working agreement for this repo

## Standing autonomy (effective permanently)

Commit, push, deploy, verify, and fix failures without pausing to ask. Loop until done.

**Only stop and check first for:**

- **Money.** Any spending, subscription, or account upgrade.
- **New external accounts.** Creating a Cloudflare / GitHub / Google / other third-party account.
- **Data deletion.** Destructive git ops (force push, hard reset, branch -D on shared branches), dropping DB tables, `rm -rf` outside `/tmp`.
- **Genuinely stuck.** When you need a decision or domain knowledge you can't derive from the code, docs, or observed behaviour.

Everything else — regular commits, pushes, PRs, workflow triggers, headless verification, secret rotation on personal-scope tokens — is pre-authorised. Report results at the end, not intent at the start.

## Working conventions (non-negotiable)

### 1. Evidence-first verification

Never call something "done" or "green" without exercising the code path and reading the output. Harness log, curl, headless Chrome, or the deployed URL — pick the one that proves the behaviour. "Should work" is not a status.

### 2. One fix = one commit

Each commit has one atomic reason. Related follow-ups that a change requires can ride along; unrelated cleanup gets its own commit. This makes `git bisect` and `git revert` actually useful.

### 3. Fresh workflow runs, never re-runs

If a workflow needs to re-execute, push a fresh commit (empty commit is fine). Do NOT use `gh run rerun` — a rerun replays with the OLD commit's tree, hiding races and stale-state bugs. Every workflow run must map 1:1 to a commit.

### 4. Don't push to main mid-workflow-run

Before every push, check `gh run list --branch main --status in_progress` (or the equivalent). If any workflow is running, wait for it to finish or coordinate the ordering. Otherwise a long-running workflow (e.g. `update-coords.yml` at ~30 min) can land its own push, and your push will be rejected non-fast-forward or the workflow's push will be rejected — either way is a stall.

### 5. Owner-review stops happen PRE-PUSH

This repo has no staging: `git push main` is a production deploy the moment Pages builds. Every owner-review checkpoint therefore runs against a **local build** with the full harness sweep, and the diff / evidence is shown BEFORE the push. Post-push review is inspection-of-fait-accompli, not a gate.

### 6. No fuzzy street matching in the valuation path

Exact translation table only. `canonStreet` (spelled-out canonical form) and `abbrevStreet` (HDB abbreviated form) are exact inverses. Any unknown or edge-case street name is either added to the exceptions map or fails loudly — it is NEVER matched by `includes()`, `startsWith`, or Levenshtein-style similarity. Field failure (postal 650118, 2026-07-15) traced to `normStr` covering 7 abbreviations while `canonStreet` covered 20, followed by an `includes()` partial-match fallback then a silent `towns[0]` fallback that built the analysis on Woodlands data. Fuzzy matching is how that happens.

### 7. Schools data must be manually verified against onemap.gov.sg/school after changes

Any change to `PRIMARY_SCHOOLS`, the Worker upstream contract, the schools rendering block, or `update-coords.yml` requires a manual spot-check against onemap.gov.sg/school for at least three known-answer postals before it ships:

- **560472** (Ang Mo Kio blk 472) — the exact MOE answer is 9 schools: Rosyth in the 1–2 km band, Townsville absent from either band.
- **123311** (Clementi Ave 4 blk 311C) — Henry Park Primary must appear in the 1–2 km band.
- **600268** (Toh Guan Rd blk 268) — Yuhua Primary and Princess Elizabeth Primary must appear in the 1 km band.

Automated tests catch shape drift; the manual OneMap parity check catches semantic drift. Both are required.

### 8. Pre-push harness runs against the COMMITTED tree, not the working tree

The gate proves the PUSHED code works, not that your uncommitted edits work. Before the pre-push sweep, `git status --short` MUST show clean of application-code / test-code / normalizer changes. If it isn't, either `git stash --include-untracked` first, `git add` the pending changes into the same commit, or re-run from a fresh clone. The harness passing while unstaged edits shadow the committed tree is the worst possible false-green — CI on a fresh checkout will crash on the first import, and prod is already live with the untested code.

Two incidents traced to skipping this:

- **`52105df` (2026-07-30)**: `git mv scripts/test-*.mjs scripts/tests/` staged the rename; the subsequent `./street-normalizers.mjs` → `../street-normalizers.mjs` import edits were not staged. Local sweep passed because the working tree had both changes. The committed content at `scripts/tests/test-*.mjs` still had the old `./` imports, which resolve to a nonexistent file at the new location — any fresh clone would crash on module resolution.
- **Earlier local-vs-prod count gap**: an unstaged edit lifted a local test count higher than the committed code could achieve; the "green" push landed a lower count on prod. Same root cause.

Procedure: after `git commit`, before `git push`, either `git stash --include-untracked && node scripts/tests/<harness>.mjs && git stash pop`, or (safer) `git worktree add /tmp/verify-$(date +%s) HEAD` and run the harness from that worktree. If either shows a change vs. what you tested from the working tree, the gate wasn't real.

Gotcha: a fresh `git worktree` doesn't inherit `scripts/node_modules/` (gitignored). Either `npm install --prefix scripts` in the worktree, or symlink: `ln -s $(git rev-parse --show-toplevel)/scripts/node_modules $WORKTREE/scripts/node_modules` before running the puppeteer tests. Offline tests (test-street-normalizers, test-prefix-pairs, test-mirror-consistency) don't need this.

## Two-source schools architecture (Phase 2, shipped 2026-07-12)

The schools section calls the Cloudflare Worker at `https://hdb-schools-parity.hdb-analyser.workers.dev` first. On any failure (3 s timeout, non-200 including 429, malformed JSON, empty-sentinel for a resolvable block), the client falls back instantly to the static straight-line list computed from the in-page `PRIMARY_SCHOOLS` constant.

- **Live path** — green badge: `Source: OneMap School Query · as of <Timestamp>`. HSD bands rendered verbatim.
- **Fallback path** — amber badge: `Approximate (straight-line) — live data unavailable`. Static list with straight-line distances. OneMap link preserved in both paths.

CSP `connect-src` in `index.html` must include the Worker origin.

Worker CORS is locked to `https://tengingofyu.github.io`. Local tests use `--disable-web-security`.

## References

- Worker source & contract: `worker/README.md`
- Historical context and open threads: `HANDOFF.md`
- Analytics setup: `ANALYTICS_SETUP.md`

---
> Source: [tengingofyu/hdb-analyser](https://github.com/tengingofyu/hdb-analyser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
