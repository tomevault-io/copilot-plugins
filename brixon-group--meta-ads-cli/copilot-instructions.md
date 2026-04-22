## meta-ads-cli

> This file explains how to work safely in `CLI-meta-ads` and which files must stay in sync when the CLI changes.

# CLAUDE.md

This file explains how to work safely in `CLI-meta-ads` and which files must stay in sync when the CLI changes.

## Start here

Read these files first:

1. `README.md`
2. `docs/TECHNICAL.md`
3. `AI_CONTEXT.md`
4. `src/cli/build-cli.ts`
5. `src/domain/meta-ads-service.ts`
6. `AGENTS.md`

## Repository overview

### Runtime and entrypoints

- `src/index.ts`: CLI entrypoint
- `src/cli/build-cli.ts`: command registration
- `src/cli/context.ts`: resolved config and command context
- `src/cli/action.ts`: command execution wrapper, output, exit codes

### Command layer

- `src/commands/accounts.ts`
- `src/commands/assets.ts`
- `src/commands/performance.ts`
- `src/commands/campaigns.ts`
- `src/commands/adsets.ts`
- `src/commands/creatives.ts`
- `src/commands/ads.ts`
- `src/commands/launch.ts`
- `src/commands/anomalies.ts`
- `src/commands/pixel.ts`
- `src/commands/capi.ts`
- `src/commands/audiences.ts`
- `src/commands/report.ts`
- `src/commands/auth.ts`
- `src/commands/diagnostics.ts`

### Provider and domain layer

- `src/domain/meta-ads-service.ts`
- `src/domain/launch-service.ts`
- `src/domain/approval-service.ts`
- `src/domain/analytics.ts`
- `src/domain/account-scope.ts`
- `src/client/meta-api-client.ts`
- `src/client/meta-discovery.ts`

### Support layers

- `src/config/file-config.ts`
- `src/config/types.ts`
- `src/auth/constants.ts`
- `src/auth/guards.ts`
- `src/auth/login.ts`
- `src/validators/common.ts`
- `src/validators/create-spec.ts`
- `src/validators/launch-spec.ts`
- `src/output/render.ts`
- `src/utils/errors.ts`
- `src/utils/date-range.ts`
- `src/utils/ids.ts`
- `src/utils/currency.ts`
- `src/utils/security.ts`
- `src/types.ts`

### Tests and docs

- `tests/unit/cli.test.ts`
- `tests/unit/meta-api-client.test.ts`
- `tests/unit/analytics.test.ts`
- `tests/unit/auth-login.test.ts`
- `tests/unit/approval-service.test.ts`
- `tests/unit/currency.test.ts`
- `tests/unit/create-commands.test.ts`
- `tests/unit/launch.test.ts`
- `tests/unit/date-range.test.ts`
- `README.md`
- `docs/TECHNICAL.md`
- `AI_CONTEXT.md`
- `AGENTS.md`
- `skills/meta-cli-operator/SKILL.md`
- `skills/meta-cli-operator/references/update-matrix.md`

## Operational rules

- Keep reads and writes separate.
- Keep the default human auth path on Facebook Login for Business user tokens; do not regress to a shared-system-user-only assumption.
- `auth login` may prompt interactively for a missing Meta app ID or Facebook Login for Business configuration ID; keep non-interactive runs failing clearly when those values are not preconfigured.
- `auth login --code-flow` is an additive escape hatch for configurations that require the authorization-code grant; keep it requiring `META_APP_SECRET` and do not make it the default human path.
- Preserve draft-first behavior for mutations.
- Do not remove `--apply` gating.
- Do not bypass approval-gated flows for `campaigns pause`, `campaigns enable`, or threshold-triggered `campaigns budget`.
- Prefer changing the smallest coherent layer instead of scattering provider logic into command handlers.
- Keep `META_APPROVAL_WEBHOOK` HTTPS-only.
- Keep `--debug` output sanitized; do not print access tokens, appsecret proofs, or raw secret-bearing URLs.
- Only explicit multi-account commands may accept `--account all`; single-account commands must reject it.
- Do not silently truncate paginated provider results at the client boundary.
- Keep runtime config sources explicit: CLI flags, actual process env vars, local config, then defaults. Do not add hidden host-local env-file loading.

## Current create/launch constraints

- Keep create specs `PAUSED`-only unless the repo explicitly broadens that contract.
- `campaigns create` must keep non-CBO campaign creation provider-safe by sending explicit ad-set budget sharing state when no campaign budget is used.
- `adsets create` must preserve the bid-strategy defaulting behavior: without `bidAmount`, default to `LOWEST_COST_WITHOUT_CAP`; with `bidAmount`, default to `LOWEST_COST_WITH_BID_CAP`.
- Advantage+ Audience targeting must be validated or normalized before Meta sees it; current repo behavior normalizes `age_min` down to `25` and `age_max` up to `65` when `targeting.targeting_automation.advantage_audience = 1`.
- `creatives create` and `launch` both support `formats.feed4x5|square1x1|story9x16`, while the legacy surface `platformCustomizations.instagram` remains for backward compatibility; keep docs and tests aligned if either surface changes.

## Files that must be updated when behavior changes

### If command behavior changes

Update:

- `src/commands/*.ts`
- `src/cli/build-cli.ts` if registration or command shape changed
- relevant tests in `tests/unit/`
- `README.md`
- `docs/TECHNICAL.md`
- `AI_CONTEXT.md`
- `AGENTS.md`
- `skills/meta-cli-operator/SKILL.md` if the operator workflow changed

### If provider/API behavior changes

Update:

- `src/domain/meta-ads-service.ts`
- `src/client/meta-api-client.ts`
- `src/client/meta-discovery.ts` if discovery is affected
- relevant tests
- `docs/TECHNICAL.md`
- `README.md` if users/operators see changed behavior

### If config or env vars change

Update:

- `src/config/file-config.ts`
- `src/auth/login.ts` if the login bootstrap or redirect handling changes
- `src/cli/context.ts` if runtime context changed
- tests
- `README.md`
- `docs/TECHNICAL.md`
- `AI_CONTEXT.md`
- `AGENTS.md`

### If approval or safety logic changes

Update:

- `src/domain/approval-service.ts`
- affected command files
- tests
- `README.md`
- `docs/TECHNICAL.md`
- `AI_CONTEXT.md`
- `AGENTS.md`
- `skills/meta-cli-operator/references/update-matrix.md`

### If output or exit codes change

Update:

- `src/output/render.ts`
- `src/utils/errors.ts`
- tests
- `README.md`
- `docs/TECHNICAL.md`
- `AI_CONTEXT.md`

## Verification after non-trivial changes

```bash
npm run lint
npm run typecheck
npm run test
npm run build
npm pack --dry-run
node dist/index.js --help
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Brixon-Group) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
