## devsandbox

> After completing the task always run:

# devsandbox

## Coding practices

After completing the task always run:

- `task test` - to run tests
- `task lint` - to run lint

Always prefer reasonable defaults if that it possible. Reduce amount of work for the user to do when this is possible.
All defaults must be secure by default, and not cause any security issues if used without modification.
Never bind to all interfaces by default, and do not expose any ports by default.

Errors must always be handled and reported when present. Silent failures are not acceptable.

## Tools to use

Tools management: mise
Create .mise.toml with all tools to be used and pin the dependencies.

Use `task` for setting up automation: create a taskfile which will cover build, lint, test, running the appliication.

CI: repository will be hosted at gitea. Update it to correlate with current project when needed.

Linters: golangci + go default linters.

---
> Source: [zekker6/devsandbox](https://github.com/zekker6/devsandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
