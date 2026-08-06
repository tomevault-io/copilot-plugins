## ops-engineering-skills

> This file gives any AI coding agent (Claude Code, GitHub Copilot, OpenAI

# AGENTS.md

This file gives any AI coding agent (Claude Code, GitHub Copilot, OpenAI
Codex, Cursor, Gemini CLI, or otherwise) working in this repository the
context it needs, independent of any vendor-specific plugin system.

## What this repository is

`ops-engineering-skills` is an open-source collection of **Agent Skills**
(the [agentskills.io](https://agentskills.io) SKILL.md standard) covering
296 skills across 22 domains: DevOps, DevSecOps, Cloud (AWS/Azure/GCP/OCI/
on-prem), Kubernetes Platform, CI/CD Tooling, GitOps/Argo Ecosystem,
Policy & Governance Tooling, IaC & Automation Tooling, Security Scanning
Tooling, Enterprise Collaboration (Jira/Confluence), Observability &
Platform Extras, Standards & Compliance Frameworks, Site Reliability
Engineering, AgentOps/AI-Agent engineering, MLOps, Database Operations,
Messaging & Data Orchestration, Service Mesh & API Gateway, Serverless &
Alternative Compute, Internal Developer Platform, Incident Tooling &
ITSM, and Role-Based Engineering Practices. Each skill is a self-contained,
production-oriented playbook an agent can load to give better, more
concrete guidance in that domain. See
[`docs/SKILLS_INDEX.md`](docs/SKILLS_INDEX.md) for the full list.

## Repository layout

```
plugins/<domain>/skills/<skill-name>/SKILL.md   # the actual skill content
plugins/<domain>/.claude-plugin/plugin.json     # per-domain plugin manifest
.claude-plugin/marketplace.json                 # Claude Code marketplace catalog
template/skill-template/SKILL.md                # starting point for new skills
spec/authoring-guide.md                         # required frontmatter + sections
scripts/validate_skills.py                      # schema/lint validator
installers/install.sh                           # cross-agent installer
```

## Rules for agents editing this repo

- Every skill must conform to `spec/authoring-guide.md` — required
  frontmatter fields and the 8 required body sections (Purpose, When to
  use, Prerequisites & environment, Step-by-step guidance, Best practices,
  Common pitfalls, Worked example, Cross-references).
- Skill folder name must equal the `name` field exactly and be kebab-case.
- Never commit real secrets, tokens, or credentials — use placeholders.
- Never present a destructive command (`terraform destroy`, `kubectl
  delete namespace`, `git push --force`, etc.) as a safe default without an
  explicit warning.
- After adding or editing a skill, run:
  ```bash
  python3 scripts/validate_skills.py
  claude plugin validate .
  ```
- Keep guidance vendor-neutral by default; call out tool-specific steps
  explicitly rather than assuming one vendor.

## Where to add a new skill

Pick the closest existing domain under `plugins/`, or propose a new domain
via an issue first (see `CONTRIBUTING.md`). Copy
`template/skill-template/SKILL.md` into
`plugins/<domain>/skills/<new-skill-name>/SKILL.md` and fill it in.

---
> Source: [selvarajmurugesan90/ops-engineering-skills](https://github.com/selvarajmurugesan90/ops-engineering-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
