## swiftui-app-template

> - Keep `View.body` small.

# Conventions

## Files
- Keep `View.body` small.
- Move sections/components to `extension` blocks.

## Naming
- Sections: `*Section`
- Rows/components: `*Row`, `*Card`, `*Screen`

## State
- `ViewModel` is `@MainActor` + `ObservableObject`
- Prefer small, testable methods over giant `body` logic

---
> Source: [YuriCernov/swiftui-app-template](https://github.com/YuriCernov/swiftui-app-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
