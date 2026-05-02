## hn-cli

> Interactive terminal UI for browsing Hacker News using OpenTUI.

# HN CLI

Interactive terminal UI for browsing Hacker News using OpenTUI.

## Commands

```bash
bun run start      # Run the app
bun run dev        # Run with watch mode
bun run test       # Run tests
bun run typecheck  # Check types
```

## Debug Commands

```bash
bun run debug              # Test long comment wrapping
bun run debug story-list   # Test story list view
bun run debug highlighted-comment # Test comment highlighting
```

## Rules

- [Keyboard Shortcuts](/.claude/rules/keyboard-shortcuts.md) — All shortcuts for main view and settings
- [Data Rules](/.claude/rules/data-rules.md) — Filter defaults and ranking algorithm
- [Publishing](/.claude/rules/publishing.md) — npm release workflow and telemetry

## Verification

After making changes:
- `bun run typecheck` - Type checking
- `bun test` - Test suite

---
> Source: [brianlovin/hn-cli](https://github.com/brianlovin/hn-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
