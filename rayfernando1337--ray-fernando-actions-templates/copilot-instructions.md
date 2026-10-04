## ray-fernando-actions-templates

> Primary briefing for coding agents. Read this before copying any workflow.

# AGENTS.md — adopt these CI templates in a consumer repo

Primary briefing for coding agents. Read this before copying any workflow.
Humans: [README.md](./README.md) and [docs/adopt-checklist.md](./docs/adopt-checklist.md).

## 1. Purpose & how to use this repo

This repository holds **sanitized GitHub Actions workflow templates** and
short ops docs. It is not CI for a product app.

Your job when adopting:

1. Copy selected files from `workflows/` into
   `YOUR_ORG/YOUR_REPO/.github/workflows/` (drop the `.example` suffix only
   when the consumer project has the matching scripts and secrets).
2. Replace every placeholder: project name, cache paths, npm/cargo scripts,
   path filters, labels, runner labels, model ids, and doc links.
3. Wire secrets/vars (names in §3; details in [docs/secrets.md](./docs/secrets.md)).
4. Confirm self-hosted macOS expectations (§4) before marking `check` required.
5. Do **not** assume the consumer monorepo layout matches any example path
   in comments (`apps/…`, `scripts/…`, etc.). Those are illustrations.

If a template step references a script that does not exist in the consumer
repo, delete or rewrite the step — do not invent a no-op that always passes.

## 2. Workflow catalog — when to adopt each

| Adopt when… | Template | Priority |
|---|---|---|
| You need a hard verify gate on Apple silicon / macOS-native tooling | `workflows/check.yml` | Core |
| You have no self-hosted runner and need a hosted verify gate | `workflows/check-hosted.yml` | Core (hosted) |
| The app has React/TSX and you want PR-diff issue comments without blocking day one. Optional escalate job past a new-issue threshold | `workflows/react-doctor.yml` | Core if React |
| You want automated Cursor CLI review + optional autofix on same-repo PRs | `workflows/cursor-review.yml` | Optional, cost-aware |
| You want agent review with zero push risk (start here, graduate to `cursor-review.yml`) | `workflows/cursor-review-readonly.yml` | Optional, cost-aware |
| Jobs run on a small self-hosted pool and can sit `queued` with no `timeout-minutes` signal | `workflows/queue-stall-alarm.yml` | Strongly recommended with self-hosted |
| Media/toolchain behavior depends on an ffmpeg major-version floor you must prove on Linux | `workflows/ffmpeg-floor.yml.example` | Only if you have a floor matrix script |
| Maintainers want `@droid` in issues/PRs to run Factory Droid | `workflows/droid.yml.example` | Optional |

Adoption order that usually works: `check` → `queue-stall-alarm` (if
self-hosted) → `react-doctor` → `cursor-review` → optional examples.
When you have no self-hosted runner, start with `check-hosted` instead of
`check`. Prefer `cursor-review-readonly` over `cursor-review` until you
need autofix.

## 3. Required secrets / vars (names only)

Configure in the **consumer** repo (or org). Full notes:
[docs/secrets.md](./docs/secrets.md).

### Almost always

| Name | Kind | Used by |
|---|---|---|
| `GITHUB_TOKEN` | Automatic | All workflows (default permissions still matter) |

### Cursor review pipeline

| Name | Kind | Used by |
|---|---|---|
| `CURSOR_API_KEY` | Secret | `cursor-review.yml` |
| `CURSOR_PUSH_TOKEN` | Secret (fine-grained PAT, contents:write on same repo) | Autofix push that must retrigger workflows |
| `CURSOR_REVIEW_MODEL` | Secret **or** Variable (optional override) | Review agent model id |
| `CURSOR_AUTOFIX_MODEL` | Secret **or** Variable (optional override) | Autofix agent model id |

### Readonly review pipeline (no push)

`CURSOR_PUSH_TOKEN` is not needed for `cursor-review-readonly.yml`.

| Name | Kind | Used by |
|---|---|---|
| `CURSOR_AGENT_REVIEWS` | Variable (kill switch) | Every agent job |
| `REACT_DOCTOR_ESCALATION_THRESHOLD` | Variable (optional, default 10) | `react-doctor.yml` escalate |
| `CURSOR_ESCALATION_MODEL` | Variable (optional) | `react-doctor.yml` escalate |

### Factory Droid

| Name | Kind | Used by |
|---|---|---|
| `FACTORY_API_KEY` | Secret | `droid.yml.example` |

### Project-specific (only if your `check` needs them)

| Name | Kind | Notes |
|---|---|---|
| Registry / cloud tokens | Secret | Only if install or E2E hits private registries |
| `VERIFY_SKIP_E2E` | Env set **in the workflow**, not a repo secret | Pattern: skip heavy E2E on `push`, run on PR/nightly |

Never commit key material. Never put PATs in `vars.*`.

## 4. Self-hosted macOS runner expectations (generic)

Details: [docs/self-hosted-macos-runner.md](./docs/self-hosted-macos-runner.md).

Agents configuring a consumer repo should assume:

- Runner labels match the YAML, typically `[self-hosted, macOS]`. Add extra
  labels only if every machine that should take the job has them.
- Toolchain on `PATH` for non-interactive jobs: Node (version your lockfile
  needs), Rust/cargo if applicable, Homebrew tools your gate asserts
  (e.g. `ffmpeg` / `ffprobe`), Xcode CLT or full Xcode if Swift/iOS steps exist.
- **Persistent cargo/target cache lives outside the checkout workspace**, e.g.
  `$HOME/.cache/<project>-cargo-target-$RUNNER_NAME`.
  `actions/checkout` clean/reset must not delete it.
- **One cache directory per `RUNNER_NAME`.** Concurrent runners must not share
  a target dir if any job can `rm -rf` the cache (wipe races).
- Caches are **not relocatable** after a Tauri/build-script bake of absolute
  paths: rename base or runner → delete old cache, accept one cold build.
- Prefer `CARGO_INCREMENTAL=0` in CI when the cache is persistent (incremental
  grows disk for little gain when each crate compiles once per run).
- Launchd/headless runners often cannot use the login keychain; Cursor CLI
  should use an in-memory credential store plus `CURSOR_API_KEY`.
- Ambient `~/.gitconfig` `insteadOf` rewrites can force SSH clones and ignore
  `GITHUB_TOKEN` — nullify global/system git config in the job env (§5).

Hosted Ubuntu remains required for the queue stall alarm so the detector does
not queue behind the outage it is meant to report.

## 5. Reusable patterns to copy

Copy the *idea*; rewrite paths and script names.

### Cargo / build cache outside checkout

```yaml
env:
  # Prefer computing under $HOME in a step — do not hardcode /Users/<name>/...
  CARGO_INCREMENTAL: "0"
# In a step, after the runner is known:
#   CARGO_TARGET_BASE="${CARGO_TARGET_BASE:-$HOME/.cache/<project>-cargo-target}"
#   CARGO_TARGET_DIR="$CARGO_TARGET_BASE-$RUNNER_NAME"
#   mkdir -p "$CARGO_TARGET_DIR"
#   echo "CARGO_TARGET_DIR=$CARGO_TARGET_DIR" >> "$GITHUB_ENV"
```

Optional `workflow_dispatch` input `wipe_cargo_cache` for one cold run.
Optional nightly schedule that starts from an empty cache to exercise the
fresh-worktree path.

### GIT_CONFIG nullification

```yaml
env:
  GIT_CONFIG_GLOBAL: /dev/null
  GIT_CONFIG_SYSTEM: /dev/null
```

Use on every job that `actions/checkout`s (and especially any job that
pushes). Prevents runner-user `url.*.insteadOf` from rewriting HTTPS to SSH.

### Draft skip + `ready_for_review`

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]
jobs:
  check:
    if: github.event_name != 'pull_request' || github.event.pull_request.draft == false
```

Skipping drafts without listening for `ready_for_review` leaves a PR ungated
after undraft until someone pushes again.

### `VERIFY_SKIP_E2E` (or equivalent)

Pattern: on `push` to the default branch, set an env flag that your verify
script honors to skip long E2E; keep full E2E on pull_request and on a
scheduled backstop. Name the flag whatever the consumer script already
documents — do not add a flag the script ignores.

### Fail-open local hooks vs hard CI

Local agent hooks (format, clippy, React Doctor on dirty files) should **fail
open** on missing toolchains/timeouts so they never block an editor turn.
CI (`check`, optional blocking React Doctor) remains the source of truth.
Do not weaken CI to match hook softness.

### Hosted queue stall alarm

- `runs-on: ubuntu-latest` (or another **hosted** label).
- `timeout-minutes` on a job does **not** bound queue time.
- Threshold should be above normal single-runner backlog (multiple PRs × job
  duration), with a “runner actually busy” check so legitimate queues do not
  page.
- If hosted minutes are unavailable, disable `schedule:` and keep
  `workflow_dispatch` so a permanently red cron does not train people to
  ignore failures. Re-enable cron when hosted runners work again.

### Cursor review cost / safety controls

- Always run on `opened` / `reopened` / `ready_for_review`; gate
  `synchronize` behind a label (e.g. `ci-review`) if push volume is high.
- Skip fork PRs on `pull_request` (no secrets, cannot push); do **not** switch
  to `pull_request_target` to “fix” forks.
- Pass attacker-influenced context (`title`, `body`, branch, login) via `env:`
  and quoted shell vars — not inline `${{ }}` inside `run:` blocks.
- Tag autofix commits (e.g. `[cursor-autofix]`) and skip a second autofix
  round when HEAD already has the tag.
- `concurrency.cancel-in-progress: false` for the review job if autofix push
  must not be cancelled mid-flight.

### React Doctor advisory → blocking

Ship advisory (`blocking` unset / `none`) first. Graduate to `error` only when
the team trusts the signal. Use `fetch-depth: 0` so PR runs diff against the
merge base.

### Pin privileged third-party actions

For Droid (or any action that receives API keys + write permissions), pin
`uses: org/action@<commit-sha>` and review the diff when bumping.

### Diff-scoped format gate when main is red

When the default branch fails a repo-wide format check, gate only the files
changed since the merge base. See
[docs/layered-review-pipeline.md](./docs/layered-review-pipeline.md)
for why, and for when to widen the gate.

### Slop gate

Copy `scripts/slop-check.sh` to `.github/scripts/` in the consumer repo.
Run the same command locally and in CI. See
[docs/layered-review-pipeline.md](./docs/layered-review-pipeline.md)
for the rule split.

## 6. Step-by-step adoption checklist

1. **Inventory** consumer verify entrypoint (`npm run check`, `cargo test`,
   etc.), React apps, ffmpeg/version floors, and whether a self-hosted macOS
   runner already exists.
2. **Copy** `check.yml`; rewrite `runs-on`, path filters, dependency assert
   steps, and the final verify command.
3. **Copy** `check-hosted.yml` instead of `check.yml` when there is no
   self-hosted runner. Copy `scripts/slop-check.sh` to `.github/scripts/`
   and adjust the generated-code exclude.
4. **Set** persistent cache base under `$HOME/.cache/<project>-…` and export
   `CARGO_TARGET_DIR` per `RUNNER_NAME` if Rust is in the gate.
5. **Add** `GIT_CONFIG_GLOBAL` / `GIT_CONFIG_SYSTEM` nullification to jobs
   that check out or push.
6. **Add** draft `if:` + `ready_for_review` to every PR-gated workflow you
   install.
7. **Install** `queue-stall-alarm.yml` on hosted Ubuntu if you use
   self-hosted; tune threshold; enable cron only if hosted jobs actually run.
8. **Install** `react-doctor.yml` if applicable; start advisory; set
   `directory` / `project` for monorepos.
9. **Install** `cursor-review-readonly.yml` when you want agent review with
   zero push. Set `CURSOR_AGENT_REVIEWS=true` only when ready to spend.
10. **Install** `cursor-review.yml` only after `CURSOR_API_KEY` (and push PAT
    if autofix) exist; set models; confirm keychain/`AGENT_CLI_CREDENTIAL_STORE`
    behavior on the runner.
11. **Add** `ffmpeg-floor.yml` from the `.example` only if
    `scripts/<your-floor-matrix>.mjs` (or equivalent) exists; set path filters
    to **your** sources.
12. **Add** `droid.yml` from the `.example` only with `FACTORY_API_KEY` and a
    pinned action SHA.
13. **Branch protection:** require `check` (and any other hard gates); keep
    React Doctor advisory until ready; do not require Cursor review on forks.
14. **Smoke test:** draft PR (must skip) → mark ready (must run) → confirm
    queue alarm job is hosted → confirm no secret values appear in logs.

## 7. What MUST be customized per project

- Repository slug references and any hardcoded `YOUR_ORG/YOUR_REPO` leftovers.
- `runs-on` labels and who owns the runner fleet.
- Absolute cache paths (`$HOME/.cache/<project>-cargo-target`).
- `paths` / `paths-ignore` filters (docs, wiki, canvas, agent dirs, etc.).
- Package manager commands and verify script names.
- E2E skip env var name and semantics (`VERIFY_SKIP_E2E` is an example only).
- ffmpeg floor script path, cache key files, and PR path filters.
- React Doctor `directory` / `project` / blocking mode.
- Cursor models, label name for synchronize gating, commit tag for autofix.
- Escalation threshold (`REACT_DOCTOR_ESCALATION_THRESHOLD`) and escalation model.
- Generated-code exclude in `slop-check.sh`.
- Reviewer bot logins in the readonly review prompt.
- Concurrency group names (avoid collisions across workflows).
- Permissions blocks (`contents`, `pull-requests`, `actions`, `id-token`).
- Timeout values (cold Rust vs warm; alarm threshold vs single-runner backlog).
- Docs links inside workflow comments (point at **consumer** architecture docs).

## 8. What NOT to copy blindly

- Any real hostname, username, `$HOME` for another machine, SSH `insteadOf`
  snippet, or healthcheck UUID from a private deployment.
- Billing-incident narrative as if it applied to the consumer — keep the
  *structural* lesson (hosted alarm; disable red forever-crons) without
  copying account-specific history.
- `pull_request_target` “so forks get Cursor review”.
- Sharing one cargo target dir across runners that wipe caches.
- Moving a warm target dir after build scripts baked absolute paths.
- Making the queue stall alarm `runs-on: [self-hosted, macOS]`.
- Turning React Doctor blocking on day one in a dirty tree.
- Mutable `@main` refs for actions that receive write tokens or API keys.
- Assuming `timeout-minutes` detects a wedged runner listener while jobs sit
  queued.
- Committing `.env`, API keys, or PATs “for the template to work”.
- Copying path filters that skip workflow files or the verify script itself.
- Requiring agent checks in branch protection.
- Running the escalation model on every PR instead of behind the threshold.

When unsure, prefer a smaller adopted surface (`check` + queue alarm) over
installing every template at once.

---
> Source: [RayFernando1337/ray-fernando-actions-templates](https://github.com/RayFernando1337/ray-fernando-actions-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
