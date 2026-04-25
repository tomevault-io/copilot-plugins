## reference-graph

> VS Code extension that visualizes code references as an interactive graph.

# Reference Graph - Agent Instructions

VS Code extension that visualizes code references as an interactive graph.

For commands, project structure, and architecture details, see **[docs/DEVELOPMENT.md](docs/DEVELOPMENT.md)**.

## Documentation Rules (CRITICAL)

**Single Source of Truth:**

| Document | Purpose |
|----------|---------|
| `README.md` | User-facing documentation (features, usage) |
| `CONTRIBUTING.md` | Contributor guidelines (PRs, tests, fixtures) |
| `docs/DEVELOPMENT.md` | Technical reference (commands, structure, architecture) |
| `AGENTS.md` | Agent workflow instructions (this file) |
| `package.json` | Extension manifest, commands, configuration schema |

**When to update what:**

| Change | Update |
|--------|--------|
| New user-facing feature | `README.md` |
| New command or config option | `package.json` contributes section |
| New npm script | `package.json` scripts, `docs/DEVELOPMENT.md` Commands |
| Changed project structure | `docs/DEVELOPMENT.md` Project Structure table |
| Test/fixture patterns | `CONTRIBUTING.md` |
| Internal refactoring | Nothing (code is the documentation) |

**Never duplicate information across files.** If something is defined in code (types, config), point to the file instead of copying content.

## Workflow Guidelines

1. **Read before writing** — Check existing patterns in similar files before adding new code
2. **Types are contracts** — `src/graph/types.ts` defines Extension↔Webview protocol. Change carefully.
3. **Test manually** — F5 debug, place cursor on a symbol, right-click "Show Reference Graph"
4. **Build both** — After webview changes, run `npm run build` (not just `build:extension`)

---
> Source: [yuichkun/reference-graph](https://github.com/yuichkun/reference-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
