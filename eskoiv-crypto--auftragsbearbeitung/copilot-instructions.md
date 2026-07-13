## auftragsbearbeitung

> > **Guiding Principle:** Virtue lies in Diligency.

# CLAUDE.md — elvinci JTL Auftragssteuerung

> **Guiding Principle:** Virtue lies in Diligency.

You are working on the **JTL Auftragssteuerung** tool for elvinci.de GmbH (Nürnberg).
This is an internal, German-language operations tool used by Backoffice & Fulfillment.

---

## Project Identity

- **Domain:** Order-processing workflow tool. Takes an order through Erfassung → Rechnung → Kommissionierung → AMM-Übergabe.
- **Output format:** Single-file HTML. Must remain a single, self-contained file deployable to a SharePoint folder. No build step required for the user. CDN imports for Chart.js / jsPDF are OK.
- **Primary user:** Dustin (Backoffice/Fulfillment lead) and Mirko (invoicing).
- **Data source:** `data/JTL_Rechnungstexte.xlsx` — 45 invoice-text templates (4 sheets). The HTML embeds these as `JTL_DATA`. **HTML and XLSX must always remain in sync.**

---

## Execution Rules

### Rule 1: Never Invent — Retrieve or Ask
- Data missing → ask. Don't fabricate column names, ZZ values, or workflow steps.
- Ambiguous → state ambiguity, propose resolution.
- Estimate → label `[ESTIMATED]` with method.
- Sources conflict → present both, don't silently pick.

### Rule 2: Confirm Before Irreversible Actions
**Confirm before:**
- Modifying the JTL_DATA matrix (45 entries — both HTML embed and XLSX)
- Changing public-facing invoice text (the `jtl_text` field — these go to customers)
- Restructuring state (S object) — many UI render paths depend on it
- Deleting any prior version files

**Proceed (with summary) for:**
- Internal CSS/UI tweaks
- Adding helper functions, cleanup, comments
- New non-breaking features

### Rule 3: Raise Concerns With Structure
```
⚠️ CONCERN: [one-line summary]
WHAT: [what's wrong or risky]
WHY IT MATTERS: [impact if ignored]
ALTERNATIVE: [other options]
RECOMMENDATION: [your suggested path]
```
No vague doubts. No silent passing of real issues.

### Rule 4: Methodical Execution
1. State plan before starting.
2. Execute in stages with checkpoints.
3. Summarize at the end.

For data changes: always re-run the HTML↔XLSX sync verification (script provided in `ROADMAP.md`).

### Rule 5: Debate Before Deciding
Cost/architecture decisions → present options + trade-offs + recommendation, then wait.
"Single-file vs build step" is a Rule-5 question if you want to introduce a build.

---

## Hard Constraints

- **Output is single-file HTML.** If you introduce a build step, the build's final artifact must remain a single self-contained `.html`.
- **JTL_DATA matrix has exactly 45 entries** (9 VORKASSE + 36 ZAHLUNGSZIEL across 4 versandart×terminwunsch sheets). Don't change shape without explicit user approval.
- **No counter dimension.** Removed in v3 (2026-05-04). Customers can only fall in payment terms, not rise.
- **Customer-facing text is English** (the `jtl_text` field). Internal UI/labels/todos are German. Code comments may be either.
- **No external network dependencies at runtime** beyond the existing CDN scripts (Chart.js, jsPDF). Tool must work offline once loaded.

---

## File & Output Standards

- **Filenames:** `YYYY-MM-DD_description.ext`, version suffix when iterating: `_v1`, `_v2`.
- **Code files:** UTF-8, LF line endings.
- **Date in artifacts:** show in headers. Easy version tracking.

---

## Working Language

- **UI labels, error messages, todos:** German
- **JTL invoice text (customer-facing):** English (it goes to international B2B customers)
- **Code:** English variable names, comments either language
- **Commit messages / changelog:** German

---

## Partner Context (don't break their assumptions)

| Partner | Notes |
|---|---|
| AMM Spedition | Primary fulfillment. Texts reference KOMMISSIONIERUNGSTERMIN, WERKTAG. |
| Customers (B2B EU) | Read the `jtl_text` field on their invoice. Tone: factual, professional. |
| Mirko (invoicing) | Runs the JTL Generator. Trust the matching logic — needs to find a template every time. |

---

## Error Protocol

When something fails:
1. **Stop.** Don't retry blindly.
2. **Report:** what was attempted, what failed, what error occurred.
3. **Diagnose:** likely cause.
4. **Propose:** recovery path.
5. **Wait** for direction.

---

## Compact Reference

```
DATA:     Retrieve or ask. Never invent. Label estimates.
CONFIRM:  Public-facing changes → yes. Internal cleanup → proceed + note.
CONCERN:  Structured: What/Why/Alternative/Recommendation.
EXECUTE:  Plan → Stages → Summarize.
DEBATE:   Options + trade-offs + recommendation. Then wait.
FAIL:     Stop → Report → Diagnose → Propose → Wait.
SYNC:     HTML JTL_DATA ↔ XLSX must always match. Verify after data changes.
```

---

**Principle:** Diligence over speed. Correctness over completion.

---
> Source: [eskoiv-crypto/Auftragsbearbeitung](https://github.com/eskoiv-crypto/Auftragsbearbeitung) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-13 -->
