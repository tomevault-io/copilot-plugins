## company-os

> This is the canonical guide for all AI tools working in this Company OS. Tool-specific files, including `CLAUDE.md`, should point here rather than duplicating the operating rules.


# Agent guide

This is the canonical guide for all AI tools working in this Company OS. Tool-specific files, including `CLAUDE.md`, should point here rather than duplicating the operating rules.

## Read first

| Task | read before acting |
| --- | --- |
| Any task | `INDEX.md`, this file |
| Repository purpose, folder model, or onboarding | `README.md` |
| Company purpose, structure, voice, positioning | `company/README.md` and relevant `company/` files |
| Strategy, prioritization, goals, tradeoffs | `strategy/README.md`, `company/README.md`, `decisions/README.md`, `decisions/AGENTS.md` |
| Strategy analysis | `strategy/analysis/README.md`, `strategy/analysis/AGENTS.md`, `strategy/today.md` |
| Current operations | `run/README.md`, relevant `workflows/` and `wiki/` files |
| Transformation work | `change/README.md`, `strategy/README.md`, relevant decisions |
| Client-specific work | `clients/README.md`, `clients/AGENTS.md`, and exactly one client folder |
| SOP or playbook work | `wiki/README.md`, `.templates/playbook.md`, relevant skills |
| Generated drafts or reports | `output/README.md`; write output there first |
| Agent tool behavior | `plugin/README.md`, `SKILLS.md`, `.ai/skills/`, `GOVERNANCE.md` |
| Creating or changing decisions | `decisions/README.md`, `decisions/AGENTS.md`, `.templates/decision.md` |

## Rules

1. This repo is markdown and config only.
2. `AGENTS.md` is canonical. Tool-specific files may add conventions, not new governance.
3. Context and output stay separate. Drafts, reports, and generated bundles go to `output/`.
4. Live state stays in systems of record, not in markdown copies.
5. Do not mix client contexts. Load one client folder at a time unless the user explicitly asks for comparison.
6. Propose durable learnings as repo updates. Do not leave important knowledge only in chat history.
7. External sends, deletes, deploys, money movement, credential changes, or client-confidential sharing require human approval.
8. Respect `load_policy`. Do not load `archive` or `never` files without explicit need and approval.
9. Every canonical markdown file, including wiki and brand files, must carry `owner`, `status`, `last_updated`, `source_of_truth`, `load_policy`, and `related` frontmatter.
10. Keep provenance in `source_of_truth` and git history, not as inline source attributions in canonical document bodies.

## Source of truth boundaries

| Domain | system of record |
| --- | --- |
| Company context, intent, playbooks, strategy, operating rules | This Company OS |
| Contacts, accounts, pipeline | CRM |
| Tasks and delivery execution | Linear, GitHub Issues, or chosen project system |
| Financial state | Accounting or finance system |
| Product analytics and funnel metrics | Analytics system |
| Conversations and notifications | Slack, email, calendar |
| Generated drafts | `output/` until harvested |

When these disagree, prefer the system of record for live state and this repo for authored intent.

## Sensitive data rules

- Do not place secrets, API keys, private credentials, personal health data, payroll details, or raw confidential exports in canonical context.
- If sensitive raw material is needed, keep it in `raw/` with restrictive metadata and summarize only durable, approved insights into canonical files.
- Client-confidential information belongs in `clients/<client>/` with clear sensitivity markers and should not be generalized into public playbooks unless anonymized.

## Harvest loop

Harvesting turns work into durable capability:

1. Human correction, repeated successful output, or project learning appears.
2. Agent proposes a small update to a skill, playbook, process, workflow, or decision.
3. Owner reviews through PR, or asks an agent to harvest. A harvest request is acceptance: write settled canonical statements without `proposed` or `pending confirmation` hedges. Evidence-gathering notes may remain where future data is genuinely needed.
4. Approved learning becomes canonical context. Convert legacy metadata to the standard frontmatter during harvesting.
5. Mark the source draft `status: read` and leave it at `output/YYYYMMDD-topic.md`; do not create nested draft folders for new output files.
6. Add a dated, newest-first entry to `company/changelog.md` saying what was harvested and where it landed.
7. Future agents load the improved context.

Be honest about what generalizes. Do not convert one-off circumstances into company rules.

## Task output

When creating new durable files, use the templates in `.templates/`. When unsure whether something is durable, put it in `output/` first and ask for review.

---
> Source: [EmpoweredHouse/company-os](https://github.com/EmpoweredHouse/company-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
