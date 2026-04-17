## interactive-visualizers

> Project instructions for Claude Code in the interactive-visualizers repository.

# CLAUDE.md

Project instructions for Claude Code in the interactive-visualizers repository.

---

## Current Implementation Batch

**Checkpoint 2: Signaling Games & Compositionality**

This is the current implementation target. Do not implement other visualizer families until this is complete and approved.

**Status:** Ready to implement
**Spec:** `docs/visualizer-specs/signaling-games.md`
**Paper notes:** `docs/paper-notes/compositional-signal.md`
**Code repo:** github.com/DavidFreeborn/compositional-signals (inspect before implementing)

---

## Do-Not-Implement List

Do NOT implement these without explicit user approval:

| Family | Reason | Status |
|--------|--------|--------|
| QFT particles | No source paper | Spec-only |
| Norton's dome | No source paper, numerical artifact risk | Spec-only |
| Maxwell's demon | No source paper, phase-space complexity | Spec-only |
| LLM visualizers | Limited source support, overclaim risk | Defer |

These may be implemented with caution:
- Deep learning submodules (only source-supported ones)
- Zollman effect (from available sources only)

---

## Stop and Update Requirements

**After completing each visualizer family:**

1. STOP implementation
2. Update `docs/progress-report.md` with:
   - What was implemented
   - What tests pass
   - Any decisions made
   - Any source gaps discovered
3. Report completion to user
4. Wait for approval before starting next family

**Do not chain implementations without user checkpoint.**

---

## When to Use Subagents

Use subagents for:
- **scientific-auditor**: Before finalizing any visualizer spec or implementation that makes claims about physics, philosophy, or formal systems
- **viz-reviewer**: After completing view implementations, before considering a visualizer done
- **test-reviewer**: Before marking a visualizer complete, to verify test coverage

## When NOT to Use Subagents

Do NOT spawn subagents for:
- Reading individual files (use Read tool directly)
- Simple searches (use Grep/Glob directly)
- Straightforward code edits
- Documentation updates
- Tasks already in your context
- Quick lookups that don't need specialized review

**Prefer direct tool use over subagent delegation for simple tasks.**

---

## Project Overview

TypeScript/React library for interactive scientific visualizers. Visualizers must be scientifically rigorous, pedagogically strong, and visually elegant.

---

## Key Documents

Read these before making changes:
- `docs/architecture.md` - System architecture
- `docs/scientific-integrity.md` - Scientific standards (CRITICAL)
- `docs/visualizer-specs/signaling-games.md` - Current implementation spec
- `docs/progress-report.md` - Current status

---

## Source Authority

**Priority order:**
1. `docs/paper-notes/` - Use these first
2. `source_materials/paper_text/` - Paper extractions
3. `source_materials/papers/` - Original PDFs (spot-check only)

**Never invent paper-specific equations, assumptions, or claims without source support.**

---

## Scientific Status Labels

Every visualizer must be labeled as one of:
- **Exact model** - Faithful mathematical representation
- **Standard toy model** - Simplified using established formalisms
- **Conceptual analogy** - Illustrative, not literal
- **Theorem intuition builder** - Intuition for formal results

---

## Working Practices

### Before Implementing

1. Read the visualizer spec in `docs/visualizer-specs/`
2. Read relevant paper notes in `docs/paper-notes/`
3. Inspect any referenced code repositories
4. Confirm scientific status label is appropriate

### During Implementation

1. Separate model / simulation / rendering / content
2. Include "What This Shows" and "What This Does NOT Show" panels
3. Make all random processes seedable
4. Write tests for model logic

### After Implementation

1. Run tests
2. Update `docs/progress-report.md`
3. Report to user and STOP

---

## Forbidden Practices

1. Do not claim precision not present
2. Do not hide assumptions
3. Do not invent source-unsupported claims
4. Do not build fake mechanism visualizations
5. Do not skip scientific status labels
6. Do not implement families on the Do-Not-Implement list

---

## Package Structure

```
packages/
+-- core-ui/         # Shared UI components
+-- core-math/       # Math utilities
+-- core-sim/        # Simulation framework
+-- core-content/    # Content structures
+-- viz-signaling/   # Current implementation target
```

---

## Commands

```bash
npm run dev          # Start showcase app
npm run test         # Run tests
npm run lint         # Lint code
npm run typecheck    # Check types
npm run build        # Build all packages
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DavidFreeborn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
