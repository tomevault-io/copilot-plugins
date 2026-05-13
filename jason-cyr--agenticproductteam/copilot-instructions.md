## agenticproductteam

> AgenticPod is an agentic product lifecycle accelerator. You (Claude Code) are the UI. Users interact with you conversationally, and you drive the pipeline by calling library functions.

# AgenticPod — Claude Code Instructions

AgenticPod is an agentic product lifecycle accelerator. You (Claude Code) are the UI. Users interact with you conversationally, and you drive the pipeline by calling library functions.

## How It Works

AgenticPod runs a staged pipeline across four agent roles (PM → Research → Design → Engineering), producing structured artifacts (markdown + json) in `./artifacts/<project_id>/`. Each stage calls an LLM to generate content based on prior artifacts.

## Calling AgenticPod Functions

Because this is an ESM project with top-level await, use script files in `./scripts/` rather than `npx tsx -e`. Write a short `.ts` script, run it with `npx tsx scripts/your-script.ts`, then delete it after.

Example pattern:
```ts
// scripts/run-stage.ts
import { runStage } from '../src/index.js';
const r = await runStage('snackquest', 'pm');
console.log(JSON.stringify(r, null, 2));
```
```bash
npx tsx scripts/run-stage.ts
```

## Commands (what users will ask you to do)

### Start a new project
When the user describes a product idea, init a project:
```ts
import { initProject } from '../src/index.js';
const r = await initProject('THE IDEA TEXT', 'optional-project-id');
console.log(JSON.stringify(r, null, 2));
```
Then show the user the generated `idea.json` and ask if they want to refine it before proceeding.

### Run the next stage
Run one stage at a time. After each stage, present:
1. What artifacts were created/updated
2. A brief summary of the content
3. Any quality issues
4. Ask: "Ready to proceed to [next stage]?"

```ts
import { runStage } from '../src/index.js';
const r = await runStage('PROJECT_ID', 'STAGE');
console.log(JSON.stringify(r, null, 2));
```

Stages in order: `pm` → `research` → `design` → `engineering`

### Check project status
```ts
import { getProjectStatus } from '../src/index.js';
console.log(JSON.stringify(getProjectStatus('PROJECT_ID'), null, 2));
```

### Run quality checks
```ts
import { runQualityChecks } from '../src/index.js';
console.log(JSON.stringify(runQualityChecks('PROJECT_ID'), null, 2));
```

### Pull Figma snapshot
After a designer has made changes in Figma, save a snapshot:
```ts
import { saveFigmaSnapshot } from '../src/index.js';
saveFigmaSnapshot('PROJECT_ID', SNAPSHOT_DATA);
```

### Run iteration
After pulling a new Figma snapshot, update specs from the changes:
```ts
import { runIteration } from '../src/index.js';
const r = await runIteration('PROJECT_ID');
console.log(JSON.stringify(r, null, 2));
```

## Stage Gate Flow (Mode A — default)

This is the expected conversational pattern:

1. **User provides idea** → You run `initProject`, show `idea.json`, ask to refine or proceed
2. **PM stage** → You run `runStage(id, 'pm')`, read and summarize `PRD.md`, show quality report, ask to proceed
3. **Research stage** → You run `runStage(id, 'research')`, summarize `Research.md`, ask to proceed
4. **Design stage** → You run `runStage(id, 'design')`, summarize `PrototypeSpec.md`, then **build coded HTML prototype screens** in `./artifacts/<project_id>/screens/`. Present screens for review. Ask if user wants to push to Figma or proceed.
5. **Engineering stage** → You run `runStage(id, 'engineering')`, summarize `TechSpec.md` and `Backlog.md`

Between each stage, **always ask for confirmation** before advancing. Show what files were written and a brief content summary.

**After every stage**, update `./artifacts/<project_id>/CONTEXT.md` with the current project state, completed stages, key decisions, next steps, and any setup notes (API keys, MCP config, etc.) so a new Claude Code session can pick up seamlessly.

## Prototype Workflow (Code-First)

After the design stage completes, **always build coded HTML prototype screens** before involving Figma:

### Step 1: Build HTML Screens
1. Read `FigmaLink.json` and `PrototypeSpec.md` for the frame structure, screen specs, and microcopy
2. Create HTML files in `./artifacts/<project_id>/screens/` — one per frame, plus a shared `styles.css`
3. Style screens to match the target platform (e.g., Slack dark theme for Slack apps, Material for Android, etc.)
4. Include all UI elements, states, copy, and interactions described in `PrototypeSpec.md`
5. Name files with numbered prefixes matching `FigmaLink.json` frame order (e.g., `01-slash-command.html`)
6. Present the screens to the user for review. They can open any HTML file in a browser to preview.

### Step 2: Send to Figma (Optional)
If the user wants to refine designs in Figma and `config/figma.mcp.json` has a file key:

1. Use the **Figma MCP** (`figma-remote`) to push the coded screens to Figma as editable frames ("Code to Canvas")
   - The MCP server is at `https://mcp.figma.com/mcp` — authenticate via `/mcp` if needed
   - Note: The Figma REST API is **read-only** for design content. Frame creation requires the MCP tools or Figma desktop app's local MCP server (`127.0.0.1:3845`)
2. Save the frame IDs/metadata using `saveFigmaSnapshot`

### Step 3: Figma Round-Trip (if Figma is used)
1. User says "I've updated the Figma designs"
2. Use Figma MCP to read the current frames and text layers
3. Save as `FigmaSnapshot.json` using `saveFigmaSnapshot`
4. Run `runIteration` to diff and update specs
5. Show what changed and what was updated

### Figma MCP Setup Notes
- **Remote server:** `claude mcp add --transport http figma https://mcp.figma.com/mcp`
- **Auth:** Run `/mcp` in Claude Code, select `figma-remote`, click Authenticate (OAuth flow)
- **Desktop server (alternative):** Requires Figma desktop app with Dev Mode MCP enabled at `127.0.0.1:3845`
- **REST API token:** `FIGMA_ACCESS_TOKEN` env var — generate in Figma Settings > Security > Personal access tokens (useful for reading file metadata, but cannot create frames)

## Artifact Locations

All artifacts live in `./artifacts/<project_id>/`:
- `idea.json` — Structured project idea
- `PRD.md` — Product requirements
- `Research.md` — Research plan
- `PrototypeSpec.md` — Design prototype spec
- `FigmaLink.json` — Figma frame structure to create
- `FigmaSnapshot.json` — Latest Figma state
- `TechSpec.md` — Technical specification
- `Backlog.md` — Prioritized work items
- `IterationLog.md` — Change log
- `QualityReport.json` — Quality check results
- `pipeline-state.json` — Pipeline state
- `screens/` — Coded HTML prototype screens (one per frame + shared CSS)
- `CONTEXT.md` — Session handoff notes for picking up work in a new instance

## LLM Configuration

Config is in `config/llm.json`. The user needs `ANTHROPIC_API_KEY` or `OPENAI_API_KEY` set depending on the provider configured.

## Context File (CONTEXT.md)

Every project must have a `CONTEXT.md` file at `./artifacts/<project_id>/CONTEXT.md`. This is the **session handoff document** — it lets a new Claude Code instance pick up exactly where the last one left off.

### When to update CONTEXT.md
- **After `initProject`** — create it with the project idea and initial setup
- **After every stage completes** — update with new artifacts, summaries, and next steps
- **After any significant action** — Figma sync, iteration, quality checks, manual edits, prototype builds
- **Before ending a session** — ensure it reflects the latest state

### What to include
- **Project status** — project ID, current stage, completed stages
- **Key artifacts** — list of files with brief descriptions of what's in each
- **Environment setup** — API keys needed, MCP servers configured, Figma file keys
- **Decisions made** — any user choices or refinements during the session
- **Next steps** — what should happen next when work resumes
- **Known issues** — anything broken, pending, or needing attention

### On session start
When picking up an existing project, **always read `CONTEXT.md` first** to understand where things stand before taking any action.

## Important Notes

- Always read artifacts after writing them to present accurate summaries to the user
- If a stage fails or produces low quality output, tell the user and ask how to proceed (re-run, edit manually, skip)
- The user can always edit artifacts directly in their editor — the pipeline will pick up changes on the next run
- Never auto-advance stages without asking in Mode A (low autonomy)
- Always keep `CONTEXT.md` up to date — it is the source of truth for session continuity

---
> Source: [Jason-Cyr/AgenticProductTeam](https://github.com/Jason-Cyr/AgenticProductTeam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
