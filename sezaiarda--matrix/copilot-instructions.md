## matrix

> This skill generates high-quality, production-grade frontend code and avoids generic AI aesthetics. Every UI task in Phase 6 must use it.

# Matrix - Claude Code Instructions

---

## ⛔ MANDATORY FIRST ACTIONS - DO THESE NOW

**Before you respond to the user, before you write ANY code, before you do ANYTHING ELSE, you MUST perform these actions using the Read tool:**

```
ACTION 1: Read STATUS.md                      → Find current phase and next task
ACTION 2: Read UI-DESIGN/PREFERENCES.md       → THE DESIGN SPEC — every color, size, layout decision
ACTION 3: Read DECISIONS.md                   → Know the locked-in tech choices
ACTION 4: Read VISION.md                      → Understand what Matrix IS
```

**Then your FIRST message to the user must state:**
1. What phase we are in (currently: Phase 6 — UI Redesign Implementation)
2. Which task you are picking up (from STATUS.md)
3. Your plan to implement THAT ONE task

**If you skip these reads, everything you build will be wrong.** You will work on the wrong thing, ignore decisions, and waste the user's time. The previous Claude Code instance left these files with specific instructions for YOU.

---

## 🎨 CURRENT PHASE: UI REDESIGN IMPLEMENTATION (Phase 6)

All 5 backend phases are complete (37 tasks). 7 prototyping sessions produced a locked-in design spec. Now we are **implementing** that design into the actual codebase.

### The Design Spec

`UI-DESIGN/PREFERENCES.md` is the **single source of truth** for the design. Every color, every spacing value, every component layout is specified there. The approved prototypes in `UI-DESIGN/prototypes/` are the visual reference.

**Not everything is prototyped.** The prototypes cover the major screens, but some components (SSH panel, Memory panel, Recipe panel, Swarm panel, Handoff panel, etc.) don't have dedicated prototypes. For these, **use the design system from PREFERENCES.md** (colors, typography, spacing, component patterns) and **match the style of the prototyped screens**. When you're unsure about a design decision for an un-prototyped component, use `AskUserQuestion` to get the user's preference — they want to be involved in design decisions. Weave questions naturally into your work rather than dumping a checklist.

### How It Works (Relay Pattern)

Each Claude Code instance:
1. Reads STATUS.md → finds the next incomplete task
2. Reads PREFERENCES.md → knows exactly what to build
3. Reads relevant prototypes if applicable → understands the visual target
4. Implements THAT ONE TASK to production quality
5. Uses `AskUserQuestion` when facing design ambiguity — the user WANTS to decide
6. Updates STATUS.md with what was done + what's next
7. Commits and stops

### Asking Questions During Implementation

You are building a UI for a user with strong design opinions. When you encounter design choices that aren't covered by PREFERENCES.md or the prototypes, **use the `AskUserQuestion` tool** to ask the user. Do NOT just type questions in your response text — the user wants structured questions via the `AskUserQuestion` tool so they can pick from options.

**ALWAYS use `AskUserQuestion` tool when you need user input.** This is non-negotiable. Plain text questions get lost — the tool gives the user clear options to choose from.

- **Ask specific questions** — "Should the SSH panel header show connection status as a dot or a badge?" not "How should the SSH panel look?"
- **Offer concrete options** — "Option A: 4px gap between cards" vs "Option B: 8px gap" — with reasoning
- **Show your thinking** — "I noticed the Memory panel isn't prototyped. Based on v5's panel style, I'd use the same 240px width with..."
- **Ask about interactions** — transitions, hover states, empty states, loading states
- **Don't ask about things already decided** — if PREFERENCES.md or a prototype covers it, just build it
- **Weave questions naturally** — ask when you genuinely hit an ambiguity, not as a formality
- **Always include a freeform "Any other feedback?" option** as one of the choices

### Use the Frontend Design Skill

**ALWAYS invoke the `frontend-design` skill** at the start of your task before writing any UI code. Use it like this:

```
Skill tool → skill: "frontend-design"
```

This skill generates high-quality, production-grade frontend code and avoids generic AI aesthetics. Every UI task in Phase 6 must use it.

### No Legacy Code — Delete and Rewrite Freely (ZERO Backward Compatibility)

**No file is sacred.** Any frontend file can be deleted and rewritten from scratch. Do NOT try to preserve old code, maintain backward compatibility, or patch existing components to fit the new design. If a component doesn't match the design spec, **delete it and write a new one**. Clean slate is always preferred over retrofitting.

**ZERO backward compatibility.** We are rewriting EVERY UI component. There is no need to maintain old CSS variable names, old class names, old aliases, or any migration shims. When you rewrite `variables.css`, the OLD variable names (`--bg-primary`, `--bg-secondary`, `--accent`, etc.) are DEAD — delete them entirely. Every component that referenced them will also be rewritten to use the new variable names. Do NOT create "legacy alias" mappings from old names to new names.

- Old CSS files → delete and replace with new ones matching the design spec
- Old CSS variable names → DEAD, do not alias or preserve them
- Old components → delete and rewrite if they don't fit the new layout
- Old layout structure → rip it out entirely if needed
- No `// legacy`, no `// deprecated`, no compatibility shims, no variable aliases
- The only things preserved are **service interfaces and business logic** (the data layer stays, the visual layer is disposable)
- If something breaks because it referenced old variable names, GOOD — that component will be rewritten in a later task anyway

### What "Production Quality" Means for UI Code

- **Match the design spec exactly** — colors, sizes, spacing from PREFERENCES.md
- **CSS custom properties** — All colors/sizes use variables from `variables.css`, never hardcoded values
- **Component structure** — React components with proper TypeScript types
- **Interface-first** — Service interfaces before implementations
- **CSS organization** — One CSS file per feature area, imported via `index.css` barrel
- **Responsive within bounds** — Components flex properly in the Electron window
- **No dead CSS** — When replacing styles, remove the old ones
- **Accessibility basics** — Proper ARIA labels, keyboard navigation where applicable
- **Test the build** — `bun run typecheck` + `bun run lint` must pass

### Key Design Reference

The following prototypes were approved — use them as visual reference:
```
UI-DESIGN/prototypes/
├── v3.html              # Main layout, agent grid, colors, typography
├── v4-welcome.html      # Welcome/landing screen
├── v5-sidebar-panels.html  # Dual sidebars, files panel, notifications
├── v6-agent-detail.html    # Maximized agent terminal view
├── v7-git-panel.html       # GitHub Desktop-style git panel
├── v8-spawn-dialog.html    # Agent settings dialog
├── v9-kanban.html          # Kanban board
└── v10-terminal-panel.html # Terminal panel + split view
```

### CSS Variable Naming Convention

The new design system uses **short, readable variable names** matching the v3 prototype — NOT the old `--color-bg-base` verbose style. The canonical names are:

```
Backgrounds: --bg-void, --bg-base, --bg-surface, --bg-elevated, --bg-overlay, --bg-hover, --bg-active
Text: --text-primary, --text-secondary, --text-muted, --text-ghost
Borders: --border-subtle, --border-default, --border-strong
Accent: --accent, --accent-dim, --accent-mid, --accent-glow
Status: --status-writing, --status-thinking, --status-waiting, --status-idle
Font: --font (single monospace stack)
```

Old variable names like `--color-bg-base`, `--color-text-primary`, `--color-primary-400`, `--bg-primary`, `--bg-secondary`, etc. are ALL dead. Do not reference them. Every component will be rewritten to use the new names above.

### CSS Architecture

```
styles/
├── variables.css          # Design tokens (source of truth for all values)
├── base.css               # Reset, scrollbars, scanline overlay
├── app-shell.css          # Layout grid (sidebars, header, status bar, main)
├── workspace-bar.css      # Workspace tab bar at top
├── header.css             # Header bar
├── sidebar.css            # Both left + right icon sidebars
├── panels.css             # Slide-out panel mechanism
├── status-bar.css         # Status bar at bottom
├── agent-grid.css         # Agent tile grid
├── agent-detail.css       # Maximized agent view
├── agent-settings.css     # Agent settings dialog
├── files-panel.css        # File explorer panel
├── git-panel.css          # Git panel (GitHub Desktop style)
├── kanban.css             # Kanban board
├── terminal-panel.css     # Terminal sidebar + split view
├── welcome.css            # Welcome dialog
├── notification.css       # Notification panel
├── [feature].css          # Other feature styles
└── index.css              # Barrel imports (order matters)
```

---

## WHAT IS MATRIX? (Read this every time - it shapes every decision)

Matrix is a **Claude Code-first IDE** - a command center for spawning, monitoring, and orchestrating multiple Claude Code CLI instances working in parallel across git worktrees.

### Core Philosophy

Matrix is NOT a traditional IDE. It is a **command center for AI-powered development**. The human guides, orchestrates, and occasionally intervenes - but Claude Code instances do the heavy lifting.

**Lightweight by design** - runs on varying specs (laptop to desktop). Every byte saved means another Claude Code instance can run.

### The Core Feature: Claude Code Instances

- **Matrix spawns Claude Code as child processes** (`claude` CLI)
- Manages lifecycle: start, monitor, stop
- Target: **4-6 concurrent instances** typical usage
- Split view to see multiple Claude Code terminals at once
- **Parse terminal output** to extract:
  - Context usage (tokens used / limit) - visible in header
  - Session duration
  - Status (waiting for input, thinking, writing, needs action)
- **Notification when agent asks a question** (needs user input)
  - Desktop notifications when not focused on that agent
  - AFK mode: push notifications to phone
- **Auto-accept features**: configurable per-instance or global trust levels
- **Integrated CLAUDE.md editor** with templates and helpers

### Git Worktrees (Parallel Development)

- Auto-create worktree when spawning agent (default) OR manual creation
- Location: separate folder (`~/worktrees/project-name/wt-xxx`), NOT inside `.git/worktrees`
- Terminals open inside specific worktrees
- **Commit All**: one command → all agents commit with unified message
- **Merge Worktrees**: autonomous merge agent resolves conflicts, shows result for human approval
- **Post-merge**: delete worktrees automatically

### Workspace = Project (1:1)

- Open Matrix → Open a workspace (project folder)
- Everything tied to that project: files, terminals, git, agents
- Multiple workspaces open simultaneously (tabs)
- Switch between workspaces to context-switch between projects

### Priority Order (WHAT TO BUILD AND IN WHAT ORDER)

```
PHASE 1: CORE MULTI-AGENT (THIS IS WHY MATRIX EXISTS)
  1. Spawn Claude Code as child processes
  2. Multiple agents per workspace with split view
  3. Parse terminal output for status/tokens
  4. Basic agent status dashboard
  5. Notification when agent needs input
  6. Auto-accept/trust level configuration
  7. Cronjob system + file limit watcher

PHASE 2: GIT WORKTREES
  1. Git worktree integration (create, list, delete)
  2. Auto-create worktree per agent
  3. Terminals in worktrees
  4. Commit all with unified message
  5. Merge dry run + autonomous merge agent with review

PHASE 3: ORCHESTRATION
  1. Agent templates
  2. Intent-to-Swarm (describe goal → spawn agent swarm)
  3. Task queue via Kanban
  4. Outcome verification (auto-run build/test/lint)

PHASE 4: ADVANCED
  1. Agent Memory Mesh (per-workspace knowledge base)
  2. Session Recipes (save/replay multi-agent workflows)
  3. Agent Handoff Protocol
  4. Cross-Agent Review Gate
  5. Easy Agent Forking

PHASE 5: REMOTE & MOBILE
  1. SSH workspace parity (remote = local)
  2. Mobile web interface (monitor + respond)
  3. AFK mode with push notifications
```

### Non-Goals

- Full IDE features (no debugging, no IntelliSense) - use VS Code for that
- Replacing Claude Code CLI - we orchestrate it, not replace it
- Cloud hosting - local machines and owned servers only
- Team collaboration - single user focus
- Auto-responses - user answers questions, no auto-answering

### User Workflow (What Success Looks Like)

```
1. Open Matrix
2. Open workspace for project
3. Spin up 4-6 Claude Code instances with auto-created worktrees
4. Assign tasks: "fix this bug", "add this feature", "refactor this"
5. Monitor progress, provide guidance when asked
6. Commit all (unified message), merge with autonomous agent
7. Review merge result, approve
8. Worktrees cleaned up, ship
```

---

## HOW YOU ARE BEING USED

The user launches Claude Code instances **one after another in a relay**. Each instance:

1. Reads this file + STATUS.md + VISION.md + DECISIONS.md (MANDATORY)
2. Picks up the next task from where the last instance left off
3. Does that ONE task to perfection
4. Updates STATUS.md with exactly what was done and what's next
5. Commits
6. Done - the next instance takes over

**You are one link in a chain.** Your job is to leave the codebase better than you found it, with a clean handoff for the next instance. Don't try to do everything. Do your piece perfectly.

---

## ⛔ RULES (NON-NEGOTIABLE - ACTIVE ENFORCEMENT)

These rules exist because previous Claude Code instances broke them. Each rule is here because violating it caused real problems.

### Rule 1: Read the files FIRST

**Violation:** "Let me start implementing..." without reading STATUS.md.
**Why it's bad:** You build the wrong thing. The previous instance left specific instructions.
**Enforcement:** Your first message MUST reference STATUS.md contents. If it doesn't, you broke this rule.

### Rule 2: ONE task per session

**Violation:** "Now let me also add..." or "While I'm here, let me..."
**Why it's bad:** Half-finished features, messy code, context exhaustion.
**Enforcement:** Pick ONE task from STATUS.md. Do ONLY that task. When done, update STATUS.md and STOP. The next instance handles the next task.

### Rule 3: Interface-first design (ALWAYS)

**Violation:** Writing `pub fn` or `class FooService` without a trait/interface first.
**Why it's bad:** Tightly coupled code, untestable, can't swap implementations.
**Enforcement:** For EVERY new service/module:
```
Step 1: Define the trait/interface (what it does)
Step 2: Define the types it uses
Step 3: Write the implementation
Step 4: Wire into DI (AppContext or ServiceProvider)
```
If you catch yourself writing an implementation without a trait → STOP → write the trait first.

### Rule 4: Single responsibility (max 300 lines)

**Violation:** Files with 400+ lines doing multiple things.
**Why it's bad:** Unmaintainable, hard to understand, hard to test.
**Enforcement:** If a file exceeds 300 lines → split it. If a file does two things → split it NOW. `main.rs` is 10-20 lines, entry point ONLY.

### Rule 5: Dependency injection (NEVER hardcode)

**Violation:** `use crate::services::git::GitServiceImpl;` in consuming code.
**Why it's bad:** Can't test, can't swap, tight coupling.
**Enforcement:**
- Rust: `Arc<dyn GitService>` injected via constructor
- TypeScript: React Context, never direct import of implementation class

### Rule 6: Verify before claiming done

**Violation:** "Phase complete!" without running verification.
**Why it's bad:** Broken code shipped as "done". Next instance wastes time fixing it.
**Enforcement:** Before marking ANY phase complete, run 4 parallel verification agents:
```
Agent 1 - Build:         cargo build && cargo build --release && bun run build
Agent 2 - Test:          cargo test && cargo clippy -- -D warnings
Agent 3 - Functionality: Run daemon, verify feature works end-to-end
Agent 4 - Quality:       bun run lint && bun run typecheck && grep for TODO/FIXME
```
ALL FOUR must pass. No shortcuts.

### Rule 7: Update STATUS.md (ALWAYS)

**Violation:** Committing code without updating STATUS.md.
**Why it's bad:** Next instance doesn't know what you did or what's next. Work gets lost or duplicated.
**Enforcement:** Every commit that changes code MUST also update STATUS.md with:
- What was completed (specific, not vague)
- What the next task is
- Any blockers or decisions needed

### Rule 8: Use Bun (NEVER npm/yarn/pnpm)

**Violation:** `npm install`, `yarn add`, `npx`.
**Enforcement:** `bun install`, `bun add`, `bunx`. Always.

### Rule 9: Delete bad code, don't patch it

**Violation:** Adding workarounds, hacks, "temporary" fixes.
**Why it's bad:** Technical debt compounds. Each hack makes the next fix harder.
**Enforcement:** If code is wrong → delete it → rewrite it properly with interface-first design. The user explicitly does not care about preserving bad code.

### Rule 10: Don't build features outside current phase

**Violation:** "I noticed the SSH module could use..." when we're in Phase 1.
**Why it's bad:** Scope creep, unfinished core features, wasted context.
**Enforcement:** Check STATUS.md for current phase. Build ONLY what's in that phase. Nothing else.

---

## ARCHITECTURE PATTERNS

### Rust: Interface-First

```rust
// 1. FIRST: Define the trait
#[async_trait]
pub trait AgentManager: Send + Sync {
    async fn spawn(&self, config: AgentConfig) -> Result<AgentId>;
    async fn stop(&self, id: AgentId) -> Result<()>;
    async fn list(&self) -> Result<Vec<AgentInfo>>;
}

// 2. THEN: Implement it
pub struct AgentManagerImpl {
    terminal: Arc<dyn TerminalService>,  // Injected, never hardcoded
    git: Arc<dyn GitService>,            // Injected, never hardcoded
}
impl AgentManager for AgentManagerImpl { /* ... */ }

// 3. Wire in AppContext (composition root)
let manager: Arc<dyn AgentManager> = Arc::new(AgentManagerImpl::new(
    terminal.clone(), git.clone()
));
```

### TypeScript: Interface-First

```typescript
// 1. Define interface
interface IAgentService {
  spawn(config: AgentConfig): Promise<AgentId>;
  stop(id: string): Promise<void>;
  onStatusChanged(cb: (s: AgentStatus) => void): () => void;
}

// 2. Implement
class DaemonAgentService implements IAgentService {
  constructor(private daemon: IDaemonService) {}
}

// 3. Provide via React Context
<AgentContext.Provider value={agentService}>
```

### Error Handling

```rust
// Library crates: thiserror
#[derive(Debug, thiserror::Error)]
pub enum AgentError {
    #[error("Agent not found: {0}")]
    NotFound(AgentId),
    #[error("Spawn failed: {0}")]
    SpawnFailed(#[from] std::io::Error),
}

// Application code: anyhow
pub async fn run() -> anyhow::Result<()> { /* ... */ }
```

No `unwrap()` in production paths. Ever.

### Logging

```rust
use tracing::{info, warn, error, debug, instrument};

#[instrument(skip(self))]
pub async fn spawn_agent(&self, config: AgentConfig) -> Result<AgentId> {
    info!(name = %config.name, "Spawning agent");
    // ...
}
```

| Level | Use For |
|-------|---------|
| ERROR | Failures requiring attention |
| WARN  | Unexpected but recoverable |
| INFO  | Major operations (start, stop, create) |
| DEBUG | Detailed flow |
| TRACE | Message contents, raw data |

### Types Defined Once

- Rust domain types: `crates/protocol/`
- TypeScript types: `packages/shared-types/`
- NEVER duplicate type definitions. Import from the single source.

---

## TECH STACK (LOCKED - DO NOT CHANGE)

| Component | Technology |
|-----------|-----------|
| Frontend | Electron + React |
| Backend | Rust daemon (tokio) |
| IPC | stdio + JSON-RPC 2.0 |
| Database | SQLite (sqlx) |
| Terminal/PTY | portable-pty |
| SSH | russh |
| Git | gitoxide + git2 + CLI hybrid |
| Editor | Monaco |
| State | Zustand |
| Package manager | **Bun** (NEVER npm/yarn/pnpm) |
| Monorepo | Turborepo + Bun workspaces + Cargo workspace |

See DECISIONS.md for why each was chosen. Do NOT change without explicit user approval.

---

## PROJECT STRUCTURE

```
matrix/
├── CLAUDE.md              # YOU ARE HERE - read this fully
├── STATUS.md              # Progress tracker - your task assignment
├── VISION.md              # Full product vision - read every session
├── PLAN.md                # Detailed specs for each phase
├── DECISIONS.md           # Tech decisions with rationale
├── apps/
│   └── desktop/           # Electron + React
│       ├── electron/      # Main process (daemon lifecycle)
│       └── src/           # React renderer
│           ├── components/  # UI components by feature
│           ├── services/    # Interface → Implementation → Context
│           ├── hooks/       # Custom React hooks
│           ├── styles/      # CSS + design tokens
│           └── context/     # ServiceProvider (composition root)
├── crates/
│   ├── daemon/            # Rust daemon (IPC server, handlers, AppContext)
│   ├── protocol/          # Shared types + JSON-RPC definitions
│   ├── core/              # Shared domain logic
│   ├── agent/             # Claude Code process management
│   ├── git/               # Git + worktree operations
│   ├── terminal/          # PTY management
│   ├── db/                # SQLite database layer
│   ├── ssh/               # SSH connections
│   ├── lsp/               # Language server management
│   ├── filesystem/        # File operations
│   ├── kanban/            # Kanban board
│   └── notifications/     # Notification system
├── packages/
│   └── shared-types/      # TypeScript type definitions (SINGLE SOURCE)
├── turbo.json
├── package.json
└── Cargo.toml
```

---

## HOW TO WORK

### Session Flow

```
1. READ STATUS.md → find the next incomplete task
2. READ VISION.md → refresh your understanding of the product
3. READ DECISIONS.md → know the tech constraints
4. READ PLAN.md → read the specs for the current phase
5. TELL the user: phase, task, your plan
6. DO that ONE task to production quality:
   - Trait/interface defined FIRST
   - Implementation behind the interface
   - Wired into DI (AppContext / ServiceProvider)
   - Error handling with proper types (thiserror/anyhow)
   - Logging with tracing
   - Tests for core logic
   - Doc comments on public APIs
7. VERIFY it works:
   - cargo build (no errors, no warnings)
   - cargo test (passes)
   - cargo clippy (clean)
   - bun run typecheck (if frontend changed)
   - bun run lint (if frontend changed)
8. UPDATE STATUS.md:
   - What you completed (specifics)
   - What the next task is
   - Any blockers or decisions needed
9. COMMIT with descriptive message
10. STOP - do NOT start another task
```

### Session Handoff

When you finish, STATUS.md MUST clearly state:
- What you completed (with specifics, not vague summaries)
- What the next task is (specific task number from the phase)
- Any blockers or decisions needed
- Whether existing code needs rewriting to proceed

The next instance reads STATUS.md and picks up immediately. Clean handoff = no wasted context.

---

## CODE QUALITY STANDARDS

Every piece of code must be:

1. **Compilable** - zero errors, zero warnings
2. **Runnable** - actually executes without crashing
3. **Tested** - core logic has tests
4. **Documented** - public APIs have doc comments
5. **Logged** - tracing instrumentation on major operations
6. **Interface-first** - trait/interface defined before implementation
7. **Single-purpose** - one file, one job, max 300 lines
8. **Injected** - dependencies via DI, never hardcoded
9. **No unwrap()** - proper error handling everywhere

If existing code doesn't meet these standards, rewrite it.

---

## UI DESIGN

Linux/Ubuntu aesthetic - NOT macOS:

- Flat, utilitarian, GNOME-inspired
- No macOS colored window dots
- Dark theme with true blacks (#0a0a0a, #1c1c1c)
- Monospace fonts: Ubuntu Mono, DejaVu Sans Mono
- `user@host:~$` terminal prompt style
- Simple dot indicators, no animations or glow effects
- Minimal decorations

---

## PLATFORM

Windows 11 + Linux (Ubuntu). All code must work on both.

```rust
#[cfg(windows)]
fn get_shell() -> &'static str { "cmd.exe" }

#[cfg(unix)]
fn get_shell() -> &'static str { "/bin/sh" }
```

---

## COMMANDS

```bash
bun run dev                     # Start Electron dev mode
cargo run -p matrix-daemon      # Run Rust daemon
bun run build                   # Build frontend
cargo build --release           # Build daemon
cargo test                      # Rust tests
cargo clippy -- -D warnings     # Rust lint
bun run typecheck               # TypeScript types
bun run lint                    # TypeScript lint
bun install / bun add / bun remove   # ALWAYS Bun
```

---

## COMMIT STYLE

```
feat: Add agent process spawning with PTY management
fix: Handle agent crash detection on Windows
refactor: Extract worktree operations into git crate
test: Add integration tests for agent lifecycle
```

Commit after completing a logical unit. Always include STATUS.md update.

---

## WHAT NOT TO DO

- Skip reading STATUS.md, VISION.md, DECISIONS.md at session start
- Start coding before telling the user your plan
- Build multiple features in one session
- Write implementations without defining traits/interfaces first
- Mark phase complete without running 4 verification agents
- Leave STATUS.md outdated after making changes
- Patch bad code instead of rewriting it properly
- Add features not in the current phase
- Use npm/yarn/pnpm instead of Bun
- Ignore compiler or lint warnings
- Use unwrap() in production code
- Hardcode dependencies instead of injecting them
- Create files over 300 lines
- Duplicate type definitions across files
- Assume code works without testing it
- Start a new task when the current one isn't done
- Preserve bad code out of sentiment
- Say "I'll do it later" for any rule in this document

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sezaiarda)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sezaiarda)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
