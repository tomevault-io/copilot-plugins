## pi-read

> Commit when a task is completed.

# Agent Instructions

## Workflow

Commit when a task is completed.

When changing tool behavior, update README examples and config docs in the same task.

## Verification

Run before completion:

```bash
npm run format:check
npm run typecheck
npm run test
npm run build
npm pack --dry-run
```

## Commit Style

Match existing commits:

- `Add initial extension implementation`
- `Update README with configuration examples`
- `Format code with Prettier`

## Dependencies and packaging

Keep test/tooling dependencies in `devDependencies` unless runtime is truly required.

Keep the published package minimal via the `files` allowlist in `package.json`.

## Documentation

Use proper formatting when writing documentation, but do not go overboard with the formatting. The content should speak for itself.

## Release checklist

1. Review every change since the previous release and update `CHANGELOG.md` under `Unreleased`.
2. Run `npm run format:check`, `npm run typecheck`, `npm run test`, `npm run build`, and `npm pack --dry-run`.
3. Verify the package contents and README configuration examples.
4. Set the package version and convert the `Unreleased` changelog section to that version and release date.
5. Commit the release changes before publishing.

---
> Source: [arcanemachine/pi-read](https://github.com/arcanemachine/pi-read) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
