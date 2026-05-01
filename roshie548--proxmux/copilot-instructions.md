## use-bun-instead-of-node-vite-npm-pnpm

> Terminal UI for managing Proxmox VE clusters, built with [Ink](https://github.com/vadimdemedes/ink) (React for CLIs) and [Bun](https://bun.sh).

# Proxmux Development Guide

Terminal UI for managing Proxmox VE clusters, built with [Ink](https://github.com/vadimdemedes/ink) (React for CLIs) and [Bun](https://bun.sh).

## Commands

```bash
bun install          # Install dependencies
bun run dev          # Run with hot reload
bun run build        # Build binary for current platform
bun run build:all    # Build binaries for all platforms
bun --bun tsc --noEmit  # Type check
```

## Architecture

```
src/
├── api/           # Proxmox API client and types
├── components/    # Reusable UI components (Sidebar, StatusBar, DetailView)
├── context/       # React contexts (EditModeContext)
├── hooks/         # Custom hooks (useProxmox, useKeyboard)
├── views/         # Main screens (Dashboard, VMs, Containers, Storage)
├── utils/         # Helpers (format.ts)
├── config/        # Config file loading
├── app.tsx        # Main app with layout and routing
└── index.tsx      # Entry point
```

## Ink Patterns

This is a terminal UI, not a web app. Uses Ink (React renderer for CLI).

### Layout with Box and Text

```tsx
import { Box, Text } from "ink";

<Box flexDirection="column" padding={1}>
  <Text bold color="blue">Title</Text>
  <Text dimColor>Subtitle</Text>
</Box>
```

### Keyboard Input

```tsx
import { useInput } from "ink";

useInput((input, key) => {
  if (input === "j" || key.downArrow) moveDown();
  if (input === "q") exit();
  if (key.return) confirm();
  if (key.escape) cancel();
});
```

### Responsive Sizing

```tsx
import { useStdout } from "ink";

const { stdout } = useStdout();
const width = stdout?.columns || 80;
const isNarrow = width < 80;

// Hide optional columns on narrow terminals
// Use compact formats when space is limited
// Add wrap="truncate" to prevent line wrapping
```

### Preventing Text Wrap

Always use `wrap="truncate"` on Text in tables/lists:

```tsx
<Box width={cols.name}>
  <Text wrap="truncate">{truncate(name, cols.name - 1)}</Text>
</Box>
```

## API Client

```tsx
import { getClient } from "./api/client";

const client = getClient();
const vms = await client.getVMs();
const config = await client.getContainerConfig(node, vmid);
```

## Adding a New View

1. Create `src/views/NewView.tsx`
2. Add to `src/app.tsx` switch statement
3. Add to `src/components/Sidebar.tsx` views array
4. Add keyboard hints to `src/components/StatusBar.tsx`

## Release Process

1. Update version in `package.json`
2. Update `CHANGELOG.md`
3. Commit and tag: `git tag v0.x.x && git push --tags`
4. GitHub Actions automatically:
   - Builds binaries for all platforms
   - Publishes to npm
   - Updates Homebrew formula

---
> Source: [roshie548/proxmux](https://github.com/roshie548/proxmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
