## spike-devops-ai-agentic-coding-assistant

> This repository manages AWS cloud infrastructure using Terraform. All cloud resources are defined as code, version-controlled, and deployed through automated pipelines. Claude Code assists with authoring, reviewing, refactoring, and validating Terraform configurations for AWS.

# Terraform AWS Infrastructure

## Description

This repository manages AWS cloud infrastructure using Terraform. All cloud resources are defined as code, version-controlled, and deployed through automated pipelines. Claude Code assists with authoring, reviewing, refactoring, and validating Terraform configurations for AWS.

---

## Project Overview

- **Cloud Provider:** AWS
- **IaC Tool:** Terraform (OpenTofu compatible)
- **State Backend:** S3 + DynamoDB locking
- **Secrets:** Never stored in code — use AWS Secrets Manager or SSM Parameter Store references
- **Environments:** Separated by Terraform workspace; variable files live in `vars/<workspace-name>/` — workspace names vary by project

---

## Architecture

```
.
├── CLAUDE.md
├── .mcp.json
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── vars/
│   └── <workspace-name>/
│       ├── terraform.tfvars
│       └── <additional>.tfvars
├── modules/
│   └── <module-name>/
│       ├── main.tf
│       ├── variables.tf
│       ├── outputs.tf
│       └── README.md
├── .claude/
│   ├── agents/           # Sub-agent definitions
│   ├── commands/         # Slash command prompts
│   ├── hooks/            # Pre/PostToolUse shell scripts
│   ├── skills/           # Reusable skill prompts
│   └── settings.json     # Hooks and permissions config
└── docs/
```

- **`vars/`** — One subdirectory per Terraform workspace. Each contains a `terraform.tfvars` plus any additional `.tfvars` files needed for that workspace. To target a workspace: `terraform workspace select <name>` then `terraform plan -var-file=vars/<name>/terraform.tfvars`.
- **`providers.tf`** — Defines the `terraform` block (`required_version`, `required_providers`) and all provider configurations (e.g., `aws`). This is the single source of truth for provider versions and AWS region/assume-role settings.
- **`modules/`** — Reusable, composable Terraform modules scoped by AWS service or logical domain. Each module is self-contained with its own variables, outputs, and README.
- **`docs/`** — Architecture decision records (ADRs), diagrams, and runbooks. Reference with `@docs/<file>.md` when needed.

> Treat every workspace with caution. See Safety Layers.

---

## MCP Servers

MCP servers are configured in `.mcp.json` at the project root. The following server is active for this repository:

- **`hashicorp/terraform-mcp-server`** — provides real-time access to Terraform Registry provider documentation, modules, and policies. Ensures generated Terraform uses accurate, up-to-date resource arguments rather than potentially stale training data.
- **`awslabs/aws-documentation-mcp-server`** — provides real-time access to AWS service documentation. Useful when authoring resources, verifying service behaviour, or checking API-level details not covered by Terraform provider docs. No AWS credentials required.

Both servers run locally over stdio. See `.mcp.json` for configuration.

---

## Custom Agents

Agent definitions live in `.claude/agents/`. Each agent file contains a YAML frontmatter block (name, description, model, tools, skills) followed by behavioural instructions.

| Agent | Model | Purpose |
|---|---|---|
| `devops-engineer` | opus | Authors, scaffolds, and refactors Terraform code; validates configurations against project conventions |
| `devsecops-engineer` | sonnet | Audits Terraform for security misconfigurations, IAM over-permissioning, public exposure, and missing encryption |
| `finops-engineer` | haiku | Analyses Terraform configurations for cost impact; identifies expensive defaults and recommends optimisations |

**Agent routing rules:**
- Always delegate AWS and Terraform questions, tasks, and commands to the `devops-engineer` agent — do not answer them directly.

---

## Memory & Team Knowledge Sharing

Whenever you save a memory (feedback, project context, or workflow rules), also add the same information to the relevant section of this `CLAUDE.md` file so all team members benefit from it in their sessions.

---

## Skills

Skills live in `.claude/skills/`, each as a subdirectory containing a `SKILL.md` file with YAML frontmatter (name + description) and instructions. Claude loads them automatically when relevant.

| Skill directory | Purpose |
|---|---|
| `skills/terraform-style/` | HCL formatting, naming, and structure conventions |
| `skills/aws-tagging/` | Required and recommended AWS resource tags |
| `skills/project-context/` | Project-specific context: account structure, approved services, and architectural decisions |

Example frontmatter for a skill:
```yaml
---
name: aws-tagging
description: Applies required AWS tagging standards to Terraform resources. Use when writing or reviewing any resource that supports tags.
---
```

*Keep each `SKILL.md` focused and under 500 lines. Move bulky reference material into a `references/` subdirectory within the skill.*

---

## Commands

Slash commands live in `.claude/commands/`. Run them with `/command-name` inside a session.

| Command | Description |
|---|---|
| `/workspaces` | Lists all Terraform workspaces, indicating the active one |
| `/workspace-select` | Presents an interactive dropdown to select a Terraform workspace |
| `/plan` | Runs `terraform plan` against the currently selected workspace (must not be `default`) |
| `/apply` | Applies the saved plan for the currently selected workspace — requires explicit human confirmation |
| `/validate` | Runs `terraform validate` and `tflint` across all modules |
| `/fmt-check` | Runs `terraform fmt -check -recursive` and lists offending files |
| `/module-new <name>` | Scaffolds a new module under `modules/<name>/` with standard files |
| `/drift` | Compares live infrastructure against Terraform state and flags drift |
| `/docs-update` | Regenerates docs for the root module (injected into `README.md`) and fully replaces each module's `README.md` |

---

## Safety Layers

These rules are non-negotiable. Follow them in every session.

**Never do without explicit human confirmation:**
- `terraform apply` on any workspace — always present the full plan and wait for explicit approval before applying, regardless of environment. After every `/plan` run, show a full resource-by-resource breakdown grouped by module (resource address + action), then ask the user to confirm before proceeding to apply.
- `terraform destroy` or `terraform plan -destroy` — these are **never acceptable**. To deprovision resources, set `create = false` (globally or per-resource entry) in the workspace tfvars and apply normally.
- Modifications to IAM policies, roles, or trust relationships
- Changes to S3 bucket ACLs, public access settings, or bucket policies
- Deletion or modification of state backend resources (S3 bucket, DynamoDB table)

**Always do:**
- Run `terraform validate` before proposing any plan
- Check for hardcoded AWS account IDs, access keys, or secrets — flag immediately if found
- Preserve existing resource tags unless explicitly told to change them
- Use `prevent_destroy = true` lifecycle on stateful resources (RDS, S3, DynamoDB) unless overriding intentionally

**Sensitive resource classes** (treat with extra care):
`aws_iam_*`, `aws_secretsmanager_*`, `aws_kms_*`, `aws_s3_bucket_public_access_block`, `aws_security_group`, `aws_vpc`

---

## Hooks

Hooks are configured in `.claude/settings.json` and scripts live in `.claude/hooks/`. They enforce rules mechanically rather than relying on Claude following instructions.

| Hook | Event | Script | Purpose |
|---|---|---|---|
| Terraform destroy blocker | `PreToolUse` | `hooks/block-destructive.sh` | Blocks `terraform destroy` unconditionally — `terraform apply` requires explicit human confirmation via the `/apply` command |
| Terraform fmt | `PostToolUse` | `hooks/terraform-fmt.sh` | Runs `terraform fmt` on any `.tf` file after it is written or edited |

Hook scripts must be executable: `chmod +x .claude/hooks/*.sh`

---

## Conventions

Naming, formatting, file structure, variable rules, versioning, state, and comment standards are defined authoritatively in the `terraform-style` skill at `.claude/skills/terraform-style/SKILL.md`. Tagging standards are defined in the `aws-tagging` skill at `.claude/skills/aws-tagging/SKILL.md`.

Both skills are loaded automatically when writing or reviewing Terraform code. They are the single source of truth for all conventions — do not restate or duplicate their rules in CLAUDE.md, agent files, or commands. Update the skill when a rule changes.

**Documentation:**
- Every module must have a `README.md` generated by `terraform-docs`
- Non-obvious decisions belong in `docs/decisions/` as short ADR files

Community module preferences (VPC, ALB, EC2) are defined in the `terraform-style` skill.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jacobscunn07) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
