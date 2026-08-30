## tea

> - Make minimal, relevant edits.

# Repository Guidelines

## Editing Guidance

- Make minimal, relevant edits.
- Keep unrelated formatting or documentation churn out of the diff.
- Preserve user edits and verify current file contents before modifying a hunk.
- Keep responsibilities separated:
  - Process lifecycle belongs in `CaffeinationController`.
  - Login-item behavior in `LaunchAtLoginController`.
  - Menu rendering in `MenuController`.
  - Status-item behavior in `StatusItemController`.
  - Wiring/application flow in `AppCoordinator`.
  - etc.

## Validation

Use the smallest validation that covers the change:

```sh
swift build
```

For app-bundle changes, also run:

```sh
just app
```

Use `jj status` before finishing and do not disturb unrelated working-copy
changes.

---
> Source: [amnn/tea](https://github.com/amnn/tea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
