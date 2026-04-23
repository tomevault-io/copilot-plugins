## m365

> C2 North Star Alignment and Architecture Decision Records (ADR) Requirement


# North Star Alignment and ADR Requirement

**Status:** MANDATORY
**Owner:** All AI Assistants (Composer, Codex, GPT-5)

---

## Rule: ALWAYS Align with NORTHSTAR.md and Document Architecture Decisions

**MANDATORY:** ALL work (code changes, investigations, implementations, documentation, etc.) MUST:
1. Align with the vision and goals defined in `Operations/NORTHSTAR.md`
2. Verify alignment BEFORE starting work (not just for significant changes)
3. Document significant architecture decisions in ADR format
4. Reference North Star metrics and objectives when making decisions
5. Include alignment check in action log entries

**This is Step 1 of the Mandatory Agent Workflow.** See `.cursor/rules/agent-workflow-mandatory.mdc` for complete workflow.

---

## North Star Alignment Requirements

### Before ANY Work (MANDATORY)

**MANDATORY:** Before starting ANY work (not just significant changes):

1. **Read Operations/NORTHSTAR.md** - Understand the vision, goals, and success metrics
2. **Check Alignment** - Verify the work aligns with:
   - Mission Statement
   - Vision 2025
   - Core Success Metrics
   - Technical Excellence Goals
   - Competitive Advantages

3. **If Misaligned:**
   - **STOP** immediately
   - Explain the misalignment
   - Propose alternatives that align with North Star
   - Request explicit approval for any deviation

4. **Document Alignment:**
   - Record alignment check result
   - Include in action log entry (Step 5 of workflow)
   - Reference in execution plan updates

### North Star Metrics to Consider

When making decisions, consider the North Star metrics defined in `Operations/NORTHSTAR.md`:

**M365 AI Workforce Key Metrics:**
- 39 AI agent personas managing Microsoft 365 operations
- M365-only tooling (SharePoint, Teams, Power Automate, Power BI, Outlook, Azure AD)
- Self-service platform (SMARTHAUS maintenance-only)
- Zero extra software cost (M365 tools only)
- Department-based organization (10 departments, 39 agents)
- Automation targets: 90% task automation, 50% manual reduction
- Quality metrics: 99.9% uptime, <2s response time, 100% security compliance
- Business metrics: 95% user satisfaction, +25% efficiency

**See `Operations/NORTHSTAR.md` for complete metrics and KPIs.**

---

## Architecture Decision Records (ADR) Requirement

### When to Create an ADR

**MANDATORY:** Create an ADR for any decision that:
- Affects system architecture, structure, or design
- Impacts multiple services or components
- Introduces new technologies or patterns
- Changes security, performance, or scalability characteristics
- Deviates from existing patterns or conventions
- Has long-term consequences or trade-offs

**Examples Requiring ADR:**
- Choosing a new database technology
- Changing authentication approach
- Introducing a new service or agent
- Changing core architecture patterns
- Adding new dependencies or frameworks
- Major API design changes

### ADR Format

**Location:** `docs/adr/ADR-###-short-title.md` (create `docs/adr/` directory if missing)

**When Required:**
- **v1:** ADRs are optional but recommended for major architectural decisions
- **v2+:** ADRs may be required for significant changes (as specified in Execution Plan)

**Template:**
```markdown
# ADR-###: [Short Title]

**Status:** [Proposed | Accepted | Rejected | Deprecated | Superseded]
**Date:** YYYY-MM-DD
**Deciders:** [Team/Individual]
**Context:** [What is the issue we're addressing?]
**Decision:** [What is the change we're proposing or have agreed to?]
**Consequences:** [What becomes easier or more difficult?]
**North Star Alignment:** [How does this align with Operations/NORTHSTAR.md?]
**Alternatives Considered:** [What other options did we consider?]
**References:** [Links to related docs, issues, PRs]
```

### ADR Process

1. **Propose:** Create ADR in `docs/adr/` directory
2. **Review:** Team reviews ADR for North Star alignment
3. **Decide:** Accept, reject, or request modifications
4. **Document:** Update ADR status and link from relevant code/docs
5. **Track:** Reference ADR in Operations/ACTION_LOG.md and Operations/EXECUTION_PLAN.md

---

## Integration with Other Rules

This rule is **Step 1** of the Mandatory Agent Workflow. See `.cursor/rules/agent-workflow-mandatory.mdc` for complete workflow.

This rule works alongside:
- **Mandatory Agent Workflow:** This is Step 1 (pre-work checklist)
- **Plan-First Execution:** Step 2 requires execution plan verification (after North Star check)
- **Action Log Requirement:** Step 5 requires logging alignment check result
- **Change Approval Protocol:** Step 3 requires including alignment in approval request
- **Project Status Updates:** Step 6 requires documenting alignment impact
- **Execution Plan Updates:** Step 7 requires linking to execution plan

---

## Enforcement

**MANDATORY:** This rule applies to ALL AI assistants working on SmartHaus M365 AI Workforce.

**Before ANY work (not just significant changes):**
1. **CHECK** North Star alignment (Step 1 of workflow)
2. **VERIFY** execution plan association (Step 2 of workflow)
3. **CREATE** ADR if required (for significant architectural decisions)
4. **DOCUMENT** alignment check in action log (Step 5 of workflow)
5. **UPDATE** logs and status documents (Steps 5-7 of workflow)

**Code Review Checklist:**
- [ ] Does this align with Operations/NORTHSTAR.md vision?
- [ ] Does this support North Star metrics?
- [ ] Is an ADR required for this change?
- [ ] Has the ADR been created and reviewed?
- [ ] Are trade-offs documented?

---

## Reference

- **North Star Document:** `Operations/NORTHSTAR.md`
- **ADR Directory:** `docs/adr/` (create if missing)
- **Architecture Overview:** `Operations/NORTHSTAR.md` (Technical Architecture section)
- **Execution Plan:** `Operations/EXECUTION_PLAN.md`

---

## Examples

### ✅ Good: North Star Aligned Decision

**Decision:** Implement iOS-first user onboarding flow

**North Star Alignment:**
- ✅ Supports app-first development priority
- ✅ Aligns with hyper-local neighbor-to-neighbor ethos
- ✅ Enhances user experience and simplicity goals

**ADR Required:** Yes (ADR-XXXX-ios-onboarding.md)

### ❌ Bad: Misaligned Decision

**Decision:** Build web-first with mobile as secondary

**North Star Alignment:**
- ❌ Violates app-first development priority
- ❌ Contradicts North Star technical excellence goals
- ❌ Not aligned with user experience principles

**Action:** REJECT and require app-first approach

---

*"Every decision should move us closer to the North Star."*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SmartHausGroup) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
