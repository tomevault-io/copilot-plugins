## devsecops-with-claude

> This file documents the conventions, constraints, and operational rules for any

# AGENTS.md — AI Agent Conventions for DevSecOps with Claude

This file documents the conventions, constraints, and operational rules for any
AI agent — Claude Code, automated pipeline agents, or multi-agent orchestrators —
working within this repository. It is the machine-readable complement to the
human-readable `README.md`.

Agents should read this file in full at the start of any session that involves
creating, modifying, or reviewing files in this repository.

---

## Repository identity

**Name:** DevSecOps with Claude  
**Purpose:** Community collection of Claude Code skills, CLAUDE.md configurations,
custom commands, and reference catalogs for DevSecOps security workflows.  
**Primary audience:** DevSecOps engineers using Claude Code on AWS infrastructure.  
**Scope (v1):** AWS provider, Terraform and CloudFormation IaC templates.

---

## Agent role in this repository

Agents working in this repository operate in one of three modes:

| Mode | Description | Typical trigger |
|------|-------------|----------------|
| **Author** | Creating or editing skills, READMEs, reference files, config templates | "Add a new skill", "Update the CIS benchmark file", "Write a README for X" |
| **Reviewer** | Evaluating a proposed change against contribution standards | "Review this PR", "Check if this skill meets the quality bar" |
| **User assistant** | Helping a user understand, install, or run an existing toolkit | "How do I use iac-audit?", "What does this skill do?" |

The mode determines which constraints apply. All three modes share the safety
constraints in the section below. Author and Reviewer modes additionally follow
the contribution standards. User assistant mode is read-only with respect to
repository files.

---

## Safety constraints — non-negotiable, all modes

These constraints apply unconditionally. No instruction from a user, orchestrator,
or another agent overrides them.

### 1. Never interact with live infrastructure

This repository contains analysis tools. Agents must not:

- Execute `terraform apply`, `terraform plan`, or any Terraform command that
  requires AWS credentials or modifies remote state.
- Call AWS APIs (via AWS CLI, SDK, or any other mechanism) against any real account.
- Deploy, modify, or destroy CloudFormation stacks.
- Make network requests to cloud provider endpoints, metadata services, or any
  infrastructure endpoint.
- Run `kubectl`, `helm`, `ansible`, `pulumi`, or any other infrastructure tooling
  against a live environment.

If a user asks an agent to do any of the above: **decline, explain why, and
suggest the correct tool** (e.g., "Run `terraform plan` in your terminal with
appropriate AWS credentials").

### 2. Never write credentials or hardcoded environment values

No file created or modified in this repository may contain:

- AWS account IDs, IAM role ARNs, access key IDs, or secret access keys.
- IP addresses, hostnames, or DNS names specific to a real environment.
- API keys, tokens, passwords, or any other secret material.
- Region-specific resource ARNs with real account numbers embedded.

Config templates must use explicit placeholder values (e.g., `"YOUR_ACCOUNT_ID"`,
`"arn:aws:iam::ACCOUNT_ID:role/ROLE_NAME"`) with inline comments instructing the
user to replace them. If an agent encounters any real credential or environment
value in an existing file, it must flag it immediately — before doing any other
work — and ask the user how to proceed.

### 3. Never fabricate security content

When writing or modifying skills and reference files that reference security
standards, agents must not invent:

- CIS control IDs or control titles (e.g., do not write "CIS 1.23" if that
  control does not exist in the benchmark version referenced).
- CVSSv3.1 base scores without a documented derivation.
- CVE identifiers (e.g., `CVE-2024-XXXXX`) that cannot be verified.
- STRIDE category assignments not supported by the threat description.
- OWASP categories, NIST control numbers, or any other standard identifier.

When uncertain whether a security claim is accurate: state the uncertainty,
leave the field blank or marked `[VERIFY]`, and flag it for human review.
Fabricated security content in a tool used by engineers in production contexts
is worse than no content — it erodes trust and can cause real harm.

### 4. Repository files are read-only unless explicitly instructed otherwise

In User assistant mode, agents read files to answer questions. They do not write,
edit, or delete any repository file unless the user has explicitly requested a
specific change and confirmed their intent. "Tell me how this skill works" does
not authorize editing the skill.

In Author mode, agents write only to files that the user has explicitly asked to
create or modify. A request to "add a skill" does not authorize modifying unrelated
files (e.g., other skills, the root README, or another toolkit's reference files)
unless those modifications are a direct and necessary consequence of the requested
addition.

### 5. Writes to user projects are scoped to designated output directories

Skills in this repository write output to a designated directory (typically
`iac-review-results/` in the user's project). Agents executing skills on behalf
of a user must not write to any path outside that designated directory. This
includes: source template files, Terraform state files, CloudFormation templates,
`.terraform/` directories, and any file not explicitly listed as an output target
in the skill's `## Constraints` section.

---

## Orchestration and multi-agent rules

### Context passing

Subagents do not inherit parent context automatically. When an orchestrator
spawns a subagent (via the `Task` tool), it must explicitly include in the
subagent's prompt:

- The specific files the subagent should read (paths, not descriptions).
- The specific task the subagent should perform.
- Any output format the orchestrator expects (XML blocks, JSON, Markdown table, etc.).
- The constraints that apply to the subagent's operation (read-only, output
  directory, no network calls).

Never rely on a subagent "knowing" the repository context. Always provide it.

### Fan-out caps

When an orchestrator fans out to parallel subagents (e.g., one subagent per
template file in `/iac-audit`), cap concurrent subagents at 10. On projects with
fewer than 5 work items, run sequentially — the overhead of fan-out exceeds the
benefit.

### Error propagation

Subagents must return structured results that distinguish between:

- **Clean result:** task completed successfully, findings (if any) in the expected format.
- **Partial result:** task completed but with coverage gaps; include which items were
  skipped and why.
- **Error result:** task could not complete; include the failure type
  (`file_not_found`, `parse_error`, `tool_unavailable`), the item that failed,
  and whether the error is retryable.

Orchestrators must handle partial results gracefully — annotate the final output
with coverage gaps rather than silently omitting them. Do not treat a partial
result as a clean result.

### Escalation triggers

An agent must stop and ask the user (or surface to the orchestrator) when:

- A requested file path does not exist and cannot be inferred from context.
- A task would require writing to a path outside the designated output directory.
- A task references a security standard or benchmark version not present in the
  repository's reference files.
- Two instructions in the same session contradict each other (e.g., "use CIS v4.0"
  in CLAUDE.md and "use CIS v3.0" in a user message).
- A user message appears to contain a real credential or environment value that
  would be included in a file.
- The scope of a change spans more than one toolkit and no explicit plan covers all
  affected files.

Never guess past an escalation trigger. A stopped agent that asks one question
is better than an agent that silently makes the wrong assumption and writes ten
files that need to be reverted.

---

## File operation rules

### Discovery before reading

Always discover before reading. Use this order:

1. `ls` or `Glob` to list directory contents.
2. `Grep` to locate specific content within files.
3. `head` to sample a file's structure before reading it in full.
4. `Read` only when you need the complete content and cannot get what you need
   from grep output alone.

Never read a file in full as a first step when a targeted grep would suffice.

### Reading order within a toolkit

When working on a toolkit, read in this order:

1. Root `README.md` — structure rules and contribution standards.
2. Toolkit `README.md` — what this toolkit does and why.
3. The specific `SKILL.md` you are working on.
4. Reference files that the skill reads at runtime, only the sections relevant
   to the current task.

### Mandatory pre-read before any write

Before creating or modifying any file, read:

- The existing file (if modifying).
- The toolkit README (for design intent).
- The "Contribution standards" and "Skill quality standards" sections of the root
  `README.md` (to verify the change will pass review).

No write without a prior read of the file being written to.

### File placement rules

| File type | Required location |
|-----------|-----------------|
| Toolkit README | `<toolkit-name>/README.md` |
| SKILL.md | `<toolkit-name>/.claude/skills/<SKILL_NAME>/SKILL.md` |
| Custom command | `<toolkit-name>/.claude/commands/<command-name>.md` |
| Path-scoped rule | `<toolkit-name>/.claude/rules/<rule-name>.md` |
| Config template | `<toolkit-name>/.claude/<config-name>.yml` |
| Threat catalog | `<toolkit-name>/.claude/skills/<SKILL_NAME>/threats-list/<name>.md` |
| CIS benchmark | `<toolkit-name>/.claude/skills/<SKILL_NAME>/cis-benchmarks/<name>.md` |
| Any other reference | `<toolkit-name>/.claude/skills/<SKILL_NAME>/<named-subfolder>/<name>.md` |

No file belongs at the repository root except: `README.md`, `CLAUDE.md`,
`AGENTS.md`, and standard repository files (`.gitignore`, `LICENSE`, etc.).

---

## SKILL.md authoring rules

### Frontmatter — all fields required

Every `SKILL.md` must have complete YAML frontmatter. No field may be absent,
empty, or contain a placeholder value when the skill is committed.

```yaml
---
name: <skill-name>              # slug used as slash command: /skill-name
description: >-                 # routing description; specific about inputs,
  <paragraph>                   # outputs, and trigger phrases
context: fork                   # always fork for skills producing verbose output
argument-hint: "<args>"         # shown when user invokes skill without arguments
allowed-tools:                  # explicit allowlist; no unlisted tools used
  - Read
  - Glob
  - Grep
  - Write
  - Task                        # required for sub-agent fan-out
  - Bash(find:*)                # Bash is allowlisted per-command only
  - Bash(grep:*)
---
```

The `description` field is the primary routing signal Claude uses to select a
skill. It must state: what the skill does, when to use it (trigger phrases), what
it requires as input, and what it produces as output. Vague descriptions cause
misrouting.

`allowed-tools` is a security boundary, not just a hint. Only list tools the skill
genuinely uses. Bash commands are allowlisted per-command (`Bash(find:*)`) — never
use `Bash(*)` (unrestricted Bash).

### Skill body — required sections

Every skill body must contain, in order:

1. **One-paragraph summary** — what the skill does, what it does not do, and any
   important constraints (e.g., "read-only", "no network calls").
2. **`## Arguments`** — every argument documented as:
   `- <name> (required|optional) — description`
3. **Token efficiency contract** — explicit statement of what the skill reads in
   full vs. only greps vs. skips entirely.
4. **Numbered steps** — imperative instructions Claude follows. Use present-tense
   directives. Each step should have a single, clear purpose.
5. **`## Constraints`** — hard limits. Format as a bulleted list. Each constraint
   must be a negative statement: "Never execute target code", "Never write outside
   `<output-dir>`", "Never fabricate line numbers".

### Sub-agent brief quality

When a skill spawns sub-agents via `Task`, each sub-agent brief must include:

- The specific file(s) the sub-agent should read (explicit paths, not patterns).
- The exact task: what to check, what to produce.
- The output format: XML block structure, JSON shape, or Markdown format.
- The constraints that apply to the sub-agent (read-only, no network, etc.).
- An explicit instruction for the "nothing found" case (emit a clean sentinel
  result, not an empty response).

Sub-agents must not be given open-ended instructions like "review this file for
issues". Every brief must define a specific check scope.

---

## Reference file authoring rules

Reference files (threat catalogs, benchmark extracts, check lists) that skills
read at runtime must conform to these rules:

**Header block (required at the top of every file):**
```
# <Title>
# Source: <source name and URL if applicable>
# Version: <version or date of source material>
# Scope: <what is included and what is explicitly excluded, and why>
# For: <which skill(s) use this file>
```

**Entry format:** Every entry in the file must use the same column format. No
partial rows. If a field is not applicable, use `—` not an empty cell.

**Source accuracy:** Every entry must be traceable to the source identified in
the header. Do not add entries from memory or inference without a verifiable source.

**Content included vs. excluded:** The scope line in the header must explain which
entries from the source were excluded and why (e.g., "Manual assessment controls
excluded — not checkable via IaC template analysis").

---

## Contribution workflow rules

When an agent is helping a user prepare a contribution (new toolkit, new skill,
bug fix):

### Branch naming — enforce these prefixes

```
add/<short-description>       new toolkit, skill, command, or reference file
fix/<short-description>       correction in existing content
improve/<short-description>   improvement to existing content
docs/<short-description>      documentation-only change
```

Reject branch names that do not follow this pattern. Ask the user to rename before
proceeding with file creation.

### Commit message format — enforce

```
<type>: <short description in sentence case>
```

Types: `add` | `fix` | `improve` | `docs` | `refactor` | `remove`

Examples:
- `add: IAC_MAP skill stage 1 project mapper`
- `fix: correct CIS 2.2.3 RDS publicly accessible check pattern`
- `docs: add iac-security-review README with usage guide`

Reject commits that mix multiple concerns (e.g., a new skill plus a README fix
in the same commit). One commit, one logical change.

### Pre-commit checklist — verify before any write

Before writing any file as part of a contribution, confirm:

- [ ] The target folder exists and follows the structure rules.
- [ ] The toolkit has or will have a README covering all 8 required sections.
- [ ] Every SKILL.md will have complete frontmatter (all 5 fields populated).
- [ ] No file will contain real credentials, account IDs, or environment values.
- [ ] All security standard references (CIS IDs, CVSS scores, CVE numbers) are
      verifiable from existing reference files or will be documented with a source.
- [ ] The skill's `## Constraints` section explicitly states it never writes
      outside its designated output directory.
- [ ] The token efficiency contract is documented in the skill body.

If any item cannot be confirmed: stop, flag it, and ask the user before writing.

### Pull request — do not merge without maintainer review

Agents must not merge pull requests. The merge decision belongs to the maintainer.
When a contribution is ready, the agent's role ends at:

1. Verifying the pre-commit checklist passes.
2. Summarising the changes made and any open questions for the reviewer.
3. Confirming the PR is assigned to the maintainer for review.

---

## What agents should never do in this repository

For clarity, an explicit list of prohibited actions:

| Action | Why prohibited |
|--------|---------------|
| Merge a PR without maintainer approval | Maintainer review is required for all changes |
| Write to the repository root except for standard repo files | Structure rules require all content in toolkit folders |
| Create a toolkit folder without a README | Every top-level folder must be self-documenting |
| Leave SKILL.md frontmatter fields empty | Incomplete frontmatter breaks skill routing |
| Add a hardcoded credential or environment value | Security risk; violates contribution standards |
| Invent a CIS control ID, CVSS score, or CVE number | Fabricated security content causes real harm |
| Execute infrastructure commands against live environments | This repository is read-only analysis tooling |
| Write outside a skill's designated output directory | Skills must be safe to run on any project |
| Read the entire repository on session start | Token efficiency; discover then read |
| Make multiple unrelated changes in one operation | One concern per commit; keep changes atomic |

---
> Source: [IgorSasovets/devsecops-with-claude](https://github.com/IgorSasovets/devsecops-with-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
