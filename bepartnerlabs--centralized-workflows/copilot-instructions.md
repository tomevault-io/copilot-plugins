## centralized-workflows

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Purpose

Reusable GitHub Actions workflows for all BePartnerLabs projects. Called via:

```yaml
uses: BePartnerLabs/centralized-workflows/.github/workflows/<name>.yml@v1
```

## Workflows

| File | Purpose |
|---|---|
| `setup.yml` | Install & cache pnpm dependencies — outputs `cache-key` and `store-path` |
| `lint.yml` | Run `pnpm lint` |
| `test.yml` | Run unit tests via configurable `test-command` input |
| `migrate-payload.yml` | Run `pnpm payload migrate` against `environment`'s `PAYLOAD_SECRET`/`DATABASE_URL` secrets — Payload projects only |
| `deploy-vercel.yml` | `vercel deploy` (optionally `--prod`) — generic, no Payload dependency, callable directly by non-Payload projects |
| `release-drafter.yml` | Auto-draft releases from PR titles |
| `labeler.yml` | Auto-label PRs |
| `tag-sync.yml` | Moves the major version tag (e.g. `v1`) when a release is published |

## Examples

`examples/` has copy-pasteable caller workflows for each common setup — copy the file into `<your-repo>/.github/workflows/` and adjust as needed. They live outside `.github/workflows/` in *this* repo so GitHub doesn't try to run them here.

| File | Use case |
|---|---|
| `examples/ci.yml` | Lint + test on push/PR to `main` |
| `examples/deploy-payload-vercel.yml` | Payload + Vercel — preview on push to `main`, production on release |
| `examples/deploy-vercel-only.yml` | Vercel only, no DB/migrations |
| `examples/release.yml` | PR auto-labeling + draft release notes |

## Release & tagging flow

**Never move version tags manually.** The flow is:

1. Merge changes to `main` — release-drafter auto-updates the draft release
2. When ready to ship: publish the draft release on GitHub (`gh release edit <tag> --draft=false`)
3. `tag-sync.yml` fires on `release: released` and moves the `v1` major tag to the new commit automatically

All consumer repos pin to `@v1` — they pick up changes as soon as the tag moves.

---
> Source: [BePartnerLabs/centralized-workflows](https://github.com/BePartnerLabs/centralized-workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
