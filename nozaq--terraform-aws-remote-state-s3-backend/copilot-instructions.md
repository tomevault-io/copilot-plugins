## terraform-aws-remote-state-s3-backend

> - Use English for all documentation and comments.

# Repository Guidelines

- Use English for all documentation and comments.

## GitHub Actions

- Pin every action to a full commit SHA with a `# vX.Y.Z` comment,
  e.g. `uses: actions/checkout@9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0 # v7.0.0`.
- Set workflow-level permissions to empty (`permissions: {}`) and grant
  the minimum required permissions per job.
- Set `persist-credentials: false` on `actions/checkout`.

---
> Source: [nozaq/terraform-aws-remote-state-s3-backend](https://github.com/nozaq/terraform-aws-remote-state-s3-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
