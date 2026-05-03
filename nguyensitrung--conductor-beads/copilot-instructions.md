## conductor-beads

> If a user mentions a "plan" or asks about the plan, and they have used the conductor extension in the current session, they are likely referring to the `conductor/tracks.md` file or one of the track plans (`conductor/tracks/<track_id>/plan.md`).

# Conductor Context

If a user mentions a "plan" or asks about the plan, and they have used the conductor extension in the current session, they are likely referring to the `conductor/tracks.md` file or one of the track plans (`conductor/tracks/<track_id>/plan.md`).

## Parallel Execution

Conductor supports parallel task execution for phases with independent tasks:

- Phases annotated with `<!-- execution: parallel -->` spawn sub-agents
- Tasks have `<!-- files: ... -->` for exclusive file ownership  
- Use `<!-- depends: taskN -->` for task dependencies within a phase
- State tracked in `parallel_state.json`

## Beads Integration

If `.beads/` directory exists alongside `conductor/`, this project uses Beads for persistent task memory. Check `conductor/beads.json` for integration config.

When Beads is enabled:
- Use `bd ready` to find tasks with no blockers
- Each Conductor track maps to a Beads epic
- Notes in Beads survive context compaction

**Beads Formula Commands (New!):**
- `/conductor:formula` - List and manage track templates (Beads formulas)
- `/conductor:wisp` - Create ephemeral exploration track (no audit trail)
- `/conductor:distill` - Extract reusable template from completed track

## Git Policy

**Important**: Conductor commits locally but never pushes automatically. Users decide when and how to push to remote repositories.

---
> Source: [NguyenSiTrung/Conductor-Beads](https://github.com/NguyenSiTrung/Conductor-Beads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
