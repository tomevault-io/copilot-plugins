## divorcehelper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a collection of specialized Claude Code subagents designed to help individuals navigate the divorce process. Each subagent is a separate AI assistant with its own context window, system prompt, and tool access, stored in `.claude/agents/`.

The subagents can be invoked individually or work together to provide comprehensive divorce guidance across legal, financial, parenting, and safety domains.

## Subagent Structure

All subagents are stored in `.claude/agents/` as Markdown files with YAML frontmatter:

```markdown
---
name: agent-name
description: When and why to use this agent
tools: Read, Write, Bash, Grep, Glob  # Optional
model: sonnet  # Optional
---

Detailed system prompt for the agent...
```

Claude Code will automatically delegate to these subagents based on the task and their descriptions.

## Core Subagent Architecture

The system consists of specialized subagents, each with distinct responsibilities:

1. **Safety & Crisis Agent** - Identifies and mitigates risk (domestic violence, stalking, coercive control, relocation risks)
2. **Legal Navigator Agent** - Maps the divorce process from filing through post-decree
3. **Document & Evidence Agent** - Intake, indexing, and preservation of financials, messages, kids' records
4. **Financial Forensics & Budget Agent** - Asset/debt inventory, characterization, dissipation detection, budget building
5. **Co-Parenting & Child-Focus Agent** - Parenting plans, best interests alignment, UCCJEA multi-state compliance
6. **Negotiation & Settlement Agent** - Proposal organization, BATNA/WATNA analysis, mediation/collaborative structures
7. **Court & Compliance Agent** - Deadline tracking, docket reminders, post-judgment tasks
8. **Tax & Benefits Agent** - Filing status, dependency claims, alimony tax treatment, COBRA, beneficiary updates
9. **Well-Being & Logistics Agent** - Therapist referrals, support groups, housing/insurance checklists

## Data Model

The system uses three core data structures:

- **Vault**: Encrypted store for IDs, finances, kids' records
- **Timeline**: Dated log of events, orders, and communications
- **Red Flags**: Violence, relocation, asset flight, coercive control, substance abuse indicators

## Legal & Ethical Guardrails

When implementing features, strictly adhere to these boundaries:

- **Never advise illegal surveillance, recording, or tracking**
- **No evidence spoliation** - preserve all potentially relevant documents/communications
- **No advice to hide funds or assets**
- **Respect court orders** including ATROs (Automatic Temporary Restraining Orders)
- **Recording compliance** - respect state-specific one-party vs all-party consent laws
- **No unauthorized access** - CFAA, Wiretap Act, ECPA compliance required
- **Mandatory disclosure compliance** - failure can trigger sanctions

## Process Flow

The divorce process follows this sequence:

1. **Safety & Stabilization** (Days 0-7) - Safety planning, protective orders, digital hygiene
2. **Resolution Track Selection** - Mediation, collaborative divorce, or litigation
3. **Pre-Filing Prep** (Weeks 1-3) - Document sweep, credit checks, budget creation
4. **Filing & Service** - Petition filing, service, temporary restraints activation
5. **Temporary Orders** (30-90 days) - Parenting schedule, support, status quo orders
6. **Financial Disclosures & Discovery** - Mandatory financial disclosures, evidence preservation
7. **Parenting Plan Development** - Best interests analysis, UCCJEA compliance
8. **Property Division** - Characterization (marital vs separate), valuation, division
9. **Support Determination** - Child support (guideline-based), spousal support (factor-based)
10. **Settlement or Trial** - Package proposals, MSA/parenting plan, or trial preparation
11. **Judgment & Post-Decree** - Asset retitling, QDRO filing, beneficiary updates

## Jurisdictional Awareness

The system must adapt to jurisdiction-specific rules:

- **Community property states** vs **equitable distribution states**
- **ATRO/temporary restraining orders** (e.g., California Fam. Code §2040)
- **Recording laws** (one-party vs all-party consent by state)
- **UCCJEA compliance** for multi-state custody jurisdiction
- **Post-2018 alimony tax treatment** (non-deductible/non-taxable)

## Key Integration Points

When building features, integrate with these external resources:

- **DivorceNet** - Process mapping and jurisdiction-specific rules
- **Reporters Committee** - Recording laws by state
- **DOL QDRO resources** - Retirement plan division requirements
- **Child Welfare Information Gateway** - Best interests factors
- **NCJFCJ UCCJEA resources** - Multi-state custody jurisdiction
- **IRS Pub 504** - Tax implications of divorce

## Working with Subagents

**Viewing available agents**: Use `/agents` command to see all available subagents

**Creating new agents**: Use `/agents` command for guided creation, or create Markdown files in `.claude/agents/` directly

**Modifying agents**: Edit the Markdown files directly, updating the system prompt and tool permissions as needed

**Invoking agents**: Claude Code will automatically delegate based on task, or you can explicitly request an agent by name

## Development Priorities for Subagents

When creating or modifying subagents:

1. **Safety First** - All guidance must prioritize user safety and legal compliance
2. **Evidence Preservation** - Never provide instructions that encourage deletion or hiding of information
3. **Jurisdiction Awareness** - Legal guidance must adapt based on user location
4. **Child-Centric** - Co-parenting subagents must focus on best interests of the child
5. **Clear Boundaries** - Each subagent should have a focused, non-overlapping responsibility
6. **Proactive Guardrails** - Build legal/ethical constraints directly into system prompts

## Communication Standards

All user-facing communication should follow the BINF principle:
- **Brief** - Concise, actionable guidance
- **Informative** - Fact-based with legal citations
- **Neutral** - De-escalatory tone
- **Firm** - Clear boundaries and requirements

## Security Requirements

- **Encryption** for vault storage
- **No unauthorized access** to third-party accounts
- **Audit trails** for all document access/modifications
- **Spoliation prevention** - block deletion of litigation-relevant content
- **Password/2FA enforcement** for digital hygiene recommendations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BuildThingsThatBuildthings) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
