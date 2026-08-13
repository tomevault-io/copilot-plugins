## bybit-tri-arbitrage

> **Language:** All specs, reports, code comments, and agent replies must be **English** unless the user explicitly requests another language. See `ai-office/global/english-only.mdc`.

# Agent Project Office

**Language:** All specs, reports, code comments, and agent replies must be **English** unless the user explicitly requests another language. See `ai-office/global/english-only.mdc`.

This repo uses **`ai-office/`** for Cursor rules (via symlink `.cursor/rules/ai-office`) and docs. Optional Cursor persona: **`.cursor/rules/infosec-persona.mdc`** — enable for **Isya** (Infosec) chats.

- **`.mdc`** — hard agent instructions (roles, prohibitions, mandatory actions).
- **`.md`** — process description, templates, reference (see `ai-office/process/`, `ai-office/templates/`).

## Rules (`ai-office/`)

| Path | Purpose |
|------|---------|
| **`ai-office/global/global-standards.mdc`** | Always on. Stack, **intake only through Team Lead**, config, logging, EPIC→Story→Task, Definition of Done. |
| **`ai-office/global/handoff-contracts.mdc`** | Always on. Valid handoffs; **Team Lead** primary user-facing entry; **Infosec (Isya)** dual entry (§6). |
| **`ai-office/global/security-baseline.mdc`** | Always on. Secrets, exposure, logging safety. |
| **`ai-office/commands/create-new-ver.mdc`** | Always on. Command `create_new_ver` for cloning strategy versions at repo root (domain command, not global standards). |
| **`ai-office/agents/team-lead/role.mdc`** | Team Lead (Tusya). Primary user-facing role; specs, routes Programmer/Tester/Quant/Infosec, merge/release, final delivery. |
| **`ai-office/agents/programmer/role.mdc`** | Programmer (Pusya). Implements tasks from Team Lead only; hands off to Tester. |
| **`ai-office/agents/tester/role.mdc`** | Tester (Trusya). QA gate; receives from Programmer; hands to Team Lead after checklist. |
| **`ai-office/agents/quant/role.mdc`** | Quant Researcher (Kasya). **Not user-facing** — FE, CatBoost/XGBoost, Optuna, variants; results to TL; code via TL → Programmer. |
| **`ai-office/agents/quant-critic/role.mdc`** | Quant Critic (Krisya). Structured research review (bias/leakage/stats); verdict APPROVED/REVISE/REJECT. |
| **`ai-office/agents/quant-judge/role.mdc`** | Quant Judge (Jasya). Closes Critic findings only; stripped packet (no agent chat). |
| **`ai-office/agents/infosec/role.mdc`** | Infosec (Isya). Security reviews; **TL assignment or direct user invoke** (`handoff-contracts.mdc` §6); no merge/release authority; fixes via TL → Programmer. |
| **`ai-office/process/quant-research-cycle.md`** | Researcher → validation → Critic → revise → Judge; human after 3–5 iterations. |

## Workflow

1. **New work** → Fill **`ai-office/task-intake.md`** (shortcut: `docs/task-intake.md` points here), invoke **Team Lead** (`ai-office/agents/team-lead/role.mdc`) for Architecture + EPIC → Stories → Tasks.
2. **Implementation** → Routed by Team Lead to **Programmer**; handoffs use **`ai-office/templates/programmer-handoff-template.md`**.
3. **QA** → **Tester** uses **`ai-office/checklists/qa-checklist.md`** and **`ai-office/templates/tester-report-template.md`**.
4. **Quant research** → TL → **Kasya** (FE / CatBoost / XGBoost / Optuna / variants) → validation → **Krisya** → revise → **Jasya** — see **`ai-office/process/quant-research-cycle.md`**.
5. **Before deploy** → **`ai-office/checklists/release-checklist.md`**, security items in **`ai-office/checklists/review-checklist.md`**, **`docs/security-checklist.md`** where applicable; involve **Infosec (Isya)** via Team Lead or direct invoke per **`handoff-contracts.mdc`** when a security review is needed.
6. **Verdicts:** hand off Pusya → Trusya (in chat/task); Trusya verdict → report to Pusya or QA sign-off + handoff to Tusya (see **`docs/process-handoff-programmer-tester-team-lead.md`**).
7. **Git:** branch per task, PR/MR, merge after PASS + approve - see **`ai-office/policies/branch-and-pr-policy.md`**, **`ai-office/policies/approval-policy.md`**, deployment - **`ai-office/policies/deployment-policy.md`**.

## Docs

| Path | Use |
|------|-----|
| `ai-office/task-intake.md` | Canonical task intake (EPIC → Story → Task). |
| `ai-office/README.md` | Index of the office layout. |
| `ai-office/distribution/CURSOR_AGENTS_GUIDE.md` | **Porting agents to Cursor** — roles, chains, setup. |
| **`ai-office/global/english-only.mdc`** | Always on. Specs/reports/code comments/replies in **English** unless user requests otherwise. |
| `ai-office/policies/` | Branch/PR, approval, deployment policy (`branch-and-pr-policy.md`, `approval-policy.md`, `deployment-policy.md`). |
| `docs/templates/` | Detailed templates (epic, story, task, architecture, release); prefer `ai-office` when they differ. |

## Quick reference

- **Official work:** **Team Lead** owns implementation routing, merge/release, and packaging of Quant analytics (`ai-office/global/global-standards.mdc`, `ai-office/global/handoff-contracts.mdc`). **Programmer**, **Tester**, **Quant Researcher / Critic / Judge** are internal execution roles. **Infosec (Isya)** — security reviews with **dual entry** (TL or direct invoke); merge/release stays with TL — see **`handoff-contracts.mdc`** §6.
- Quant cycle: Researcher (Kasya) → Critic (Krisya) → Judge (Jasya) — **`ai-office/process/quant-research-cycle.md`**.
- All work: **EPIC → Story → Task** before coding.
- Use **docker compose** only when the task/stack includes Docker (`ai-office/global/global-standards.mdc`); Python via **conda**; config via shell/conda env vars (**no `.env` files**); no secrets in Git.
- Trading: log signals/orders, idempotent execution, strategy separate from execution.

---
> Source: [ryu878/Bybit_tri_Arbitrage](https://github.com/ryu878/Bybit_tri_Arbitrage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
