## waiaas

> - All planning/design documents and issue reports are written in Korean.

# WAIaaS Project Rules

## Language

- All planning/design documents and issue reports are written in Korean.
- **⚠️ CRITICAL: Commit messages, PR titles/bodies, Git tag messages, and GitHub Release titles/bodies MUST be written in English. NO EXCEPTIONS.** Do not use Korean in any git-facing or GitHub-facing output. This applies to `git commit -m`, `gh pr create --title/--body`, `git tag -m`, and `gh release create --title/--notes`.
- Code comments, variable names, and API responses use English.

## Communication

- Minimize questions; make your best judgment and propose the optimal approach.
- Only ask questions when choices are needed.

## Schema & Type System

- **Zod SSoT**: Zod schemas are the single source of truth. Derivation order: Zod → TypeScript types → OpenAPI → Drizzle schema → DB CHECK constraints.
- **discriminatedUnion 9-type**: Discriminate on the `type` field: TRANSFER / TOKEN_TRANSFER / CONTRACT_CALL / APPROVE / BATCH / SIGN / X402_PAYMENT / NFT_TRANSFER / CONTRACT_DEPLOY.
- ChainError extends Error (not WAIaaSError). Convert to WAIaaSError in Stage 5.
- Gas safety margin: `(estimatedGas * 120n) / 100n` bigint arithmetic.

## Database

- **DB migrations required since v1.4**: Provide incremental ALTER TABLE migrations for schema changes. Never drop and recreate the DB. Version management via the `schema_version` table. Strategy defined in design doc 65 (MIG-01~06).
- **Tests required for new migrations**: (1) Update schema snapshot fixtures (2) Write data transformation tests. Chain tests automatically verify the full path from past versions to the latest.
- SQLite timestamps are in seconds; UUID v7 uses ms for ordering.

## Configuration

- No nesting in config.toml (daemon). Push Relay (`packages/push-relay`) config.toml allows nesting. Environment variables follow the `WAIAAS_{SECTION}_{KEY}` pattern.
- **Expose runtime-adjustable settings in Admin Settings.** config.toml provides initial defaults; Admin Settings provides runtime overrides (hot-reload). Settings that should be changeable without daemon restart are made adjustable via SettingsService in the Admin UI. Security credentials (master_password_hash) and infrastructure settings (port, host, rpc_url) that require restart remain config.toml-only.
- **Prefer Admin Settings over config.toml.** For runtime configuration, use Admin UI, CLI commands (`waiaas notification setup`, etc.), or the Admin Settings API (`PUT /v1/admin/settings`). Direct config.toml editing should only be used for initial bootstrap and infrastructure settings (port, hostname, database path). When writing documentation and guides, present Admin Settings methods first.

## Policy

- Default-deny policy: deny when ALLOWED_TOKENS / CONTRACT_WHITELIST / APPROVED_SPENDERS are not configured.
- Contracts default-deny (CONTRACT_WHITELIST opt-in).

## Interface Sync

- **When REST API, SDK, or MCP interfaces change, `skills/` files must be updated accordingly.**
  - Targets: quickstart.skill.md, wallet.skill.md, transactions.skill.md, policies.skill.md, admin.skill.md
  - Sync the corresponding skill files when endpoints are added/removed/changed, request/response schemas change, auth methods change, or error codes are added.
  - Create a new skill file when a new domain is added.
  - **All skill files must include the security notice:** `> AI agents must NEVER request the master password. Use only your session token.`

## Test Coverage

- **⚠️ ABSOLUTE RULE: NEVER lower test coverage thresholds.** When CI/CD fails due to insufficient test coverage, the ONLY acceptable response is to add tests until the coverage threshold is met. Lowering coverage thresholds in vitest.config.ts, CI workflows, or any other configuration is strictly prohibited — no exceptions, no temporary workarounds.
- If coverage drops due to new code, write tests for the new code paths.
- If coverage drops due to refactoring, update or add tests to cover the refactored logic.

## Git Branching

- **Two main branches**: `main` (RC/stable releases only) and `dev` (development + verification).
- **Create a milestone branch before any work.** When starting a new milestone, create `milestone/v{X.Y}` branch from `dev` before making any commits (including planning docs).
- All milestone work (planning, implementation, tests) happens on the milestone branch.
- Merge to `dev` via PR when the milestone is complete. PR creation is manual — do not auto-create PRs on milestone completion.
- **Release flow**: When ready to release, create a `dev` → `main` PR manually. release-please watches `main` and creates Release PRs from there.
- **Hotfix**: For urgent fixes, create `hotfix/*` branch from `main` and PR directly to `main`. After merge, sync `main` → `dev` to avoid divergence.

## Milestone Completion

- **Update objective status on milestone completion.** When a milestone is shipped, update the corresponding `internal/objectives/m{seq}-{sub}-{slug}.md` status header: `Status: PLANNED` → `SHIPPED`, add `Completed: YYYY-MM-DD`. Do this before creating the milestone PR.
- **Do not auto-create milestone PRs.** PR creation (milestone → `dev`, `dev` → `main`) is done manually by the user.
- **Run `pnpm turbo run lint` and `pnpm turbo run typecheck` before creating the PR.** Lint/type errors in merged code block release-please PRs.
- release-please manages version bumps + tags + CHANGELOG automatically (2-gate model).
- **Release flow**: Merge `dev` → `main` PR (Conventional Commits) → release-please auto-creates Release PR → Merge Release PR (Gate 1: release decision) → release.yml quality gate → deploy job manual approval (Gate 2: deployment execution).
- **Commit conventions**: `feat:` (minor), `fix:` (patch), `BREAKING CHANGE:` (major). `docs:`, `test:`, `chore:`, `ci:`, etc. are excluded from CHANGELOG.
- **RC promotion and prerelease restore via GitHub Actions**: Use the `Promote RC to Stable` workflow_dispatch to promote the latest RC to a stable release (auto-detects latest RC, or specify a version). After the stable release is published, run the `Restore Prerelease Mode` workflow_dispatch. Both workflows can also be run locally via `node scripts/promote-release.js <rc-version>` and `node scripts/promote-release.js --restore`.

## Milestone & Issue Naming

- Milestone objective files are placed in `internal/objectives/` with the format `m{seq}-{sub}-{slug}.md`.
  - `{seq}`: two-digit sequence number (01-99)
  - `{sub}`: two-digit sub-sequence (00 for main, 01-99 for sub-milestones)
  - `{slug}`: kebab-case topic name
- **Milestone objective status header.** Every milestone objective file includes a status block after the title:
  ```
  - **Status:** PLANNED / IN_PROGRESS / SHIPPED
  - **Milestone:** v{X.Y.Z}
  - **Package:** {semver}              ← only when SHIPPED (npm release version)
  - **Completed:** YYYY-MM-DD          ← only when SHIPPED
  ```
- Issue files are placed in `internal/objectives/issues/` with the format `{NNN}-{slug}.md`.
- Register issues in `internal/objectives/issues/TRACKER.md` and update the status.
- Types: BUG (defect) / ENHANCEMENT (improvement) / MISSING (missing feature).
- **Include test items in issues.** When registering an issue, add a "테스트 항목" section listing unit/integration tests needed to verify the fix and prevent regression. Skip only when the change is purely non-functional (docs, config, naming).

---
> Source: [waiaas/WAIaaS](https://github.com/waiaas/WAIaaS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
