## shadowos

> This document describes the automated agents, bots, and external services that interact with this repository.

# Agents

This document describes the automated agents, bots, and external services that interact with this repository.

## Current agents

*No active agents are currently configured in this repository.*

## Adding a new agent — checklist

Before adding or enabling a new automated agent, follow this checklist:

1. **Design & justification**: Document the goal and expected behavior in an issue or RFC.
2. **Owner & contact**: Assign a maintainer or team responsible for the agent.
3. **Permissions**: Determine minimum permissions (prefer short-lived tokens/OIDC).
4. **Secrets**: Store secrets in GitHub Actions Secrets; never in the repo.
5. **Config**: Add config files under `.github/` with clear comments.
6. **Review**: Configuration changes require code review.
7. **Testing**: Test workflows locally (e.g., using `act`) or in a fork.
8. **Monitoring**: Ensure failures are routed to the team.
9. **Rollback**: Document how to disable the agent quickly.
10. **Security**: Perform a security review for agents with write permissions.

## Common Configuration Locations

- **GitHub Actions**: `.github/workflows/*.yml`
- **Dependabot**: `.github/dependabot.yml`
- **Renovate**: `renovate.json`

## Ownership

- **CI**: TBD
- **Security**: TBD

## Examples

### Minimal GitHub Actions Workflow

```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: cargo test
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shadowm-mounarch)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shadowm-mounarch)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
