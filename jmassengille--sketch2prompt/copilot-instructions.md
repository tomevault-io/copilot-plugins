## sketch2prompt

> > "I build tools that help humans define systems so AI can stop guessing."

# sketch2prompt

## Purpose

> "I build tools that help humans define systems so AI can stop guessing."

Sketch2Prompt is a **system definition tool** for software architecture. It transforms a builder's mental model into an explicit, structured specification that **constrains** downstream AI behavior.

The core problem it solves: **underspecified intent**. AI coding agents hallucinate structure, invent inconsistent conventions, and degrade coherence over time—not because models are weak, but because they're given ambiguous input.

### What Sketch2Prompt IS

- A **semantic boundary-setting tool** that defines what exists, why it exists, and where responsibilities lie
- An **intent-freezing mechanism** that captures architectural decisions before entropy sets in
- A **constraint generator** that produces specifications AI agents can reliably follow

### What Sketch2Prompt IS NOT

- A diagramming tool for documentation
- An architecture or code generator
- A framework prescriber or implementation enforcer
- A performance optimizer or correctness validator

**Its job ends once structure and intent are clear.** Everything else is downstream.

### Success Criteria

Sketch2Prompt succeeds when:

- AI agents stop guessing architecture
- Follow-up prompts become **shorter**, not longer
- Refactors feel safer instead of scarier
- New features slot into existing structure instead of creating sprawl
- Output feels like: *"This system already has rules."*

### Philosophical Stance

- Humans are better at defining intent
- Agents are better at executing within constraints
- Systems degrade without explicit boundaries

**Freeze intent early**, before entropy sets in.

### What This Is NOT (Important)

Sketch2Prompt is **not a magic-box SaaS**. It doesn't generate code or claim to build your app for you.

It's a **context-engineering bootstrap** — a professional-grade starting point with best practices baked in. Developers still build their own tool; they just start with guardrails instead of a blank slate.

### Core Guardrails (What the Output Provides)

| Guardrail | What It Prevents |
|-----------|------------------|
| **Version anchoring** | Hallucinated/outdated package versions — the #1 cause of AI code rot |
| **Library-first policy** | Custom solutions when existing dependencies solve the problem |
| **Repo structure** | Spaghetti file organization, unclear boundaries |
| **Modularity constraints** | God functions, 1000-line files, deep nesting |
| **START.md bootstrap** | AI proceeding without confirming user intent |

These aren't suggestions — they're the foundation that makes AI-assisted development actually work.

---

## ⚠️ Critical Distinction: Dev Environment vs Output Artifacts

This project has a **meta-level complexity**: we're building an AI-instruction generator while working within an AI-instructed environment. These two contexts must never be conflated.

### Dev Environment (This Workspace)

Files that govern agent behavior **in this codebase**:

| File | Purpose |
|------|---------|
| `~/.claude/CLAUDE.md` | Global user instructions |
| `./CLAUDE.md` | Project-specific instructions (this file) |
| `.claude/STATUS.md` | Current development state |
| `.claude/phases/*.md` | Implementation phase guides |

**These are for the agent working on sketch2prompt itself.**

### Output Artifacts (Generated for Downstream Users)

Files that sketch2prompt **generates** for a builder's new project:

| File | Purpose |
|------|---------|
| `[OUTPUT] START.md` | Bootstrap protocol with confirmation gates |
| `[OUTPUT] PROJECT_RULES.md` | System constitution for downstream agent |
| `[OUTPUT] AGENT_PROTOCOL.md` | Workflow guidance for downstream agent |
| `[OUTPUT] specs/*.md` | Component specifications (Markdown + XML tags) |
| `[OUTPUT] diagram.json` | Re-import capability |

**These are for a different agent (Cursor, Windsurf, etc.) working on a different project that doesn't exist yet.**

### Naming Convention

When discussing output artifacts, always use the `[OUTPUT]` prefix or terms like "generated", "blueprint", or "downstream":

- ✓ "The `[OUTPUT] PROJECT_RULES.md` should include..."
- ✓ "The generated AGENT_PROTOCOL needs..."
- ✓ "Downstream agents read specs/*.yaml to..."
- ✗ "CLAUDE.md should tell agents..." (ambiguous — which one?)

### Subagent Guidance

When delegating to subagents, explicitly state:

> "You are modifying **generators** in `src/core/template-generator/` that produce output for downstream projects. The output files (`PROJECT_RULES.md`, `AGENT_PROTOCOL.md`, `specs/*.yaml`) are NOT instructions for you — they are templates that will guide a different AI assistant in a different codebase."

---

## Output Format

The primary deliverable is structured specifications that AI coding assistants (Cursor, Windsurf, Claude) can consume:

1. **Architecture Overview** - Components grouped by type with clear responsibilities
2. **Responsibility Boundaries** - What exists, what's allowed, what's forbidden
3. **Build Order** - Recommended implementation sequence by component type
4. **Component Guidance** - Per-component notes and watch-fors

**Key design decision:** Edges are visual-only. Relationships are implied by component metadata, not parsed from canvas connections. This keeps the specification deterministic and focused on boundaries rather than flow.

---

## Tech Stack

- npm, Vite 7, React 19, TypeScript (strict)
- @xyflow/react 12 (React Flow) for canvas
- Zod for schema validation
- Tailwind CSS v4 for styling
- Zustand + zundo for state with undo/redo

## Commands

- `npm run dev` - Start dev server
- `npm run build` - Production build
- `npm run lint` - ESLint check
- `npm run format` - Prettier format

## Architecture

```
/src
  /components - React components (Canvas, Inspector, Toolbar, ExportDrawer)
  /core       - Business logic (model, schema, export/import)
  /app        - App shell and routing
  /styles     - Global styles
```

## Key Patterns

- React Flow handles all canvas state; our store syncs to it
- Zod schemas validate all I/O boundaries (import/export)
- Export functions are **pure transformations** (diagram → structured spec)
- No external API calls—output is consumed by external tools, not generated by them

## Testing

- Vitest for unit tests on `/core` functions
- Focus on export/import round-trip integrity
- Schema validation coverage

---

## Version & Pattern Enforcement

### Minimum Versions (enforce strictly)

| Package | Min Version | Rationale |
|---------|-------------|-----------|
| React | 19.x | Use React 19 features (Actions, useOptimistic, use()) |
| Vite | 7.x | ESM-only, baseline-widely-available target |
| @xyflow/react | 12.x | Dark mode, SSR support, new API |
| Tailwind CSS | 4.x | CSS-first config, @theme directive |
| Node.js | 20.19+ | Required by Vite 7 |

### Modern Patterns (required)

**React 19:**
- Use `use()` for async data in render
- Use `useOptimistic` for optimistic UI updates
- Use `useTransition` for non-urgent updates
- NO: class components, legacy context, string refs

**Tailwind v4:**
- Use CSS-first configuration (`@theme` in CSS, not JS config)
- Use `@import "tailwindcss"` (not @tailwind directives)
- NO: tailwind.config.js (unless absolutely necessary)

**Vite 7:**
- Target is `baseline-widely-available`
- ESM imports only (no CJS require)

**TypeScript:**
- Strict mode enabled
- Use `satisfies` for type narrowing with inference
- NO: `any`, enums (use const objects), namespaces

### Before Adding Dependencies

1. Check if functionality exists in React 19 or native CSS/Tailwind v4
2. Verify package supports React 19 + Vite 7
3. Prefer packages with ESM exports

---

## Project Management

### Status Tracking

`.claude/STATUS.md` tracks current phase, milestone, and progress. **Update after every milestone.**

### Phase Documentation

Implementation phases live in `.claude/phases/`:
- `001-foundation.md` - Bootstrap, Types, Store
- `002-canvas.md` - React Flow, Custom Nodes
- `003-ui-panels.md` - Toolbar, Inspector
- `004-export.md` - Prompt, JSON, Drawer
- `005-polish.md` - Undo/Redo, Theme, Tests

Reference the relevant phase file before starting work.

### Session Recovery (Post-Compact Protocol)

After `/compact` or starting a new session, **immediately**:

1. **Read `.claude/STATUS.md`** — Contains current plan, active milestone, checkpoint state
2. **Read the active plan file** — Referenced in STATUS.md (e.g., `.claude/plans/output-spec-alignment.md`)
3. **Check todo list** — Todos persist through compaction; shows granular progress
4. **Re-read this file** — Refresh on project patterns and the OUTPUT vs dev env distinction

**You are the ORCHESTRATOR after recovery.** Do not:
- Jump into implementation without context
- Assume you remember the plan details
- Skip the checkpoint protocol

**Subagent context reminder:** When delegating after recovery, always include:
> "You are modifying **generators** in `src/core/template-generator/` that produce output for downstream projects. The output files (`PROJECT_RULES.md`, `AGENT_PROTOCOL.md`, `specs/*.yaml`) are NOT instructions for you — they are templates for a different AI in a different codebase."

### Active Plans

Plans live in `.claude/plans/`. Current active plan is always referenced in STATUS.md.

| Plan | Status | Description |
|------|--------|-------------|
| `output-spec-alignment.md` | **Active** | Align generators with output spec (M0-M5) |

---

## Git Commits

- Use conventional commits format (`feat:`, `fix:`, `docs:`, etc.)
- Keep commits lean and professional
- **NO AI attribution** - no "Generated with Claude" or "Co-Authored-By" footers
- Focus on what changed, not how it was created


---

## Frontend Standards (REQUIRED)

### Design System Tokens

**NEVER hardcode colors.** Use CSS variables from `src/styles/app.css`:

| Wrong | Correct |
|-------|---------|
| `#0a0f1a` | `var(--color-blueprint-bg)` |
| `#00d4ff` | `var(--color-blueprint-accent)` |
| `bg-slate-900` | Design system token or `bg-[var(--color-bg)]` |

### Responsive Requirements

All modals/overlays MUST:
- Use `overflow-y-auto` for scrollable content
- Set `max-h-[80vh]` or similar viewport constraints
- Work at 320px, 768px, 1024px widths
- Never cause horizontal scroll

### Utility Classes

Use design system utilities from `app.css`:
- `card-hover` - Card lift effect
- `input-focus` - Input focus ring
- `label-sm` - Mono uppercase labels
- `btn-hover` - Button hover states

### Verification Before Completion

Before marking any frontend task complete:
- [ ] No hardcoded hex colors
- [ ] Responsive scroll handling present
- [ ] Design system tokens used
- [ ] Tested at mobile viewport (320px)

---

## Subagent Orchestration Protocol

**Critical:** Use custom agents from `.claude/agents/`. They have project-aware prompts built in.

### Custom Agents (USE THESE)

| Agent | Model | Purpose | When to Use |
|-------|-------|---------|-------------|
| `indexer` | haiku | Package version verification, code flow mapping | Research phase, version checks, finding files |
| `planner` | opus | Implementation design, pattern research | Before non-trivial implementations |
| `code-reviewer` | sonnet | Quality verification, verified findings only | After implementations |
| `design-system-enforcer` | haiku | Quick design token check | Pre-commit, fast validation |

### Agent Selection

```
Research/Indexing → indexer (fast, no opinions)
Planning         → planner (thorough, pattern-aware)
Implementation   → You (orchestrator) or delegate with constraints below
Review           → code-reviewer (verified findings only)
```

### When Delegating Implementation (Not Using Custom Agent)

If spawning a general agent for implementation, include:

```
READ FIRST:
- CLAUDE.md (project conventions)
- [relevant source files]

CONSTRAINTS:
- Functions <50 lines, files <300 lines
- No `any`, no hardcoded colors
- Use existing patterns
- Escalate if uncertain

DEFINITION OF DONE:
- [ ] [specific outcome]
- [ ] Lint passes
```

### Frontend Work

Frontend constraints are in this file (see "Frontend Standards" section). The `code-reviewer` agent verifies compliance. No separate frontend agent needed.

### Generator Work Reminder

When delegating template generator work, add:

> "You are modifying **generators** in `src/core/template-generator/`. The output files are templates for a different AI in a different codebase — not instructions for you."

### Workflow

1. **Research**: Use `indexer` for package versions, code mapping
2. **Plan**: Use `planner` for non-trivial work
3. **Implement**: Orchestrator or constrained delegate
4. **Review**: Use `code-reviewer` (returns VERIFIED findings only)

### Anti-Patterns

- Using generic Task when custom agent exists ❌
- Skipping code-reviewer after implementation ❌
- Vague prompts without READ FIRST ❌
- Assuming subagent knows conventions ❌

---
> Source: [jmassengille/sketch2prompt](https://github.com/jmassengille/sketch2prompt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
