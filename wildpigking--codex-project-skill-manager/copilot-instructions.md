## codex-project-skill-manager

> - This repository plans how Codex should create future projects and manage user-level skills.

# Codex Project & Skill Manager

## Product boundary

- This repository plans how Codex should create future projects and manage user-level skills.
- Existing projects are out of scope. Never move, rename, or reorganize them.
- Existing skills may be inventoried read-only. Propose relocation or classification only after showing the exact plan, and apply it only after explicit user approval.
- Keep all shipped instructions, skills, scripts, configuration, and tests in English. Maintain README.zh-CN.md as a faithful Chinese translation for readers.

## Startup routing

- For a question about what this repository does, explain the product from README.md. Do not inspect the user's environment.
- For requests to inspect, set up, bootstrap, or plan a Codex workspace, use $bootstrap-codex-workspace. Start in read-only mode and return a plan.
- For requests about the location or internal structure of a new project, use $organize-codex-projects.
- For requests to inventory, classify, route, install, or update skills or the skills menu, use $manage-codex-skills.
- On the first substantive writable task in this repository, or after a meaningful milestone, use $maintain-project-ai-records.
- When the user asks whether project experience should become a reusable skill, or explicitly authorizes a skill update, use $reflect-codex-workflows.
- Read .agents/skills/SKILLS_MENU.md before selecting a shipped skill. For each distinct task or subrequest, shortlist at most three skills and read only their complete SKILL.md files.

## Consent contract

- Discovery and planning are read-only.
- A full workspace bootstrap plan must state the resolved global AGENTS.md path, user skills root, future-projects root, proposed writes, optional skill relocations, and a plan ID. A narrow new-project structure plan needs only the locations and writes relevant to that project.
- Do not treat cloning, opening, or reading this repository as approval.
- Before any write outside this repository, require explicit approval for the current plan ID.
- If the plan changes, invalidate the earlier approval.
- Do not overwrite a complete user AGENTS.md. Update only a clearly delimited managed block and preserve unrelated content.
- A read-only category recommendation is not a persisted classification. Do not record a classification, rebuild a user menu, or create, revise, move, or promote a user skill without approval.
- Repository-local development explicitly requested by the user follows normal task authorization and does not require a workspace plan ID.
- A read-only request suppresses project-record writes, even when the inspection produces a useful milestone.
- Workflow reflection may read the relevant user skills menu and shortlisted user SKILL.md files when cross-project comparison is requested. Do not scan the broader user home.

## Repository records

- Preserve existing user work and report material file changes.
- Maintain milestone-level notes in docs/ai/dev_log.md.
- Capture unpromoted reusable lessons in docs/ai/experience_candidates.md.
- Do not log credentials, private identifiers, every command, or hidden reasoning.

---
> Source: [WildPigKing/codex-project-skill-manager](https://github.com/WildPigKing/codex-project-skill-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
