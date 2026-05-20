## nx-plugin-for-aws

> Install dependencies:

# Instructions for Claude

## Commands

Install dependencies:

```bash
pnpm i
```

Build:

```bash
pnpm nx run-many --target build --all
```

Run tests and update snapshots:

```bash
pnpm nx run @aws/nx-plugin:test -u
```

## Tips

- Always use `npx -y` (not bare `npx`) to avoid the "Ok to proceed?" prompt hanging in non-interactive environments.

## Best Practices

- Always ensure the build passes before raising a PR
- Update snapshots if there are failures due to snapshot changes
- Use conventional commits, referencing the generator you are working on, eg "feat(ts#project): my commit message"
- Raise PRs following the PR template
- Use the existing codebase to inform code style, testing style, etc

---
> Source: [awslabs/nx-plugin-for-aws](https://github.com/awslabs/nx-plugin-for-aws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
