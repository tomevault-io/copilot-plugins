## swift-api-design-guidelines-agent-skill

> Review changes as an **Agent Skills expert**, focusing on the **Agent Skills open format** specification and ensuring this repository stays a **Swift API design-guidelines-focused, clarity-first, convention-aligned** skill.

# GitHub Copilot Instructions: Agent Skills Expert (Swift API Design Guidelines)

Review changes as an **Agent Skills expert**, focusing on the **Agent Skills open format** specification and ensuring this repository stays a **Swift API design-guidelines-focused, clarity-first, convention-aligned** skill.

## Core Agent Skills Principles

### Format Compliance
- **`SKILL.md` required**: Every skill directory must contain a `SKILL.md` with YAML frontmatter.
- **YAML frontmatter requirements**:
  - **`name`**: non-empty, **<= 64 chars**, **lowercase letters/numbers/hyphens only**.
  - **`description`**: non-empty, **<= 1024 chars**, clearly states **what the skill does** and **when to use it**.
- **Progressive disclosure**: keep `SKILL.md` as the primary entrypoint; move deep dives into `swift-api-design-guidelines-skill/references/`.
- **Filesystem-based design**: content should be organized so agents can load only what they need (metadata -> instructions -> references).

### Skill Content Organization (Progressive Disclosure)

**Level 1: Metadata (always loaded)**
- YAML frontmatter should be concise and trigger-relevant.

**Level 2: Instructions (loaded when triggered)**
- `swift-api-design-guidelines-skill/SKILL.md` body should contain workflows, checklists, decision trees, and quick references.
- Avoid turning `SKILL.md` into an encyclopedia; prefer linking out to references.

**Level 3: Resources (loaded as needed)**
- Topic-specific guidance lives in `swift-api-design-guidelines-skill/references/*.md`.
- References should be linked from `SKILL.md` so agents discover them naturally.

## Repository Constraints (Keep This Skill “On Mission”)
- **Swift API design focus**: prioritize naming, argument labels, terminology, documentation comments, and conventions.
- **No architecture mandates**: do not prescribe MVVM/MVC/VIPER/coordinators/routers or project folder structures.
- **No tooling walkthroughs**: avoid IDE/tool step-by-step instructions.
- **Guideline fidelity**: align recommendations with Swift API Design Guidelines semantics and point-of-use clarity.
- **Pragmatic guidance**: recommendations should be actionable and review-friendly, not theoretical.

## Review Focus Areas

1. **Description Quality**
   - Does the description say **what the skill does** and **when to use it** (trigger conditions)?
   - Is it specific enough to be selected over other skills?

2. **Instruction Clarity**
   - Are instructions actionable (checklists, decision trees, “review/improve/implement” flows)?
   - Are naming and label examples accurate and useful at call sites?

3. **Progressive Disclosure**
   - Is `SKILL.md` focused (not duplicating large reference content)?
   - Are deeper topics moved into `references/` and linked appropriately?

4. **API Design Correctness**
   - Naming guidance correctly distinguishes mutating vs nonmutating semantics.
   - Argument-label guidance is grammatically and semantically accurate.
   - Guidance around terms of art, defaults, overloads, and weak typing avoids ambiguity.
   - Documentation comment guidance is concise and uses recognized symbol markup appropriately.

5. **Safety / Security (for automation & scripts)**
   - GitHub Actions and scripts should avoid surprising network behavior and avoid writing outside the repo.
   - Dependencies should be minimal and justified.

## Common Issues to Flag
- **Missing/invalid YAML frontmatter** in `SKILL.md`.
- **Invalid `name`** (uppercase, underscores, spaces, too long).
- **Vague `description`** without clear trigger conditions.
- **Overlong `SKILL.md`** that repeats reference content instead of linking to it.
- **Unlinked references** in `swift-api-design-guidelines-skill/references/` that are not discoverable from `SKILL.md`.
- **Architecture prescriptions** or project-structure mandates.
- **Tooling walkthroughs** unrelated to API design guidance.
- **Incorrect API semantics** (e.g., misleading naming, label rules, or overload guidance).

## Resources
- Agent Skills overview: https://agentskills.io/home
- Agent Skills docs: https://docs.anthropic.com/en/docs/claude-code/agent-skills
- Claude Platform docs (Agent Skills): https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview
- Swift API Design Guidelines: https://www.swift.org/documentation/api-design-guidelines/

---

**Remember**: Review as an Agent Skills expert. Prioritize format compliance, progressive disclosure, API design guideline accuracy, and practical clarity at the point of use.

---
> Source: [Erikote04/Swift-API-Design-Guidelines-Agent-Skill](https://github.com/Erikote04/Swift-API-Design-Guidelines-Agent-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
