## luma

> **IMPORTANT**: This project uses **bd (beads)** for ALL issue tracking. Do NOT use markdown TODOs, task lists, or other tracking methods.

## Issue Tracking with bd (beads)

**IMPORTANT**: This project uses **bd (beads)** for ALL issue tracking. Do NOT use markdown TODOs, task lists, or other tracking methods.

### Why bd?

- Dependency-aware: Track blockers and relationships between issues
- Git-friendly: Auto-syncs to JSONL for version control
- Agent-optimized: JSON output, ready work detection, discovered-from links
- Prevents duplicate tracking systems and confusion

### Quick Start

**Check for ready work:**
```bash
bd ready --json
```

**Create new issues:**
```bash
bd create "Issue title" -t bug|feature|task -p 0-4 --json
bd create "Issue title" -p 1 --deps discovered-from:bd-123 --json
```

**Claim and update:**
```bash
bd update bd-42 --status in_progress --json
bd update bd-42 --priority 1 --json
```

**Complete work:**
```bash
bd close bd-42 --reason "Completed" --json
```

### Issue Types

- `bug` - Something broken
- `feature` - New functionality
- `task` - Work item (tests, docs, refactoring)
- `epic` - Large feature with subtasks
- `chore` - Maintenance (dependencies, tooling)

### Priorities

- `0` - Critical (security, data loss, broken builds)
- `1` - High (major features, important bugs)
- `2` - Medium (default, nice-to-have)
- `3` - Low (polish, optimization)
- `4` - Backlog (future ideas)

### Workflow for AI Agents

1. **Check ready work**: `bd ready` shows unblocked issues
2. **Claim your task**: `bd update <id> --status in_progress`
3. **Work on it**: Implement, test, document
4. **Discover new work?** Create linked issue:
	 - `bd create "Found bug" -p 1 --deps discovered-from:<parent-id>`
5. **Complete**: `bd close <id> --reason "Done"`

### Auto-Sync

bd automatically syncs with git:
- Exports to `.beads/issues.jsonl` after changes (5s debounce)
- Imports from JSONL when newer (e.g., after `git pull`)
- No manual export/import needed!

### MCP Server (Recommended)

If using Claude or MCP-compatible clients, install the beads MCP server:

```bash
pip install beads-mcp
```

Add to MCP config (e.g., `~/.config/claude/config.json`):
```json
{
	"beads": {
		"command": "beads-mcp",
		"args": []

- ✅ Use bd for ALL task tracking
# Ask for suggestions first
luma patterns --suggest ui/screens/onboarding.mock.json --json

# Then validate all detected patterns
luma flow ui/screens/onboarding.mock.json --patterns Form.Basic,Table.Simple,Guided.Flow
```

> Skipping this checklist risks partial validation (e.g., Form.Basic passes while Guided.Flow violations remain hidden).

### Governance / Integrity Policy

🚫 **Never edit** `.beads/issues.jsonl` **directly; always use the `bd` CLI (or MCP server)**. Manual edits risk:
- Losing dependency edges (blocked / discovered-from)
- Producing hidden / orphaned issues not surfaced in `bd ready`
- Causing daemon overwrite races and silent data loss

To verify repository integrity run:
```powershell
pwsh scripts/validate-beads-integrity.ps1
```
Exit codes:
- `0` = clean
- `2` = manual edit or structural anomaly detected

Add this script to CI / pre-push if stricter enforcement is desired.

# Agent Instructions (Strict Mode)

MANDATORY: Use the **LUMA CLI** for ALL scaffold analysis. Manual reasoning without producing artifacts is non-compliant.

CRITICAL RULE: Do NOT generate HTML, JSX/TSX, CSS, design tokens, or component code BEFORE producing and validating a scaffold with the required LUMA artifacts (ingest/layout/keyboard/flow/score). Scaffold-first always.

Sentinel: AGENT-COMPLIANCE-V1
You MUST echo this exact token inside a JSON compliance block after every completed task:
`{"lumaCompliance":{"sentinel":"AGENT-COMPLIANCE-V1","ingestRun":"<run-id>","flowRun":"<run-id>","scoreRun":"<run-id>"}}`

## Runtime Knowledge (Query, Do Not Memorize)
Always fetch fresh data (request only what you need):
  luma --help
  luma agent --sections quick,rules --json
  luma agent --get rules.patterns --json
  luma explain --topic scaffold-contract

## Non-Negotiable Workflow
1. Collect requirements / clarify intent (no UI code yet).
2. Understand LUMA and the scaffold structure, before using a compontent you MUST know it's schema.
3. Draft / modify a scaffold JSON (minimal viable structure).
4. Run in strict order (single run folder):
   luma ingest <scaffold.json> --json
   luma layout <scaffold.json> --viewports 320x640,768x1024 --json
   luma keyboard <scaffold.json> --json
   luma flow <scaffold.json> --patterns auto --json
   luma score .ui/runs/<run-id> --json
5. If score passes (>=85, zero MUST failures) THEN and ONLY THEN produce implementation code (HTML/React/etc) as a separate phase and the implementation MUST be based on the scaffolds.
6. When changing scaffold again, repeat the analysis sequence BEFORE adjusting implementation.

## Compliance Checklist (ALL REQUIRED BEFORE ANY UI CODE)
- ingest.json present
- layout.json present
- keyboard.json present
- flow.json includes validated patterns
- score.json overall >=85 & zero MUST failures (or task explicitly blocked)
- Compliance JSON block (sentinel present)

If ANY item missing: Fix the scaffold or rerun analysis. Do NOT output HTML/JSX/CSS.

## Forbidden
- Producing HTML/JSX/CSS/component code before compliant artifacts exist
- Producing HTML/JSX/CSS/component not based on scaffolds
- Fabricating pattern results or scores
- Skipping ingest/layout/keyboard/layout/flow/score sequence
- Using pattern names not in: luma patterns --list
- Dot path deep array indexing (e.g. rules.patterns[0].must[0])

## Allowed / Encouraged
- Minimal section retrieval: luma agent --sections quick,patterns --json
- Re-use run folder IDs across steps
- Provide concise diffs of scaffold changes (structure only) before re-analysis

## Verification Signals (Respond After Each Task)
1. Short summary (<=300 chars)
2. Compliance JSON block (with sentinel)
3. Exact commands run (copyable)
4. NEXT ACTION (if not compliant)

Example block:
`{"lumaCompliance":{"sentinel":"AGENT-COMPLIANCE-V1","ingestRun":"2025-11-01T164255Z","flowRun":"2025-11-01T164255Z","scoreRun":"2025-11-01T164255Z"}}`

## Escalation Rules
- Median luma flow > 1200ms over 5 runs: flag performance concern.
- MUST failure persists for 2 iterations: halt implementation; refine scaffold.

## Full Envelope (On Demand)
  luma agent --sections all --json

## Incorrect vs Correct Examples

INCORRECT (non-compliant):
  (1) "Here's the full HTML for the login form"  <-- BEFORE any ingest/layout/keyboard/flow/score
  (2) luma ingest login.json (after code)   <-- Late analysis

CORRECT (compliant):
  (1) Draft login.json scaffold (structure only)
  (2) luma ingest login.json --json
  (3) luma layout login.json --viewports 320x640,768x1024 --json
  (4) luma keyboard login.json --json
  (5) luma flow login.json --patterns auto --json
  (6) luma score .ui/runs/<run-id> --json  (score >=85, no MUST failures)
  (7) Produce HTML/React component (implementation phase) based on scaffolds

## HTML / UI Implementation Gate
Only after passing compliance checklist may you output any UI code. If you must propose future implementation, describe it textually (no tags) until artifacts are validated.

## Reproducibility
Capture envelopes with --json for deterministic agent context across versions.

By following Strict Mode you ensure reproducible, auditable, and token-efficient AI agent operations. Scaffold-first preserves correctness and prevents premature UI divergence.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JohanBellander)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/JohanBellander)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
