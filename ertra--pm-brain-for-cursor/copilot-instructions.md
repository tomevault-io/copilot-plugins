## product-sense

> >-


# Product sense (coaching layer)

Before generating a **product brief** or **PRD** in this repo, follow **[`PRODUCT-RULES.md`](../../PRODUCT-RULES.md)**.

1. Read `PRODUCT-RULES.md` when the user asks for a brief, PRD, one-pager, feature pitch, or similar — including ad-hoc prompts (not only slash commands).
2. Apply the **"When the gate applies"** table in that file:
   - **Product brief** → run the braindump gate (3–5 product-sense questions + sufficiency checklist) **before** opening templates or the `create-product-brief` skill's research step.
   - **PRD** → run the gate **only if** no brief exists (`01-product-brief.md` in the feature hub **or** an explicit brief path from the user). Otherwise proceed with the doc skill.
   - **IFA, support article, competitive landscape, interview-frameworks** → no braindump gate.
3. Do **not** open `02-templates/` or write structured outlines until the sufficiency checklist passes or the user says **"skip braindump"** (log the override per `PRODUCT-RULES.md`).
4. When a doc skill is invoked (`create-product-brief`, `create-prd`), follow that skill's Step 0.5 if present — it implements this gate in detail.

Doc pack workflow (order, feature hubs) remains in **[`AGENTS.md`](../../AGENTS.md)** and **[`pm-brain-doc-workflow.mdc`](pm-brain-doc-workflow.mdc)**.

---
> Source: [ertra/pm-brain-for-cursor](https://github.com/ertra/pm-brain-for-cursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
