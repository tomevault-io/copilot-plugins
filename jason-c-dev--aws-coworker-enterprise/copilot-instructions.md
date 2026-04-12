## aws-coworker-enterprise

> This skill defines:

# CLAUDE.md - AWS Coworker Usage Instructions

**This file ensures all AWS interactions go through AWS Coworker's safety model.**

---

## About AWS Coworker

AWS Coworker is built on **[Claude Code](https://claude.ai/code)** and **Anthropic's Claude Agent SDK**, augmenting AWS best practices with Claude's agentic capabilities. The core works out of the box. Extend it for your organization's needs.

| Claude Code Capability | AWS Coworker Implementation |
|------------------------|----------------------------|
| Task tool (sub-agents) | Specialized AWS agents (Planner, Executor, Guardrail) |
| Skill system | AWS CLI patterns, Well-Architected guidance, governance policies |
| Slash commands | Safety-first workflows with approval gates |
| Model selection | Tiered strategy: orchestrator (your model) + workers (Haiku/Sonnet) |

AWS Coworker gives Claude the right context, guardrails, and patterns for enterprise cloud work.

### Three-Layer Architecture

AWS Coworker has three layers. The CLI is the core product — everything else exists to support it.

| Layer | What | Purpose |
|-------|------|---------|
| **AWS Coworker (CLI)** | Commands, skills, agents, config | The core. What we're building. |
| **ACW Server** (`server/`) | REST + SSE API | Wraps the CLI for remote deployment |
| **Web UI** (`web-ui/`) | Reference implementation | Demonstrates the server API. Optional. |

**Dependencies flow downward only.** The CLI never knows the server exists. The server never knows the web UI exists. See Tenet 10 in `docs/DESIGN.md`.

---

## Critical Instruction

**NEVER execute AWS CLI commands directly.** All AWS operations must go through AWS Coworker commands to ensure:

- Profile and region are announced before any operation
- Governance guardrails are consulted
- Mutations require explicit approval
- Production changes go through CI/CD, not direct CLI
- Rollback procedures are considered

### DO NOT (Prohibited Without Approval)

- **DO NOT** execute any AWS CLI command that changes state without user approval
- **DO NOT** delegate mutations to sub-agents without first presenting a plan
- **DO NOT** treat "small" changes (tagging, config updates) as exempt from approval
- **DO NOT** assume user intent implies approval—wait for explicit confirmation

---

## How to Handle AWS Requests

When a user asks anything related to AWS, route to the appropriate command:

| User Intent | Route To |
|-------------|----------|
| Discover/query AWS resources | `/aws-coworker-plan-interaction` (discovery mode) |
| Plan a change or deployment | `/aws-coworker-plan-interaction` |
| Execute in non-production | `/aws-coworker-execute-nonprod` |
| Deploy to production | `/aws-coworker-prepare-prod-change` |
| Rollback a change | `/aws-coworker-rollback-change` |
| Set up a new AWS account | `/aws-coworker-bootstrap-account` |
| Cost or monitoring questions | `/aws-coworker-plan-interaction` with observability focus |

### Trigger Patterns

**ANY request involving these keywords or concepts MUST route through AWS Coworker:**

- AWS services: S3, EC2, Lambda, RDS, VPC, IAM, CloudWatch, Cost Explorer, etc.
- AWS operations: list, describe, create, delete, start, stop, deploy, etc.
- Cost/billing: cost summary, spending, budget, billing, charges, usage
- Monitoring: metrics, alarms, logs, CloudWatch, dashboards
- Infrastructure: instances, buckets, databases, functions, networks

### Examples

When a user makes a free-form request, invoke the appropriate command and pass the user's request as context:

| User Says | Action |
|-----------|--------|
| "What S3 buckets exist?" | Invoke `/aws-coworker-plan-interaction` with goal: "Discover S3 buckets" |
| "List my EC2 instances" | Invoke `/aws-coworker-plan-interaction` with goal: "List EC2 instances" |
| "Create a new VPC" | Invoke `/aws-coworker-plan-interaction` with goal: "Create a new VPC" |
| "Show me my AWS costs" | Invoke `/aws-coworker-plan-interaction` with goal: "Retrieve cost summary" |
| "What's my spending this month?" | Invoke `/aws-coworker-plan-interaction` with goal: "Cost analysis" |
| "Check CloudWatch alarms" | Invoke `/aws-coworker-plan-interaction` with goal: "Review CloudWatch alarms" |
| "Deploy this to staging" | Invoke `/aws-coworker-execute-nonprod` with the user's deployment context |
| "Push to production" | Invoke `/aws-coworker-prepare-prod-change` with the user's change context |
| "Undo the last change" | Invoke `/aws-coworker-rollback-change` with details of what to roll back |

**When in doubt, route through `/aws-coworker-plan-interaction`.** It's always safer to go through the safety model than to execute AWS CLI directly.

**Invocation pattern:** When invoking a command on behalf of the user, pass their original request as the input/goal so the command workflow has full context of what the user wants to achieve.

---

## Execution Flow (Mandatory)

**This is the ONLY valid execution path for mutations:**

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. USER REQUEST                                                    │
│     "Create an S3 bucket"                                           │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│  2. PLAN: /aws-coworker-plan-interaction                            │
│     - Discovery                                                     │
│     - Generate plan with rollback                                   │
│     - Present plan to user                                          │
│     - Output: "To execute, run /aws-coworker-execute-nonprod"       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│  3. USER APPROVAL                                                   │
│     "Yes, proceed" / "Approved"                                     │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│  4. EXECUTE: /aws-coworker-execute-nonprod    ← MANDATORY STEP      │
│     - Guardrail validation                                          │
│     - Execute approved commands                                     │
│     - Verify results                                                │
└─────────────────────────────────────────────────────────────────────┘
```

### Critical Rule

**After user approves a plan, you MUST invoke `/aws-coworker-execute-nonprod` to execute.**

DO NOT:
- Run AWS CLI commands directly, even with user approval
- Treat plan approval as execution approval
- Skip the execute command "for simplicity"

The plan command generates a plan. The execute command runs it. These are separate steps BY DESIGN for audit trails and safety.

---

## Safety Model (Non-Negotiable)

1. **Announce before action** — Always state AWS profile and region before any operation
2. **Read-only by default** — Unknown profiles are treated as read-only
3. **Explicit approval for mutations** — Never modify resources without user confirmation
4. **Production is protected** — Production changes generate CI/CD artifacts, not direct CLI
5. **Consult guardrails** — Check `skills/org/aws-governance-guardrails/` before mutations

---

## AWS Profile Handling

Before ANY AWS operation:

1. Identify which profile will be used
2. State it explicitly: "I will use profile `{profile}` in region `{region}`"
3. If profile is unknown/new, treat as read-only until configured
4. Check environment classification (sandbox/dev/staging/prod)

---

## Quick Reference: Available Commands

| Command | Purpose |
|---------|---------|
| `/aws-coworker-plan-interaction` | Plan any AWS operation (start here) |
| `/aws-coworker-execute-nonprod` | Execute approved plan in non-prod |
| `/aws-coworker-prepare-prod-change` | Generate CI/CD changes for production |
| `/aws-coworker-rollback-change` | Design and execute rollback |
| `/aws-coworker-bootstrap-account` | Set up new AWS account |
| `/aws-coworker-audit-library` | Audit AWS Coworker health |

---

## What NOT to Do

❌ `aws s3 ls` — Direct CLI without going through AWS Coworker
❌ `aws ec2 describe-instances` — Even read-only should announce profile first
❌ `aws ce get-cost-and-usage` — Cost queries should also go through AWS Coworker
❌ `aws cloudwatch get-metric-data` — Monitoring queries need profile announcement
❌ `aws ec2 terminate-instances` — Never without plan, approval, and guardrail check
❌ Any AWS CLI in production — Must go through CI/CD
❌ **Running `aws` commands after plan approval** — Use `/aws-coworker-execute-nonprod` instead
❌ **Treating plan approval as execution permission** — Plan and execute are separate commands

**Rule of thumb:** If it starts with `aws `, it MUST go through AWS Coworker commands. User approval of a plan authorizes the EXECUTE COMMAND to run—not you running CLI directly.

---

## Always-Agent Mode

AWS Coworker operates in **Always-Agent Mode**: every request spawns at least one agent via the Task tool. This ensures consistent execution paths, comprehensive audit trails, and efficient handling of enterprise workloads.

### Why Always-Agent Mode?

| Benefit | Explanation |
|---------|-------------|
| **Consistency** | Same execution path regardless of task complexity |
| **Auditability** | Every operation tracked through agent invocation |
| **Scalability** | Seamless transition from simple to complex tasks |
| **Enterprise-scale patterns** | Designed for environments where complex tasks are common |

Simple tasks like "list my S3 buckets" work perfectly fine — they use a single agent rather than spawning parallel workers. The overhead is minimal; the consistency benefits are significant.

### Model Hierarchy (MANDATORY)

AWS Coworker uses a tiered model strategy for cost efficiency:

| Role | Model | Purpose |
|------|-------|---------|
| **Primary (Orchestrator)** | Opus (recommended) | Reasoning and orchestration: planning, decisions, user communication, result synthesis |
| **Sub-Agents (Read-Only)** | Haiku | Fast parallel discovery, audits, cost analysis |
| **Sub-Agents (Mutations)** | Sonnet | Thorough state-change analysis |

**CRITICAL:** When spawning sub-agents, you MUST explicitly pass the `model` parameter:

```
Task:
  subagent_type: "general-purpose"    # NOT "Bash" - Bash bypasses agent context
  model: "haiku"                      # MUST specify model explicitly
  prompt: "..."

CORRECT:
  Discovery/Audit task → model: "haiku"
  Mutation task        → model: "sonnet"

WRONG:
  Discovery/Audit task → model: "sonnet"  (wastes cost)
  Any task             → (no model specified)  (NEVER - always specify model)
```

Read-only operations include: `describe-*`, `list-*`, `get-*`, `head-*`, and any audit or analysis of existing state.

**Why this matters:** If you run AWS Coworker with Opus 4.5, Opus handles the "thinking" while Haiku does the parallelized "doing" — best quality where it counts, optimized cost at scale.

### Sub-Agent Delegation (MANDATORY)

**The orchestrator MUST NEVER run AWS CLI commands directly.**

All AWS CLI execution MUST go through sub-agents using the defined agent identities from `.claude/agents/`.

```
FORBIDDEN (orchestrator running directly):
  ⏺ Bash(aws s3api create-bucket --bucket my-bucket ...)
  ⏺ Bash(aws ec2 describe-instances ...)

ALSO FORBIDDEN (raw Bash agents - no agent identity):
  ⏺ 3 Bash agents finished        ← WRONG: bypasses agent context
     ├─ Verify AWS identity
     ├─ Discover VPC

REQUIRED (Task with agent identity and model):
  ⏺ Task(Create S3 bucket) Sonnet 4.5
    prompt: "You are acting as aws-coworker-executor..."
  ⏺ Task(Discover EC2 instances) Haiku 4.5
    prompt: "You are acting as aws-coworker-planner..."
```

**How to identify correct vs incorrect execution:**
| Output Pattern | Status | Problem |
|----------------|--------|---------|
| `Task(description) Haiku 4.5` with planner identity | ✅ Correct | Agent context included |
| `Task(description) Sonnet 4.5` with executor identity | ✅ Correct | Agent context included |
| `Bash agents finished` | ❌ Wrong | No agent identity, bypasses context |
| `Bash(aws ...)` | ❌ Wrong | Orchestrator running directly |

**Task tool invocation pattern (from agent definitions):**
```yaml
Task:
  description: "Create S3 bucket"
  subagent_type: "general-purpose"   # NOT "Bash"
  model: "sonnet"
  prompt: |
    You are acting as aws-coworker-executor.

    ## Permission Context
    User has approved this operation.

    ## Target
    Profile: {profile}
    Region: {region}

    ## Approved Action
    {command}
```

**If a sub-agent refuses to execute:**
1. DO NOT bypass and run the command directly
2. Re-spawn the sub-agent with clearer authorization context
3. If still refusing, report to user and request explicit confirmation
4. NEVER fall back to orchestrator running commands directly

### Configurable Thresholds

Thresholds determine **how many agents** to spawn, not **whether** to spawn agents.

**Configuration:** `config/orchestration-config.md`

| Factor | Single Agent | Parallel Agents |
|--------|--------------|-----------------|
| Resources | < 50 | >= 50 |
| Regions | <= 3 | > 3 |
| Accounts | <= 3 | > 3 |
| Est. Time | < 5 min | > 5 min (advise user) |

### User Advisement

For tasks above thresholds, advise the user before proceeding:

```
This task involves:
- 847 resources across 8 regions
- Estimated time: 8-10 minutes

I'll work in parallel (8 agents) to minimize time. Do you want to proceed?
```

### Permission Delegation

When spawning sub-agents via the Task tool:
1. Read thresholds from `config/orchestration-config.md`
2. Pass the user's explicit approval scope
3. Constrain sub-agent to approved actions only
4. Sub-agents cannot expand scope beyond user's approval

### Result Aggregation

After execution (single or parallel):
1. Wait for all sub-agents to complete
2. Merge results into coherent summary
3. Present unified response to user
4. Note any partial failures clearly

---

## Two Modes of Operation

AWS Coworker has two distinct modes. Identify which mode applies BEFORE proceeding:

| Mode | User Intent | Applies When |
|------|-------------|--------------|
| **Mode 1: AWS Interaction** | Using AWS Coworker to work with AWS | User wants to query, create, modify, or delete AWS resources |
| **Mode 2: Extending AWS Coworker** | Building/modifying AWS Coworker itself | User wants to add skills, agents, commands, or change architecture |

### Mode 1: AWS Interaction (Default)

Follow all instructions in this file. Route requests through AWS Coworker commands. This is the primary use case.

### Mode 2: Extending AWS Coworker

**STOP. This mode has different guardrails.**

If ANY of these apply, switch to Mode 2:
- Adding new skills, agents, or commands
- Modifying existing AWS Coworker components
- Adding support for new AWS services
- Discussing directory structure or file organization
- Keywords: "extend", "customize", "add to" AWS Coworker

**Required:** Read `skills/meta/aws-coworker-development/SKILL.md` and follow its guardrails.

This skill defines:
- Mandatory reads before any action
- Plan-then-execute workflow
- DO / DO NOT behaviors
- Testing requirements
- Verification checklist

---

## Skills Reference

AWS Coworker agents use these skills for guidance:

| Skill | Purpose |
|-------|---------|
| `skills/aws/aws-cli-playbook/` | AWS CLI patterns for all services |
| `skills/aws/aws-well-architected/` | Well-Architected Framework alignment |
| `skills/aws/aws-governance-guardrails/` | Never-do / always-do policies |
| `skills/org/aws-org-strategy/` | Organization's account structure |

Commands load the appropriate skills automatically. You don't need to read them manually unless debugging.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jason-c-dev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jason-c-dev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
