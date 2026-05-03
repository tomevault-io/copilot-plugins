## 08-security

> Security practices – OpenSSF Scorecard, CodeQL, and dependency auditing


# Security

## OpenSSF Scorecard – Token Permissions

Every workflow file **must** declare `permissions: read-all` (or explicit minimal permissions) at the top level. This is required to pass the Scorecard Token-Permissions check.

```yaml
# ✅ GOOD – top-level declaration
permissions: read-all

jobs:
  build:
    permissions:
      contents: write  # job overrides only what it needs
```

```yaml
# ❌ BAD – no top-level permissions block
jobs:
  build:
    steps: ...
```

## Reusable Workflows (`workflow_call`)

When a job calls a reusable workflow, GitHub enforces that the **caller's job-level permissions** must be a superset of the **called workflow's top-level `permissions` declaration**.

If the called workflow has `permissions: read-all`, the calling job must explicitly list all read permissions it covers, plus any write permissions needed:

```yaml
# release.yml – calling job
publish:
  uses: ./.github/workflows/publish.yml
  permissions:
    contents: write        # write needed by the called workflow's job
    actions: read
    attestations: read
    checks: read
    deployments: read
    discussions: read
    id-token: read
    issues: read
    models: read
    packages: read
    pages: read
    pull-requests: read
    repository-projects: read
    security-events: read
    statuses: read
```

Never reduce the caller's permissions hoping the called workflow will "just work" — GitHub validates this at parse time, not at runtime.

## CodeQL

- CodeQL analysis runs on `push` to `main` and on `pull_request`. Do not remove or skip these triggers.
- The `security-events: write` permission is required by the CodeQL action to upload SARIF results.
- Keep the language matrix in `codeql-analysis.yml` up to date with the languages in the project.

## Dependency Auditing

- Run `pnpm audit --audit-level=high` in CI. Fail the build on high or critical vulnerabilities.
- Fix transitive dependency vulnerabilities using `pnpm.overrides` in `package.json` — do not pin direct deps to workaround transitive issues:

```json
"pnpm": {
  "overrides": {
    "minimatch": ">=10.2.1"
  }
}
```

- After adding an override, run `pnpm install` and re-run `pnpm audit` to confirm the vulnerability is resolved.

---
> Source: [timoa/workflow-editor](https://github.com/timoa/workflow-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
