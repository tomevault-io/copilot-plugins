## rich-zig

> <!-- BEGIN:header -->

<!-- BEGIN:header -->
# CLAUDE.md

we love you, Claude! do your best today
<!-- END:header -->

<!-- BEGIN:rule-1-no-delete -->
## RULE 1 - NO DELETIONS (ARCHIVE INSTEAD)

You may NOT delete any file or directory. Instead, move deprecated files to `.archive/`.

**When you identify files that should be removed:**
1. Create `.archive/` directory if it doesn't exist
2. Move the file: `mv path/to/file .archive/`
3. Notify me: "Moved `path/to/file` to `.archive/` - deprecated because [reason]"

**Rules:**
- This applies to ALL files, including ones you just created (tests, tmp files, scripts, etc.)
- You do not get to decide that something is "safe" to delete
- The `.archive/` directory is gitignored - I will review and permanently delete when ready
- If `.archive/` doesn't exist and you can't create it, ask me before proceeding

**Only I can run actual delete commands** (`rm`, `git clean`, etc.) after reviewing `.archive/`.
<!-- END:rule-1-no-delete -->

<!-- BEGIN:irreversible-actions -->
### IRREVERSIBLE GIT & FILESYSTEM ACTIONS

Absolutely forbidden unless I give the **exact command and explicit approval** in the same message:

- `git reset --hard`
- `git clean -fd`
- `rm -rf`
- Any command that can delete or overwrite code/data

Rules:

1. If you are not 100% sure what a command will delete, do not propose or run it. Ask first.
2. Prefer safe tools: `git status`, `git diff`, `git stash`, copying to backups, etc.
3. After approval, restate the command verbatim, list what it will affect, and wait for confirmation.
4. When a destructive command is run, record in your response:
   - The exact user text authorizing it
   - The command run
   - When you ran it

If that audit trail is missing, then you must act as if the operation never happened.
<!-- END:irreversible-actions -->

<!-- BEGIN:code-discipline -->
### Code Editing Discipline

- Do **not** run scripts that bulk-modify code (codemods, invented one-off scripts, giant `sed`/regex refactors).
- Large mechanical changes: break into smaller, explicit edits and review diffs.
- Subtle/complex changes: edit by hand, file-by-file, with careful reasoning.
- **NO EMOJIS** - do not use emojis or non-textual characters.
- ASCII diagrams are encouraged for visualizing flows.
- Keep in-line comments to a minimum. Use external documentation for complex logic.
- In-line commentary should be value-add, concise, and focused on info not easily gleaned from the code.
<!-- END:code-discipline -->

<!-- BEGIN:no-legacy -->
### No Legacy Code - Full Migrations Only

We optimize for clean architecture, not backwards compatibility. **When we refactor, we fully migrate.**

- No "compat shims", "v2" file clones, or deprecation wrappers
- When changing behavior, migrate ALL callers and remove old code **in the same commit**
- No `_legacy` suffixes, no `_old` prefixes, no "will remove later" comments
- New files are only for genuinely new domains that don't fit existing modules
- The bar for adding files is very high

**Rationale**: Legacy compatibility code creates technical debt that compounds. A clean break is always better than a gradual migration that never completes.
<!-- END:no-legacy -->

<!-- BEGIN:dev-philosophy -->
## Development Philosophy

**Make it work, make it right, make it fast** - in that order.

**This codebase will outlive you** - every shortcut becomes someone else's burden. Patterns you establish will be copied. Corners you cut will be cut again.

**Fight entropy** - leave the codebase better than you found it.

**Inspiration vs. Recreation** - take the opportunity to explore unconventional or new ways to accomplish tasks. Do not be afraid to challenge assumptions or propose new ideas. BUT we also do not want to reinvent the wheel for the sake of it. If there is a well-established pattern or library take inspiration from it and make it your own. (or suggest it for inclusion in the codebase)
<!-- END:dev-philosophy -->

<!-- BEGIN:testing-philosophy -->
## Testing Philosophy: Diagnostics, Not Verdicts

**Tests are diagnostic tools, not success criteria.** A passing test suite does not mean the code is good. A failing test does not mean the code is wrong.

**When a test fails, ask three questions in order:**
1. Is the test itself correct and valuable?
2. Does the test align with our current design vision?
3. Is the code actually broken?

Only if all three answers are "yes" should you fix the code.

**Why this matters:**
- Tests encode assumptions. Assumptions can be wrong or outdated.
- Changing code to pass a bad test makes the codebase worse, not better.
- Evolving projects explore new territory - legacy testing assumptions don't always apply.

**What tests ARE good for:**
- **Regression detection**: Did a refactor break dependent modules? Did API changes break integrations?
- **Sanity checks**: Does initialization complete? Do core operations succeed? Does the happy path work?
- **Behavior documentation**: Tests show what the code currently does, not necessarily what it should do.

**What tests are NOT:**
- A definition of correctness
- A measure of code quality
- Something to "make pass" at all costs
- A specification to code against

**The real success metric**: Does the code further our project's vision and goals?
<!-- END:testing-philosophy -->

<!-- BEGIN:footer -->
---

we love you, Claude! do your best today
<!-- END:footer -->


---

## Project-Specific Content

<!-- Add your project's toolchain, architecture, workflows here -->
<!-- This section will not be touched by haj.sh -->

# rich_zig - Terminal Rich Text Library

A full-featured Zig port of Python's Rich library. Provides beautiful terminal output with styled text, tables, panels, progress bars, trees, and more.

- **Version**: 2.0.0
- **Minimum Zig**: 0.16.0
- **No external dependencies** - uses only Zig standard library

---

## Zig Toolchain

```bash
zig build              # Build library and executable
zig build run          # Run the comprehensive demo
zig build test         # Run all tests
zig build test -Doptimize=ReleaseSafe  # Test with optimization
zig fmt src/           # Format before commits
```

---

## Project Layout

```
rich_zig/
├── build.zig           # Build configuration
├── build.zig.zon       # Package manifest
├── src/
│   ├── root.zig        # Library root (public API)
│   ├── main.zig        # Demo executable
│   │
│   ├── color.zig       # Color types and conversion
│   ├── style.zig       # Text styling attributes (13 attrs incl underline2/frame/encircle)
│   ├── segment.zig     # Atomic rendering unit
│   ├── cells.zig       # Unicode width calculation
│   ├── measure.zig     # Measurement protocol (min/max width reporting)
│   ├── theme.zig       # Named style registry for theming
│   │
│   ├── markup.zig      # BBCode-like syntax parsing
│   ├── text.zig        # Styled text with spans
│   ├── box.zig         # Box drawing styles
│   ├── pretty.zig      # Comptime pretty printer for Zig types
│   ├── highlighter.zig # Pattern-based auto-highlighting (numbers, URLs, paths)
│   ├── ansi.zig        # ANSI escape sequence parsing and stripping
│   │
│   ├── terminal.zig    # Terminal detection
│   ├── console.zig     # Main console interface (theme-aware)
│   ├── emoji.zig       # Emoji support
│   │
│   └── renderables/    # Complex UI components
│       ├── mod.zig
│       ├── panel.zig
│       ├── table.zig
│       ├── rule.zig
│       ├── progress.zig
│       ├── tree.zig
│       ├── padding.zig
│       ├── align.zig
│       ├── columns.zig
│       ├── layout.zig
│       ├── live.zig
│       ├── json.zig
│       └── syntax.zig
│
└── .claude/
    ├── agents/         # Claude agents
    ├── skills/         # Claude skills (/test)
    └── settings.local.json
```

---

## Architecture: 4 Phases

**Phase 1 - Core Types**: `color`, `style`, `segment`, `cells`, `measure`, `theme`
**Phase 2 - Text/Markup**: `markup`, `text`, `box`, `pretty`, `highlighter`, `ansi`
**Phase 3 - Terminal/Console**: `terminal`, `console`, `emoji`
**Phase 4 - Renderables**: `panel`, `table`, `rule`, `progress`, `tree`, `layout`, `json`, `syntax`, etc.

All renderables implement: `render(width, allocator) ![]Segment`
Renderables may also implement: `measure(max_width, allocator) !Measurement`

---

## Key Patterns

### Explicit Allocators

All public APIs take `allocator: std.mem.Allocator`. No global state.

```zig
var panel = Panel.fromText(allocator, "content");
defer panel.deinit();
```

### Builder Pattern (Fluent API)

```zig
const panel = Panel.fromText(alloc, "content")
    .withTitle("Title")
    .withWidth(30)
    .withStyle(box.rounded);
```

### Error Handling

```zig
fn loadConfig(path: []const u8) !Config {
    const file = try fs.open(path);
    defer file.close();
    return try parseConfig(file);
}

// Explicit error sets for API boundaries
const ConfigError = error{ FileNotFound, ParseFailed, InvalidFormat };
```

### Optional Handling

```zig
// Prefer if/orelse over .? when handling is needed
if (items.get(index)) |item| {
    // safe to use item
} else {
    // handle missing case
}

// Use orelse for defaults
const value = optional orelse default_value;

// Use .? only when null is truly unexpected (will panic)
const ptr = maybe_ptr.?;
```

### Memory Safety

```zig
// Always use defer for cleanup
const buffer = try allocator.alloc(u8, size);
defer allocator.free(buffer);

// Prefer slices over raw pointers
fn process(data: []const u8) void { ... }
```

---

## Bug Severity

### Critical - Must Fix Immediately

- `.?` on null (panics)
- `unreachable` reached at runtime
- Index out of bounds
- Integer overflow in release builds (undefined behavior)
- Use-after-free or double-free
- Memory leaks in long-running paths

### Important - Fix Before Merge

- Missing error handling (`try` without proper catch/return)
- `catch unreachable` without justification
- Ignoring return values from `!T` functions
- Race conditions in threaded code

### Contextual - Address When Convenient

- TODO/FIXME comments
- Unused imports or variables
- Suboptimal comptime usage
- Excessive debug output

---

## Available Claude Tools

### Agents

| Agent | Model | Purpose |
|-------|-------|---------|
| coder-sonnet | sonnet | Fast, precise code changes |
| gemini-analyzer | sonnet | Large-context analysis via Gemini CLI |
| build-verifier | sonnet | Cross-platform build validation |

### Skills

| Skill | Purpose |
|-------|---------|
| `/test` | Run `zig build test` with optional optimization level |

---

## Version Updates (SemVer)

When making commits, update `version` in `build.zig.zon`:

- **MAJOR** (X.0.0): Breaking changes or incompatible API modifications
- **MINOR** (0.X.0): New features, backward-compatible additions
- **PATCH** (0.0.X): Bug fixes, small improvements, documentation

---

## Issue Tracking: beads_rust (br)

Local-first, non-invasive issue tracker stored in `.beads/`. No external services required.

### Core Commands

```bash
br init                    # Initialize in current repo
br create "Title"          # Create issue (prompts for details)
br q "Quick note"          # Quick capture with minimal ID
br list                    # Show all open issues
br ready                   # Show unblocked, actionable work
br show <id>               # Display issue details
br close <id>              # Mark complete
```

### Issue Properties

```bash
br create "Bug title" --type bug --priority 1
br update <id> --status in_progress
br update <id> --priority 2
br label add <id> "refactor"
br dep add <child-id> <parent-id>   # child blocked by parent
```

**Priority**: 0=critical, 1=high, 2=medium, 3=low, 4=backlog
**Status**: open, in_progress, closed, deferred
**Type**: bug, feature, task

### Filtering

```bash
br list --status open --priority 1
br list --type bug --assignee user@example.com
br list --label "refactor"
br blocked                  # Issues waiting on dependencies
```

### Sync for Git

```bash
br sync --flush-only        # Export DB to .beads/issues.jsonl
br sync --import-only       # Import JSONL back to DB
```

**Workflow**: After modifying issues, run `br sync --flush-only` then commit `.beads/`.

### Machine-Readable Output

```bash
br list --json              # JSON output for scripting
br ready --json             # Actionable items as JSON
br show <id> --json         # Single issue as JSON
```

### Storage

```
.beads/
  beads.db      # SQLite database (local state)
  issues.jsonl  # Git-friendly export (commit this)
```

---
> Source: [hotschmoe/rich_zig](https://github.com/hotschmoe/rich_zig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
