## shipyard

> Development guidelines for the shipyard repository.

# shipyard

Development guidelines for the shipyard repository.

## Commit Messages

@.agents/commit-templates.md

## Workflows

### Testing

#### Markdown

Run after editing any `.md` file, before committing:

```bash
make markdownlint
```

### CVE Fixes

@.agents/workflows/cve-fix.md

### Konflux Builds

On `devel`, `.rpm-lockfiles/` has documentation (`README.md`) and diagnostic scripts (`check-repo-access.sh`, `verify-packages.sh`).

---
> Source: [submariner-io/shipyard](https://github.com/submariner-io/shipyard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
