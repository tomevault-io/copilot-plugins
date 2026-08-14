## youknowme

> The official product name is YouKnowMe.

# AGENTS.md

## Project

The official product name is YouKnowMe.

This repository is the home for the production YouKnowMe project.

## Hard Boundary

The POC in 'POC/' should be ignored and never mentioned. Do not try to maintain the POC code it will be deleted.

## Corpus Change Boundary

Do not make direct changes in the YouKnowMe corpus unless Roger explicitly asks for that exact
manual corpus edit. When Roger asks about corpus feedback, missing knowledge, stale served content,
or why a submitted corpus-change request has not appeared yet, treat it as a request to inspect or
improve the Curator/agent workflow so the agent makes the corpus change. The normal answer should be
a YouKnowMe feature fix, bug fix, diagnostic, or operational status lookup, not bypassing Curator by
editing the corpus yourself.

## Documentation

Place the PRD and supporting documentation in `docs/`.

Prefer small, explicit documents with stable names over burying project requirements in chat history.

## Tooling

Use `mise` for local tool versions and `uv` for Python dependency management.

Common commands:

```bash
mise install
mise run sync
mise run lint
mise run test
```

Before opening or updating a PR, run both `mise run lint` and the full `mise run test` suite unless
there is a concrete blocker. Focused tests are useful while iterating, but they are not enough for
presubmit.

## Test Organization

Keep test files focused by behavior or domain. Do not grow a single test module into a broad
catch-all file; split it before it reaches 1,000 lines. The lint task enforces this limit for files
under `tests/`.

## Agent Delivery Standard

Give Roger PRs ready to review. Do not just return once changes are written and put the ball in
Roger's court. The goal is nearly always to produce a PR that is well tested, including staging
validation when applicable, before sending it for review.

For repo-mutating tasks, finish with:

1. A committed change on a feature branch.
1. A pushed branch.
1. A non-draft PR in Ready-for-review state.
1. Validation summary (at minimum `mise run lint` and `mise run test`, or an explicit blocker with rationale).
1. A clean task worktree or a documented blocker.

Never, ever send Roger a PR in Draft mode. It makes no sense for this repository's agent workflow.

If a task cannot produce a ready PR, do not leave uncommitted local edits. Report the exact dirty file
set and the concrete blocker so Roger can decide next action.

Code review is the human gate for acceptance; agents do not merge their own PRs unless Roger explicitly
instructs an exception.

## Deployment

Production deploys happen automatically through GitHub Actions. The workflow is
`.github/workflows/deploy-production.yml` and it is triggered after YKM CI passes on a push to
`main`.

The production deploy runs Ansible from the `grubbyhacker/vps-ops` repository over SSH to
`hermes-vps` (`srv1656293.hstgr.cloud`). The deploy user is `github-deployer`.

Agents must never SSH directly to `hermes-vps` to make operational changes. All production changes
must go through the GitHub Actions deployment pipeline.

Agents have standing authorization to perform read-only diagnostics on `hermes-vps` when needed to
understand production state. Acceptable read-only checks include commands such as `docker ps`,
`docker inspect`, `docker logs`, file listing, and targeted reads of logs, run reports, and intake
metadata. Do not edit files, restart services, change containers, or run deploy/maintenance commands
on `hermes-vps` unless explicitly authorized for that specific operation.

The GitHub production environment gate requires approval before the deploy proceeds.

## Local Staging

Staging runs on localhost with Docker from the `grubbyhacker/vps-ops` repository:

```bash
mise run deploy:staging -- youknowme
```

Staging uses `YKM_AUTH_MODE=local` and requires the local auth header:

```text
X-YKM-Local-Secret: staging-local-secret
```

When staging is running, the service is available at `http://127.0.0.1:8765`.

The staging index must be rsynced from production before staging can start. The expected local path
is `~/staging/youknowme/data/index-current`.

## Docker Image

CI builds and publishes the Docker image on every push to `main`.

Images are multi-arch for `linux/amd64` and `linux/arm64`, and are published to
`ghcr.io/grubbyhacker/youknowme` with `sha-` tags.

## Health Endpoints

Use `/livez` for liveness. It returns `200` as soon as uvicorn starts.

Use `/readyz` for readiness and health checks. It returns `200` only after the index is fully
loaded.

## Auth Modes

`public` mode requires a Cloudflare Access JWT and is used in production.

`local` mode requires the `X-YKM-Local-Secret` header and is used in staging and development.

## Git Workflow

`main` is protected. Always create or switch to a feature branch before making implementation changes.

Do not make changes on `main` and then waste time porting them to a feature branch later.

Use the standard flow: feature branch, pull request, CI passing, then merge. CI must pass for lint
and tests before merge. The Docker image is published by CI on pushes to `main`.

Do not push directly to `main`.

After merge, clean up local and remote state:

1. `git fetch --prune`
1. Checkout `main` and fast-forward to `origin/main`.
1. Delete the merged local branch.
1. Delete the remote feature branch if needed.
1. Remove the task worktree.
1. Verify clean status.

Delete old feature branches after they are merged or no longer needed.

When a YouKnowMe task requires changes in another repository, create an isolated worktree for that
repository from a clean head before editing. Do not reuse a dirty checkout or a branch where another
agent or human may be working.

## Secrets

Do not commit `.env`, `.env.*`, tunnel tokens, Access credentials, VPS credentials, or generated private runtime files.

---
> Source: [grubbyhacker/youknowme](https://github.com/grubbyhacker/youknowme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
