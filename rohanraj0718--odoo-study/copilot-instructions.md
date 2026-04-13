## odoo-study

> Use when neither local docs nor web search resolves the question.

# Rohan Raj — Odoo Business Analyst Project

## Agent Bootstrap Instructions

> **Read this first. Every session. Before any task.**

1. Read `.agents/skills/odoo-ba.md` completely before responding to any message.
2. This applies regardless of how the task is phrased — question, command, or request.
3. If the file cannot be found, stop and report the missing path rather than proceeding
   without context.
---

## Source Routing Rules for Odoo Questions

> **High priority. Apply before answering any Odoo-related question.**

Do NOT load the entire documentation repository for every query — that is slow and
expensive. Use targeted lookup only.

### Tier 1 — Local Docs (default for Odoo questions)
Search the local `odoo-official-docs` repo at branch `19.0` using specific keywords
from the question. **Primary search location: `odoo-official-docs\content\`** — this
is where all readable documentation lives. Read only the relevant file(s), not the whole repo.
Use for: module behavior, configuration steps, menu paths, field names,
feature capabilities, Indian localization (`l10n_in`).

### Tier 2 — Web Search (fallback only)
Use only when local docs do not contain a clear answer.
Use for: GST/India regulatory updates, third-party integrations,
community workarounds, anything not in official docs.

### Tier 3 — Model Reasoning (last resort)
Use when neither local docs nor web search resolves the question.

### Step 1 — Use Known Folder Paths First

Before running a broad search, check the known folder structure.
All paths are relative to `odoo-official-docs\content\applications\`:

| Module | Folder Path |
|--------|------------|
| Inventory | `inventory/` |
| Inventory Valuation | `inventory/warehouses_storage/` |
| Purchase | `inventory/purchase/` |
| Manufacturing / MRP | `manufacturing/` |
| Accounting | `finance/accounting/` |
| India Localization | `finance/fiscal_localizations/india.rst` |
| Sales | `sales/` |
| CRM | `crm/` |
| HR & Payroll | `hr/` |
| Point of Sale | `sales/point_of_sale/` |
| E-commerce | `websites/ecommerce/` |
| Multi-company | `general/companies/` |

Using the folder path + keyword is faster and more accurate than a full repo search.


### Step 2 — Silent Failure Prevention Rule

**This is critical.** If a file path guess returns "not found":
- Do NOT answer from model memory
- Do NOT proceed silently as if the file was read
- Instead, run `search_files()` with the expanded keyword in the known folder
- Only answer after a file has been successfully opened and read
- If no file is found after search, switch to Tier 2 (web) and label the answer `[Web]`

A wrong answer delivered confidently is worse than saying the file wasn't found.

---

### Mandatory Source Label

Every Odoo answer must begin with one of these labels:

| Label | Meaning |
|-------|---------|
| `[Local Docs]` | Read from local 19.0 repo — version accurate |
| `[Web]` | From web search — may not reflect Odoo 19.0 accurately |
| `[Model Reasoning]` | Inferred from training data — not verified against docs |

When using `[Web]` or `[Model Reasoning]`, always add:
*"Verify this against your local 19.0 docs if it's critical."*

---

## Mission

This project exists to support Rohan Raj's work as a **Business Analyst Intern at Infintor
Solutions**, with the end goal of converting the internship into a full-time BA role.

All AI assistance in this project is oriented toward that mission: learning Odoo correctly,
producing professional BA artifacts, handling client interactions with integrity, and
passing Infintor's internal assessments.

---

## Skill Priority Order

When a task could involve multiple skills, apply them in this order:

```
1. odoo-ba.md          ← Primary. Always active. Governs all BA and Odoo work.
2. Supporting Skills   ← Secondary. Used only when they serve the BA mission.
```

If any supporting skill's output conflicts with the BA skill's guidance, the BA skill wins.

---

## Primary Skill

| Skill | File | When to Use |
|-------|------|-------------|
| Odoo Business Analyst | `.agents/skills/odoo-ba.md` | All Odoo, BA, client, documentation, and internship tasks |

This skill is **always active**. It contains Rohan's internship context, operating modes,
client communication rules, and career progression tracker. Read it before responding to
any task in this project.

---

## Supporting Skills

Use these only when they directly support a BA or Odoo deliverable.

### Design & Planning

| Skill | File | When to Use |
|-------|------|-------------|
| Brainstorming | `.agents/skills/brainstorming.md` | Exploring solution options before writing a BRD or FRS |
| Writing Plans | `.agents/skills/writing-plans.md` | Structuring long-form BA documents or blog posts |

### Implementation

| Skill | File | When to Use |
|-------|------|-------------|
| TDD | `.agents/skills/test-driven-development.md` | Writing UAT scripts or test cases for Odoo configurations |
| Subagents | `.agents/skills/subagent-driven-development.md` | Breaking large documentation tasks into parallel workstreams |
| Parallel Work | `.agents/skills/dispatching-parallel-agents.md` | Running multiple module analyses simultaneously |

### Quality & Verification

| Skill | File | When to Use |
|-------|------|-------------|
| Code Review | `.agents/skills/code-review.md` | Reviewing any Odoo Python customization or script |
| Debugging | `.agents/skills/systematic-debugging.md` | Diagnosing unexpected Odoo configuration behavior |
| Final Checks | `.agents/skills/verification-before-completion.md` | Before submitting any artifact to a mentor or client |

### Git & Workflow

| Skill | File | When to Use |
|-------|------|-------------|
| Git Worktrees | `.agents/skills/using-git-worktrees.md` | Managing parallel branches in the Odoo study repo |
| Branch Finishing | `.agents/skills/finishing-a-development-branch.md` | Completing and merging a documentation or config branch |

### Document & Data Output

| Skill | File | When to Use |
|-------|------|-------------|
| Word Documents | `.agents/skills/docx/SKILL.md` | Producing BRD, FRS, SOP, or any `.docx` artifact |
| PDF Documents | `.agents/skills/pdf/SKILL.md` | Creating or reading PDF-based client deliverables |
| Spreadsheets | `.agents/skills/xlsx/SKILL.md` | Gap Analysis tables, test case trackers, or data exports |
| CSV Summarizer | `.agents/skills/csv-data-summarizer-claude-skill/SKILL.md` | Analyzing Odoo data exports or report outputs |

---

## How to Read This File

This is the entry point for the project. It tells you:

1. **What this project is for** — BA internship at Infintor, goal is full-time hire
2. **Which skill governs all BA work** — `odoo-ba.md`, always read it first
3. **Which supporting skills exist** — and exactly when to pull them in

When in doubt, default to `odoo-ba.md`. The supporting skills are tools.
The BA mission is the purpose.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RohanRaj0718)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RohanRaj0718)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
