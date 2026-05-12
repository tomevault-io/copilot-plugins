## mnehmos-rpg-mcp

> The backend game engine for Quest Keeper AI. **32 MCP tools** (28 consolidated action-routed + 4 meta/event) for complete RPG mechanics.

# RPG-MCP - Claude Code Instructions

## This Repository

The backend game engine for Quest Keeper AI. **32 MCP tools** (28 consolidated action-routed + 4 meta/event) for complete RPG mechanics.
**Philosophy:** "LLM describes, engine validates" - Database is source of truth.
**Status:** Alpha - 1889 tests passing, 85+ test files, MCP Protocol fully integrated

## Key Commands

```bash
npm test                          # Run all tests (Vitest)
npm test -- tests/specific.test.ts   # Single test file
npm test -- --watch               # Watch mode
npm run build                     # Compile TypeScript
npm run build:binaries            # Create standalone executables
```

## Key Directories

```
src/
├── server/
│   ├── consolidated/  # 28 action-routed tool handlers
│   ├── handlers/      # Extracted handler implementations (combat, spatial)
│   ├── index.ts       # MCP server entry + transport setup
│   └── meta-tools.ts  # search_tools, load_tool_schema
├── engine/
│   ├── combat/   # Encounters, initiative, damage
│   ├── magic/    # Spells, concentration, scrolls, auras
│   ├── spatial/  # Grid, collision, movement
│   ├── worldgen/ # Procedural generation
│   └── strategy/ # Nation simulation
├── storage/      # SQLite repos & migrations
├── schema/       # Zod validation (29 schemas)
├── utils/        # Action router, fuzzy matching
└── math/         # Dice, algebra, physics

tests/            # Mirror of src/ structure (85+ files)
docs/             # White paper, ADRs, LLM spatial guide
```

## Git Commit Convention

```
fix(component): description   # Bug fixes
feat(component): description  # New features
test(component): description  # Test additions
refactor(component): description  # Code cleanup
```

## The Git Pulse Rule

**After successful test pass, immediately commit:**

```bash
git add . && git commit -m "type(scope): message"
```

Do NOT ask permission for local commits. Just save the state.

## TDD Loop

1. Write failing test (RED)
2. Implement fix (GREEN)
3. Refactor if needed
4. Commit
5. Repeat

## Deploy to Frontend

After building binaries:

```powershell
copy dist-bundle\rpg-mcp-win.exe "C:\Users\mnehm\Desktop\Quest Keeper AI attempt 2\src-tauri\binaries\rpg-mcp-server-x86_64-pc-windows-msvc.exe"
```

---
> Source: [Mnehmos/mnehmos.rpg.mcp](https://github.com/Mnehmos/mnehmos.rpg.mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
