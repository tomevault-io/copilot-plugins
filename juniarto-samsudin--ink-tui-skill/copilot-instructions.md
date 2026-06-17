## ink-tui-skill

> Reference and workflow for building terminal user interfaces with Ink and React. Use when building CLI tools, interactive command-line apps, keyboard navigation, multi-panel terminal layouts, Ink components such as Box and Text, Ink hooks such as useInput and useApp, or meow-based cli.tsx entry points.


# Ink TUI Skill

Build interactive terminal applications with Ink, React, and TypeScript.

## When to Use

- Building a CLI tool or interactive terminal app with Ink and React
- Adding keyboard navigation, multi-panel layouts, popups, or focus management to a TUI
- Working with Ink components such as `Box`, `Text`, `Newline`, `Spacer`, and `Static`
- Using Ink hooks such as `useInput`, `useApp`, `useFocus`, `useFocusManager`, or `useStdout`
- Wiring a `cli.tsx` entry point with `meow` and `render()`
- Testing terminal UI output with `ink-testing-library`

## Procedure

1. Confirm the CLI entry point and app boundary. Put argument parsing in `cli.tsx` and render the main UI from `app.tsx`.
2. Model the layout with `Box` and `Text` first. Prefer a simple header/body or sidebar/content split before adding transient UI.
3. Add `useInput()` early so the app stays alive and all keyboard handling is explicit.
4. Keep global keys in the top-level app and guard panel-specific input with a `focused` flag or Ink focus hooks.
5. Reset transient UI, such as popups, when selection or focus changes.
6. Build and test the CLI with `npm run build && node dist/cli.js`.

## Project Pattern

Use this structure for a standard Ink CLI:

```text
my-app/
├── package.json
├── tsconfig.json
└── source/
    ├── cli.tsx
    └── app.tsx
```

## References

- [Ink reference](./references/ink-reference.md)
- [Ink on GitHub](https://github.com/vadimdemedes/ink)

---
> Source: [juniarto-samsudin/ink-tui-skill](https://github.com/juniarto-samsudin/ink-tui-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
