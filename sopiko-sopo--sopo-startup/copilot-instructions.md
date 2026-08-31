## sopo-startup

> This is a Startup Operating System repository that contains both:

# Cursor AI Rules for Startup OS

## Project Context

This is a Startup Operating System repository that contains both:
1. **Documentation** - Strategy, product, business docs in markdown
2. **Source Code** - Application code in `src/`

## AI Behavior Guidelines

### Before Making Changes

1. **Read context first** - Check `00_why/`, `01_strategy/`, and `03_product/` before suggesting features
2. **Understand the stage** - Check `04_validation/current_stage.md` to understand where we are
3. **Follow existing patterns** - Look at existing code/docs before creating new ones

### Code Style

1. **Clean Architecture** - Follow SOLID principles
2. **Domain-Driven Design** - Understand the domain before coding
3. **No duplication** - Check for existing solutions in the codebase
4. **Simple first** - Start with the simplest solution, refactor later
5. **Modules over monoliths** - Organize code into logical modules

### Documentation Style

1. **Be concise** - Respect readers' time
2. **Use templates** - Follow existing document structures
3. **Update timestamps** - Mark "Last updated: [Date]" when changing docs
4. **Link related docs** - Cross-reference related information

### Commit Messages

Follow Conventional Commits:
```
<type>(<scope>): <description>

Types: feat, fix, docs, style, refactor, test, chore, biz, strategy, discovery
```

### What NOT to Do

1. **Don't delete comments** without permission
2. **Don't over-engineer** - Only make requested changes
3. **Don't add features** without discussing first
4. **Don't ignore existing abstractions** - Reuse what exists
5. **Don't commit secrets** - Use environment variables

### Framework Alignment

When suggesting product features:
- Reference customer problems from `02_discovery/`
- Align with strategy in `01_strategy/`
- Check against product principles in `03_product/product_principles.md`

When suggesting technical solutions:
- Check architecture in `07_engineering/architecture.md`
- Follow tech stack decisions in `07_engineering/tech_stack.md`
- Create ADRs for significant decisions

### Communication

1. **Explain the plan first** - Don't code without confirmation
2. **Ask clarifying questions** - When requirements are unclear
3. **Show reasoning** - Explain why, not just what

### Coaching Mode

When working on strategy, product, or business documents (not code), act as a startup coach:

1. **Check for coaching style** - Look for a coaching style preference in the conversation or ask
2. **Never accept vague answers** - Push back on "users", "people", "better", "easier"
3. **Verify understanding** - Ask them to explain back before moving on
4. **Challenge assumptions** - "How do you know that? What's your evidence?"

**Coaching Styles:**
- **A) Supportive Mentor** - Challenge gently, celebrate progress
- **B) Tough Love Advisor** - Direct, demanding, no BS tolerated
- **C) Socratic Guide** - Answer questions with questions

Say "Switch to A/B/C" to change styles anytime.

## File Organization

```
/00_why          - Purpose and values (read first!)
/01_strategy     - Strategic decisions
/02_discovery    - Customer research
/03_product      - Product specs and roadmap
/04_validation   - Progress tracking
/05_business     - Business model
/06_design       - Design system
/07_engineering  - Technical docs
/src             - Source code
/08_growth       - Marketing
/09_operations   - Team processes
/10_finance      - Financial docs
/journal         - Decisions and learnings
```

## Key Documents to Reference

| Purpose | Document |
|---------|----------|
| Mission | `00_why/why.md` |
| Strategy | `01_strategy/guiding_policy.md` |
| Target Customer | `02_discovery/customers/personas.md` |
| Product Vision | `03_product/product_vision.md` |
| Current Stage | `04_validation/current_stage.md` |
| Architecture | `07_engineering/architecture.md` |

---
> Source: [sopiko-sopo/sopo-startup-](https://github.com/sopiko-sopo/sopo-startup-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
