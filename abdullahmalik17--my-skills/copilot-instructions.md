## my-skills

> Use when users need to create new documents or work with tracked changes.

# My Skills - Reusable Claude Code Skills Library

## Overview

This repository contains a curated collection of 50+ reusable skills designed to extend Claude Code's capabilities across various software engineering domains. Each skill encodes battle-tested patterns, domain expertise, and procedural knowledge that transforms Claude from a general-purpose agent into a specialized expert.

## Repository Structure

```
My_skills/
├── CLAUDE.md                    # This file - project documentation
├── README.md                    # Project overview
└── .claude/
    └── skills/                  # Skills library
        ├── SKILLS-INDEX.md      # Comprehensive skills catalog
        └── [skill-name]/        # Individual skill packages
            ├── SKILL.md         # Skill definition and instructions
            ├── scripts/         # Executable automation scripts
            ├── references/      # Deep-dive documentation
            └── assets/          # Output templates and resources
```

## What Are Skills?

Skills are **NOT documentation**. They are self-contained packages that encode:

- **Frozen decisions**: Judgment calls that took hours or days to determine
- **Production gotchas**: Edge cases and failure modes discovered in real deployments
- **Expert patterns**: Workflows that domain experts use but rarely document
- **Tool combinations**: Proven integrations between multiple technologies

**The test:** If you would need to recreate this knowledge after deletion, and Claude doesn't already know it, then it belongs in a skill.

## Skill Categories

This repository organizes skills into eight categories:

### 1. MCP-Backed Skills (3)
Require MCP (Model Context Protocol) servers for functionality.
- `browsing-with-playwright` - Browser automation
- `fetching-library-docs` - Library documentation retrieval
- `researching-with-deepwiki` - Repository research

### 2. Infrastructure (3)
Container orchestration and Kubernetes workflows.
- `containerizing-applications` - Docker and Helm patterns
- `operating-k8s-local` - Local Kubernetes development
- `deploying-cloud-k8s` - Cloud deployment pipelines

### 3. Application Development (4)
Full-stack application patterns and frameworks.
- `building-nextjs-apps` - Next.js 16+ patterns
- `scaffolding-fastapi-dapr` - Microservices architecture
- `configuring-better-auth` - Authentication systems
- `building-rag-systems` - Retrieval-augmented generation

### 4. UI/Frontend (4)
Chat interfaces and component styling.
- `styling-with-shadcn` - Component library integration
- `building-chat-interfaces` - AI chat UI patterns
- `building-chat-widgets` - Embeddable chat widgets
- `streaming-llm-responses` - Real-time streaming

### 5. Development Practices (2)
Debugging and production operations.
- `systematic-debugging` - Four-phase debugging methodology
- `operating-production-services` - SLO alerting and incident response

### 6. Document Processing (2)
Office file manipulation and generation.
- `working-with-spreadsheets` - Excel and financial models
- `working-with-documents` - Word, PDF, PowerPoint

### 7. Meta Skills (2)
Skills for creating and managing skills.
- `creating-skills` - Skill creation framework
- `installing-skill-tracker` - Usage analytics

### 8. Integration (2)
External service and tool integration patterns.
- `building-mcp-servers` - MCP server development
- `internal-comms` - Corporate communication templates

## Working with This Repository

### For Claude: Skill Usage Guidelines

When working with this repository:

1. **Skill Selection**
   - Check the `description` field in SKILL.md frontmatter for trigger conditions
   - Match user intent, not just keywords
   - Verify exclusions (some skills specify "NOT when...")
   - Choose the most specific skill when multiple could apply

2. **Progressive Loading**
   - Skill metadata (name + description) is always loaded (~100 tokens)
   - SKILL.md body loads when skill triggers (<5000 tokens)
   - References and scripts load as needed during execution

3. **Skill Dependencies**
   - Some skills work together (see SKILLS-INDEX.md for dependencies)
   - Follow cross-references in "Related Skills" sections
   - Chain skills when tasks span multiple domains

4. **Verification**
   - Each skill includes `scripts/verify.py` for validation
   - Run verification before packaging or distribution

### For Developers: Contributing Skills

#### Before Creating a Skill

Ask: **Does Claude actually need this?**

- Standard library usage → NO
- Well-documented public patterns → NO
- Spent hours debugging it → YES
- Broke production because of it → YES
- Domain expert knowledge → YES

#### Skill Creation Workflow

1. **Understand with examples**
   - Gather concrete use cases
   - Identify trigger phrases
   - Document expected inputs/outputs

2. **Plan reusable contents**
   - Identify repetitive code → scripts/
   - Identify reference docs → references/
   - Identify output templates → assets/

3. **Initialize skill**
   ```bash
   python3 .claude/skills/creating-skills/scripts/init_skill.py <skill-name>
   ```

4. **Implement skill**
   - Write YAML frontmatter (name, description)
   - Author markdown instructions
   - Create bundled resources
   - Keep SKILL.md under 500 lines

5. **Package skill**
   ```bash
   python3 .claude/skills/creating-skills/scripts/package_skill.py <path/to/skill>
   ```

6. **Iterate based on usage**
   - Test on real tasks
   - Refine based on struggles
   - Update documentation

#### Skill Quality Standards

Every skill must meet these criteria:

- **Clear trigger**: "Use when..." in description field
- **Concise**: Under 500 lines / 5000 tokens in SKILL.md
- **Verified**: Includes `scripts/verify.py` that exits 0 (pass) or 1 (fail)
- **Battle-tested**: Contains production gotchas, not theory
- **Structured references**: References are one level deep only

#### Description Guidelines (CRITICAL)

The `description` field determines when skills activate. Write it correctly:

```yaml
# BAD: Summarizes workflow
description: Use when executing plans - dispatches subagent per task with code review

# BAD: Process details
description: Use for TDD - write test first, watch it fail, write minimal code

# GOOD: Just triggering conditions
description: Use when executing implementation plans with independent tasks

# GOOD: With exclusions
description: |
  Use when users need to create new documents or work with tracked changes.
  NOT when converting between formats (use converting-documents skill).
```

**Description checklist:**
- Start with "Use when..." for triggering conditions
- Include symptoms, situations, contexts
- Add "NOT when..." for collision avoidance
- NEVER summarize the skill's process
- Maximum 1024 characters

## Skill Anatomy

### Required: SKILL.md

```yaml
---
name: skill-name
description: |
  Use when [triggering conditions].
  NOT when [exclusions].
---

# Skill Title

Instructions for using this skill...
```

### Optional: Bundled Resources

#### scripts/
Executable code for deterministic reliability.
- Use when: Same code rewritten repeatedly
- Example: `scripts/rotate_pdf.py`
- Benefits: Token efficient, deterministic

#### references/
Documentation loaded as needed into context.
- Use when: Claude needs reference material
- Example: `references/api_docs.md`
- Benefits: Keeps SKILL.md lean

#### assets/
Files used in output, not loaded into context.
- Use when: Templates, logos, fonts needed
- Example: `assets/template.pptx`
- Benefits: Separates output from documentation

## Key Principles

### 1. Concise is Key

The context window is a shared resource. Only include information Claude doesn't already possess.

**Challenge each sentence:** "Does Claude really need this explanation?"

Prefer concise examples over verbose explanations.

### 2. Progressive Disclosure

Load information in layers:
1. Metadata (always loaded)
2. SKILL.md body (when triggered)
3. Resources (as needed)

### 3. Appropriate Freedom

Match specificity to task fragility:
- **High freedom** (text instructions): Multiple valid approaches
- **Medium freedom** (pseudocode/parameters): Preferred patterns with variation
- **Low freedom** (specific scripts): Fragile operations requiring consistency

### 4. Domain Expert Test

> Would a senior engineer in this domain say "yes, this captures what we actually do"?

Common failures:
- Tutorial-level content Claude already knows
- Missing edge cases experts handle automatically
- Idealized patterns that break in production
- Wrong tool choices for scale

## Verification

### Verify All Skills
```bash
for skill in .claude/skills/*/; do
  python3 "$skill/scripts/verify.py" 2>/dev/null || echo "FAIL: $skill"
done
```

### Verify Specific Skill
```bash
python3 .claude/skills/creating-skills/scripts/verify.py /path/to/skill
```

### Diagnostic Mode
```bash
python3 .claude/skills/creating-skills/scripts/verify.py /path/to/skill --verbose
```

## Evaluation Flow for New Skills

```
User submits skill idea
        │
        ▼
┌─────────────────────────────┐
│ Does Claude already know it? │──YES──► REJECT (not a skill)
└─────────────────────────────┘
        │ NO
        ▼
┌─────────────────────────────┐
│ Did it cause production pain?│──YES──► HIGH PRIORITY
└─────────────────────────────┘
        │ NO
        ▼
┌─────────────────────────────┐
│ Would expert recreate it?   │──YES──► MEDIUM PRIORITY
└─────────────────────────────┘
        │ NO
        ▼
      REJECT
```

## Merge vs New Skill Decision

| Scenario | Action |
|----------|--------|
| Extends existing skill domain | Add to existing skill's references/ |
| New domain, similar trigger | Evaluate collision, pick clearer name |
| New domain, unique trigger | Create new skill |
| Overlaps multiple skills | Merge into most relevant, cross-reference |

## Skill Enhancement Checklist

When enhancing existing skills to expert level:

1. Search for production patterns experts actually use
2. Create `references/*.md` for deep patterns
3. Update SKILL.md with summary and links
4. Run `verify.py` to ensure validation passes
5. Test with real-world scenarios

## What NOT to Include in Skills

Skills should only contain essential files. Do NOT create:
- README.md (redundant with SKILL.md)
- INSTALLATION_GUIDE.md
- QUICK_REFERENCE.md
- CHANGELOG.md

Skills are for AI agents, not human documentation.

## Git Workflow

This repository follows professional commit practices:

1. **Atomic commits**: One logical change per commit
2. **Descriptive messages**: Clear, concise commit descriptions
3. **Conventional format**:
   ```
   <type>: <description>

   [optional body]
   ```
   Types: feat, fix, docs, refactor, test, chore

4. **Commit every step**: Track progress incrementally
5. **No force pushes**: To main/master without explicit approval

## Professional Standards

When working with this repository:

- **Objective guidance**: Technical accuracy over validation
- **No unnecessary files**: Only create required artifacts
- **Concise communication**: Professional, technical tone
- **Rigorous standards**: Apply same standards to all contributions
- **Evidence-based**: Base decisions on facts and testing

## Getting Started

1. **Browse skills**: Review `.claude/skills/SKILLS-INDEX.md`
2. **Understand structure**: Read skill examples
3. **Create new skill**: Use `creating-skills` skill
4. **Verify quality**: Run verification scripts
5. **Contribute**: Follow quality standards above

## References

- **Skills Index**: `.claude/skills/SKILLS-INDEX.md` - Comprehensive catalog
- **Creating Skills**: `.claude/skills/creating-skills/SKILL.md` - Creation guide
- **Design Patterns**: `.claude/skills/creating-skills/references/design-patterns.md`
- **Workflows**: `.claude/skills/creating-skills/references/workflows.md`

## Future Skill Ideas

Areas with no skills yet:
- Database migrations at scale (Alembic, Drizzle patterns)
- Feature flags and gradual rollouts
- Incident response playbooks
- Observability instrumentation (OpenTelemetry)
- GraphQL federation patterns
- WebSocket scaling patterns

---

**Repository Purpose**: Provide reusable, battle-tested skills that encode domain expertise and production knowledge for AI-assisted software engineering workflows.

**Maintenance Philosophy**: Quality over quantity. Every skill must justify its existence by encoding knowledge Claude doesn't already possess.

---
> Source: [AbdullahMalik17/My_skills](https://github.com/AbdullahMalik17/My_skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
