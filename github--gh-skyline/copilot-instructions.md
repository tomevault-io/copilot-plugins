## gh-skyline

> GitHub Actions workflow development patterns and security recommended practices


# GitHub Actions Workflow Development

Guidelines for secure and maintainable GitHub Actions workflows.

## Security Best Practices

### Minimal Permissions

```yaml
# ✅ Minimal at workflow level
permissions:
  contents: read

# Increase per-job only when needed
jobs:
  deploy:
    permissions:
      contents: read
      deployments: write
```

### SHA Pinning (CRITICAL)

**Always pin third-party actions to full commit SHA and use the latest release:**

```yaml
# ✅ Pin to commit SHA - immutable and secure
- uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2
```

**Before submitting workflow changes:**

1. **Find Latest Release**: Check the action's releases page (e.g., `https://github.com/actions/checkout/releases`)
2. **Get Commit SHA**: Use GitHub MCP tools or the releases page to get the full 40-character commit SHA
3. **Include Version Comment**: Always include the version tag as a trailing comment (e.g., `# v6.0.2`)

### Credential Security

**Always set `persist-credentials: false` on checkout actions:**

```yaml
# ✅ Secure - credentials not persisted
- uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2
  with:
    persist-credentials: false
```

This prevents credentials from being stored in the Git config.

### Script Injection Prevention

```yaml
# ✅ Safe - environment variable
- name: Check PR title
  env:
    TITLE: ${{ github.event.pull_request.title }}
  run: |
    if [[ "$TITLE" =~ ^feat ]]; then
      echo "Valid feature PR"
    fi

# ❌ Unsafe - direct interpolation
- run: |
    if [[ "${{ github.event.pull_request.title }}" =~ ^feat ]]; then
```

### Secrets Handling

```yaml
# ✅ Reference secrets properly
env:
  API_KEY: ${{ secrets.API_KEY }}

# Mask generated sensitive values
- run: |
    TOKEN=$(generate-token)
    echo "::add-mask::$TOKEN"
    echo "TOKEN=$TOKEN" >> $GITHUB_ENV
```

## Workflow Structure

### YAML Document Start

Always begin workflow files with `---` for proper YAML parsing:

```yaml
---
name: CI
on:
  push:
    branches: [main]
```

### Environment Consistency

**Prefer configuration files over hardcoded versions:**

```yaml
# Go projects - use go.mod
- uses: actions/setup-go@7a3fe6cf4cb3a834922a1244abfce67bcef6a0c5 # v6.2.0
  with:
    go-version-file: go.mod

# Node.js projects - use .node-version or .nvmrc
- uses: actions/setup-node@v4
  with:
    node-version-file: ".node-version"
```

### Conditional Execution

```yaml
# Run only on main
- run: ./deploy.sh
  if: github.ref == 'refs/heads/main'

# Continue on error
- run: ./optional-step.sh
  continue-on-error: true

# Run even if previous failed
- run: ./cleanup.sh
  if: always()
```

## Verification Checklist (MANDATORY)

Before finalizing any workflow changes:

1. **Verify Each Action's Commit SHA:**
   - Use GitHub MCP tools (`get_commit` with the tag) to retrieve the correct 40-character SHA
   - **Do NOT assume** the SHA in comments or existing workflows is correct
   - **Always use the latest release** - check the releases page for each action

2. **Validate Action Existence:**
   - After updating SHAs, confirm the action exists at that commit
   - If the commit lookup fails or returns 404, the SHA is invalid

3. **Security Checks:**
   - Ensure `persist-credentials: false` is set on all checkout actions
   - Verify minimal permissions are configured at workflow level
   - Check that no secrets are directly interpolated in run commands

4. **Test YAML Syntax:**
   - Verify the workflow has valid YAML syntax
   - Ensure the file starts with `---`

## Before Making Changes

1. Check existing `.github/workflows/` for established patterns
2. Check `.github/dependabot.yml` for dependency automation settings
3. Verify action versions via releases pages using GitHub MCP tools
4. Consider CI time and complexity trade-offs

## Keeping Instructions Up-to-Date

**IMPORTANT**: When making changes to workflow files that introduce new patterns or security practices, update this instruction file to reflect those changes.

---

## Antipatterns

| Antipattern                  | Why It's Problematic                         | Better Approach                             |
| ---------------------------- | -------------------------------------------- | ------------------------------------------- |
| Using version tags (`v4`)    | Tags can be moved/deleted; supply chain risk | Pin to full 40-char commit SHA              |
| Direct string interpolation  | Script injection vulnerability               | Use environment variables                   |
| Workflow-level `write` perms | Excessive access if job compromised          | Minimal perms at workflow, increase per-job |
| Hardcoded language versions  | Drift between local and CI                   | Use version files (go.mod, .node-version)   |
| Assuming SHA validity        | Outdated SHAs break workflows                | Verify SHA against latest release           |
| Missing YAML document start  | Parser warnings and inconsistency            | Always start with `---`                     |

---
> Source: [github/gh-skyline](https://github.com/github/gh-skyline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
