## openpets

> This is the OpenPets monorepo - a desktop pet application for Claude Code and coding agents. It provides an animated desktop companion that reacts to agent activity.

# AGENTS.md

## Repository Map

This is the OpenPets monorepo - a desktop pet application for Claude Code and coding agents. It provides an animated desktop companion that reacts to agent activity.

### Root Structure

```
openpets/
├── package.json              # Workspace root: bun workspaces, build scripts
├── tsconfig.base.json        # Shared TypeScript configuration
├── README.md                 # User documentation
├── INSTALL.md                # Installation guide
├── codemap.md                # Root architecture documentation
├── apps/
│   └── desktop/              # Electron desktop application
│       ├── src/
│       │   ├── main.ts       # Main process: window, tray, IPC server
│       │   ├── preload.ts    # Renderer bridge API
│       │   ├── ipc-server.ts # IPC socket server implementation
│       │   └── renderer/     # React frontend
│       │       └── src/
│       │       │   ├── App.tsx      # Pet rendering and interactions
│       │       │   ├── main.tsx     # React entry point
│       │       │   └── styles.css   # Sprite animation styles
│       │       └── codemap.md
│       └── codemap.md
├── packages/
│   ├── core/                 # Domain logic, state, IPC protocol
│   │   └── src/
│   │       ├── states.ts     # Pet state definitions (13 states)
│   │       ├── event.ts      # Event validation and creation
│   │       ├── reducer.ts    # State machine reducer
│   │       ├── lifecycle.ts  # Multi-client lease management
│   │       ├── ipc.ts        # IPC protocol (v2, sockets/pipes)
│   │       ├── config.ts     # Platform config paths
│   │       └── codex-mapping.ts # Animation state mapping
│   ├── client/               # TypeScript IPC client
│   │   └── src/
│   │       ├── client.ts     # Client implementation
│   │       ├── event-input.ts # Input normalization
│   │       └── errors.ts     # Error types
│   ├── cli/                  # Command-line interface
│   │   └── src/
│   │       └── index.ts      # CLI commands (start, event, install)
│   ├── mcp/                  # MCP server for Claude Code
│   │   └── src/
│   │       ├── server.ts     # MCP server bootstrap
│   │       ├── tools.ts      # Tool implementations
│   │       ├── launcher.ts   # Desktop app launcher
│   │       └── safety.ts     # Speech validation and rate limiting
│   └── pet-format-codex/     # Pet format loader
│       └── src/
│           ├── loader.ts     # Directory loading and validation
│           ├── types.ts      # Type definitions
│           └── validation.ts # Manifest and path validation
```

### Package Dependencies

```
core (foundation - no deps)
  ↑
  ├──→ pet-format-codex
  ↑
  ├──→ client
  ↑       ↑
  │       ├──→ cli
  │       ↑
  └──→ mcp
  ↑
desktop (uses core + pet-format-codex directly)
```

### Key Architectural Patterns

1. **State Machine**: Pure reducer function transforms events into pet states with temporary state expiration
2. **Lease Management**: Multi-client coordination via heartbeat-based leases (MCP, CLI, OpenCode)
3. **IPC Protocol**: Version 2 protocol over Unix sockets (macOS/Linux) or named pipes (Windows)
4. **Security**: Sandboxed renderer, CSP headers, path validation, symlink prevention
5. **Pet Format**: Codex/Petdex compatible - spritesheet + pet.json manifest

### Integration Points

- **Desktop App**: Electron main process hosts IPC server, renderer shows animated pet
- **CLI**: `openpets` command for launching, events, pet installation
- **MCP**: `claude mcp add openpets` for Claude Code integration
- **Client Library**: `@open-pets/client` for custom integrations

### Build Commands

```bash
bun install          # Install all dependencies
bun run build        # Build all packages + desktop
bun run typecheck    # Type check all packages
bun run test         # Run tests
bun run dev:desktop  # Dev mode with hot reload
```

### Exclusions

- Tests (`*.test.ts`) are excluded from codemap documentation
- Build outputs (`dist/`, `out/`) are generated and excluded
- Documentation in `docs/` folders is excluded

---
> Source: [alvinunreal/openpets](https://github.com/alvinunreal/openpets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
