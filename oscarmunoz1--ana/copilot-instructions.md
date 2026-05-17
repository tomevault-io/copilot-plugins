## ana

> The main workflow `workflows/whatsapp-conversational.workflow.json` (84KB) has its JavaScript code extracted into individual files under `workflow-nodes/`.

# Ana Project Instructions

## Workflow Code Editing Convention

The main workflow `workflows/whatsapp-conversational.workflow.json` (84KB) has its JavaScript code extracted into individual files under `workflow-nodes/`.

**When editing workflow Code node logic:**
1. Edit the corresponding `.js` file in `workflow-nodes/` (NOT the workflow JSON directly)
2. Run `npx tsx scripts/workflow-build.ts` to inject changes back into the workflow JSON

**When editing non-code parts** (connections, settings, non-code node parameters):
- Edit the workflow JSON directly

**If the workflow JSON is re-exported from n8n:**
- Run `npx tsx scripts/workflow-extract.ts` to re-sync the `.js` files

**Node file mapping:**
- Node name "Parse & Extract" → `workflow-nodes/parse-extract.js`
- Node name "Build Prompt" → `workflow-nodes/build-prompt.js`
- Slug rule: lowercase, non-alphanumeric replaced with `-`

**Never read the full 84KB workflow JSON just to edit a Code node.** Read the individual `.js` file instead.

---
> Source: [oscarmunoz1/ana](https://github.com/oscarmunoz1/ana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
