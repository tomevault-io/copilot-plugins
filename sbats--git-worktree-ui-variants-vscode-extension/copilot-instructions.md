## git-worktree-ui-variants-vscode-extension

> VS Code/Cursor extension for git worktree visual differentiation. Changes the title bar color and shows a status bar indicator with the current worktree name.

# Claude Code Guidelines for Git Worktree UI Variants

## Project Context
VS Code/Cursor extension for git worktree visual differentiation. Changes the title bar color and shows a status bar indicator with the current worktree name.

## Key Files to Understand
- `package.json`: Extension manifest with commands, configuration, and activation events
- `src/extension.ts`: Main entry point
- `src/worktree/detector.ts`: Git worktree detection logic
- `src/ui/color-manager.ts`: Title bar color customization
- `src/ui/status-bar.ts`: Status bar item management

## Architecture Overview

```
src/
├── extension.ts          # Activate/deactivate, wire up components
├── worktree/
│   ├── detector.ts       # Detect worktree type and name
│   ├── git-commands.ts   # Execute git commands
│   └── types.ts          # Worktree-related types
├── ui/
│   ├── color-manager.ts  # Apply workspace color customizations
│   ├── status-bar.ts     # Manage status bar item
│   └── color-utils.ts    # Color generation/manipulation
├── config/
│   └── settings.ts       # Read/write extension settings
└── types/
    └── index.ts          # Shared type definitions
```

## Common Tasks

### Adding a New Command
1. Add command to `package.json` contributes.commands
2. Create handler function in appropriate module
3. Register in `extension.ts` using `vscode.commands.registerCommand`
4. Add to context.subscriptions for cleanup

### Adding Configuration Option
1. Add to `package.json` contributes.configuration.properties
2. Update `src/config/settings.ts` to read new option
3. Handle configuration changes via `onDidChangeConfiguration`

### Modifying Worktree Detection
- Core logic in `src/worktree/detector.ts`
- Git commands abstracted in `src/worktree/git-commands.ts`
- Test with both main repos and linked worktrees

## Testing Commands
```bash
npm run compile    # Compile TypeScript
npm run lint       # Run ESLint
npm run test       # Run tests
npm run package    # Create VSIX package
```

## Debug Workflow
1. Press F5 in VS Code to launch Extension Development Host
2. Open a git repository (preferably with multiple worktrees)
3. Check Output panel > "Git Worktree UI" for logs
4. Use "Developer: Reload Window" to test changes

## Important Constraints
- Extension must work with `window.titleBarStyle: "custom"` setting
- Colors must have sufficient contrast (WCAG AA minimum)
- Must handle repositories without worktrees gracefully
- Support both VS Code and Cursor

## Dependencies
- No runtime dependencies (only VS Code API)
- Dev dependencies: TypeScript, ESLint, esbuild, @types/vscode, @types/node

## Build System
- esbuild for fast bundling
- Output to `dist/extension.js`
- Source maps in development mode

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SBats) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
