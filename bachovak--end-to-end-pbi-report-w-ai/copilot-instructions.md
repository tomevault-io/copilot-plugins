## end-to-end-pbi-report-w-ai

> A single-issue Power BI portfolio project (PBIP) built to mirror a LEGO Business

# Project: LEGO Supplier OTIF Dashboard

A single-issue Power BI portfolio project (PBIP) built to mirror a LEGO Business
Intelligence Manager Specialist role (Supplier Services, Business Service Organization).
It tracks supplier **On-Time-In-Full (OTIF)** reliability on one governed semantic model
and one report page.

## Layer ownership — use the right tool for each layer

- **Semantic model, TMDL, DAX, governance, Fabric** → data-goblin plugins
  (`pbip`, `tmdl`, `connect-pbid`, the `tabular-editor` suite incl. `bpa-rules`, `fabric-cli`).
- **Report / PBIR / visuals / theme** → the `pbir` CLI (pbir.tools) and its report skills.
  The semantic-model tools CANNOT edit report pages; all report work goes through `pbir`.
- **Domain modelling conventions** → the `otif-conventions` skill.
- **DAX style and the canonical measure library** → the `dax-style` skill.

## Source-of-truth design brief

- `/context/OTIF_Dashboard_Spec.html` — the one-page spec: question → visual → measure → action,
  plus the full measure library. Treat this as the design brief for the report.
- `/context/LEGO_SupplierServices_Theme.json` — the Power BI theme (LEGO palette, red reserved
  for alerts). Apply via `pbir` theme management.

## Governed OTIF definition (single, do not vary)

A PO line is **OTIF** when it is received on or before the promised date plus the
**On-Time Window** tolerance AND the received quantity is at least the ordered quantity
times the **In-Full Threshold**. Defaults: On-Time Window = 2 days, In-Full Threshold = 1.0.

## Hard rules

1. **Back up before bulk edits.** Run `pbir backup` on the report folder before large changes.
2. **Validate after every mutation.** Run `pbir validate` after any report edit; for the model,
   execute a query to confirm it still loads.
3. **Never trust generated DAX un-executed.** After writing or changing a measure, run it via
   `connect-pbid` (or the modeling MCP) and check the result against an expected value before moving on.
4. **One OTIF definition.** All on-time / in-full logic must reference the two What-If parameters,
   never hard-coded numbers.
5. **Small commits, reviewed diffs.** Commit in steps; review TMDL diffs as if they were code.

## Licensing note

The `data-goblin/power-bi-agentic-development` plugins are GPL-3.0 with a re-use restriction,
and `pbir.tools` is proprietary. Use them to build, but do not copy or paraphrase their content
into any course, product, or tool of mine.

---
> Source: [bachovak/end-to-end-pbi-report-w-AI](https://github.com/bachovak/end-to-end-pbi-report-w-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
