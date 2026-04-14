## ai-workflow-runner

> 1. [Critical Rules](#1-critical-rules) - MUST read before ANY task

# Agent Guidelines

## Table of Contents

1. [Critical Rules](#1-critical-rules) - MUST read before ANY task
2. [Skills](#2-skills) - MUST load before testing tasks
3. [Workflows](#3-workflows) - Step-by-step processes
4. [Story Creation](#4-story-creation) - Templates for creating stories
5. [Quality Checks](#5-quality-checks) - Post-implementation validation
6. [Knowledge Base Reference](#6-knowledge-base-reference) - Standards documentation index

---

## 1. Critical Rules

> **STOP: Read this section before starting ANY task.**

### Pre-Work Requirements (MANDATORY)

Before starting ANY task, you MUST:

1. **Read the Project Standards**: `.knowledge-base/technical/standards/index.md`
2. **Read the System Documentation**: `.knowledge-base/technical/application-design/index.md`

These documents contain critical rules and context that must inform all work performed in this codebase.

---

## 2. Skills

> **STOP: This section OVERRIDES all workflow instructions. Load skills BEFORE executing any workflow steps.**

Using specialized skills for testing is **MANDATORY**. You MUST call the `skill` tool to load the appropriate skill BEFORE doing any testing-related work.

### When to Load Skills

| Task Type                          | Required Skill                | Load Command                                |
| ---------------------------------- | ----------------------------- | ------------------------------------------- |
| Write/review unit tests            | `typescript-unit-testing`     | `skill(name="typescript-unit-testing")`     |
| Write/review E2E/integration tests | `typescript-e2e-testing`      | `skill(name="typescript-e2e-testing")`      |
| Writing code                       | `typescript-clean-code`       | `skill(name="typescript-clean-code")`       |
| Writing frontend code              | `vercel-react-best-practices` | `skill(name="vercel-react-best-practices")` |
| Debug frontend code                | `agent-browser`               | `skill(name="agent-browser")`               |
| Debug failing tests                | Load skill matching test type | See above                                   |
| Test quality review                | Load BOTH skills              | Load both before review                     |

### Skill Loading Rules

1. **Load skills FIRST** - Before reading test files, before executing workflows, before any analysis
2. **Skills override workflows** - If a workflow doesn't mention skills, load them anyway
3. **When in doubt, load** - If task involves `*.spec.ts`, `*.e2e-spec.ts`, or test review, load the skill
4. **Both for reviews** - Test quality reviews require BOTH skills

### Verification Checkpoint

Before proceeding with any testing task, confirm:

- [ ] Did I load the required skill(s)?
- [ ] Did I read the skill instructions completely?
- [ ] Am I applying the skill's patterns and checklists?

**Failure to load skills will result in non-compliant work that must be redone.**

---

## 3. Workflows

### Standard Task Workflow

For every task:

1. Read both index files mentioned in [Critical Rules](#1-critical-rules)
2. Apply the standards and rules from those documents
3. Proceed with the requested task while adhering to project guidelines
4. Run [Quality Checks](#5-quality-checks) after implementation

### Test Review Workflow Example

```
1. User requests: "Review tests for story 4.2"
2. FIRST: Load skill(name="typescript-unit-testing")
3. SECOND: Load skill(name="typescript-e2e-testing")
4. THEN: Execute the workflow steps
```

---

## 4. Story Creation

### Task 1 Template (MANDATORY)

When creating a story, AI agents MUST add the following as **Task 1** (before any implementation tasks):

```markdown
- [ ] **Task 1: Read Required Standards (MANDATORY)** (AC: All)
  - [ ] Read `.knowledge-base/technical/standards/backend/error-handling.md` - NO try-catch in use cases, errors bubble
  - [ ] Read `.knowledge-base/technical/standards/backend/coding-style.md` - Naming, SOLID, TypeScript
  - [ ] Read `.knowledge-base/technical/standards/global/commenting.md` - Zero-tolerance for obvious comments
  - [ ] Read `.knowledge-base/technical/standards/backend/logging.md` - Minimal logging only
  - [ ] Read `.knowledge-base/technical/standards/backend/validation.md` - class-validator with ErrorCode
  - [ ] Read `.knowledge-base/technical/standards/testing/unit-testing.md` - AAA pattern, @golevelup/ts-jest
  - [ ] Read `.knowledge-base/technical/standards/testing/e2e-testing.md`
  - [ ] (Add skill-specific needed, including coding, debugging, testing, or design pattern skills)
  - [ ] (Add any other applicable standards for this story)
```

### Final Task Template (MANDATORY)

Every story MUST include this as the **last task**:

```markdown
- [ ] **Final Task: Quality Checks**
  - [ ] Run `npm run lint` - Fix any linting issues
  - [ ] Run `npm run format` - Verify code formatting
  - [ ] Run `npm run typecheck` - Ensure type safety
```

### Test Design References

When creating stories, reference the appropriate test design files:

| Scope  | File Location                                                        |
| ------ | -------------------------------------------------------------------- |
| EPIC   | `_bmad-output/implementation-artifacts/test-design-epic-{number}.md` |
| System | `_bmad-output/planning-artifacts/test-design-system.md`              |

---

## 5. Quality Checks

After completing ANY implementation, run the following commands in order:

```bash
npm run lint
npm run format
npm run typecheck
```

---

## 6. Knowledge Base Reference

### Quick Lookup

| Question                             | Standard                                                                                                                                                  |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Should I log this?                   | [logging](/.knowledge-base/technical/standards/backend/logging.md)                                                                                        |
| Do I need try-catch?                 | [error-handling](/.knowledge-base/technical/standards/backend/error-handling.md)                                                                          |
| How to name this?                    | [coding-style](/.knowledge-base/technical/standards/backend/coding-style.md)                                                                              |
| How to structure tests?              | [unit-testing](/.knowledge-base/technical/standards/testing/unit-testing.md) / [e2e-testing](/.knowledge-base/technical/standards/testing/e2e-testing.md) |
| What error code?                     | [error-codes](/.knowledge-base/technical/standards/backend/error-codes.md)                                                                                |
| How to validate?                     | [validation](/.knowledge-base/technical/standards/backend/validation.md)                                                                                  |
| Multiple strategies?                 | [Factory Pattern](/.knowledge-base/technical/standards/backend/design-pattern-factory.md)                                                                 |
| Need auto-discovery with decorators? | [Factory + Explorer](/.knowledge-base/technical/standards/backend/design-pattern-factory-and-explorer.md)                                                 |
| Sequential data enrichment?          | [Chain of Responsibility](/.knowledge-base/technical/standards/backend/design-pattern-chain-of-repository.md)                                             |

### Standards Categories

#### Backend Standards

| Category            | Standards                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mandatory**       | [coding-style](/.knowledge-base/technical/standards/backend/coding-style.md), [error-handling](/.knowledge-base/technical/standards/backend/error-handling.md), [logging](/.knowledge-base/technical/standards/backend/logging.md), [error-codes](/.knowledge-base/technical/standards/backend/error-codes.md), [usecases](/.knowledge-base/technical/standards/backend/usecases.md), [domain-events](/.knowledge-base/technical/standards/backend/domain-events.md), [kafka](/.knowledge-base/technical/standards/backend/kafka.md), [telemetry](/.knowledge-base/technical/standards/backend/telemetry.md), [validation](/.knowledge-base/technical/standards/backend/validation.md) |
| **Design Patterns** | [Factory Pattern](/.knowledge-base/technical/standards/backend/design-pattern-factory.md), [Factory + Explorer](/.knowledge-base/technical/standards/backend/design-pattern-factory-and-explorer.md), [Chain of Responsibility](/.knowledge-base/technical/standards/backend/design-pattern-chain-of-repository.md)                                                                                                                                                                                                                                                                                                                                                                    |
| **Best Practices**  | [api](/.knowledge-base/technical/standards/backend/api.md), [models](/.knowledge-base/technical/standards/backend/models.md), [queries](/.knowledge-base/technical/standards/backend/queries.md), [migrations](/.knowledge-base/technical/standards/backend/migrations.md)                                                                                                                                                                                                                                                                                                                                                                                                             |

#### Testing Standards

| Standard                                                                     | Key Points                                              |
| ---------------------------------------------------------------------------- | ------------------------------------------------------- |
| [unit-testing](/.knowledge-base/technical/standards/testing/unit-testing.md) | @golevelup/ts-jest, mongodb-memory-server, 80% coverage |
| [e2e-testing](/.knowledge-base/technical/standards/testing/e2e-testing.md)   | Replicate main.ts, --runInBand, 8s Kafka waits          |

#### Global Standards

| Standard                                                                    | Key Points                                     |
| --------------------------------------------------------------------------- | ---------------------------------------------- |
| [commenting](/.knowledge-base/technical/standards/global/commenting.md)     | Zero-tolerance obvious comments, minimal JSDoc |
| [security](/.knowledge-base/technical/standards/global/security.md)         | No hardcoded secrets, OWASP Top 10             |
| [code-quality](/.knowledge-base/technical/standards/global/code-quality.md) | Type safety, DI, SOLID                         |
| [conventions](/.knowledge-base/technical/standards/global/conventions.md)   | Project structure, version control             |

#### Frontend Standards

| Standard                                                                        | Key Points                            |
| ------------------------------------------------------------------------------- | ------------------------------------- |
| [components](/.knowledge-base/technical/standards/frontend/components.md)       | Single responsibility, minimal props  |
| [css](/.knowledge-base/technical/standards/frontend/css.md)                     | Consistent methodology, design tokens |
| [accessibility](/.knowledge-base/technical/standards/frontend/accessibility.md) | Semantic HTML, 4.5:1 contrast         |
| [responsive](/.knowledge-base/technical/standards/frontend/responsive.md)       | Mobile-first, 44x44px touch targets   |

#### Architecture

| Document                                                                                               | Description                                            |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| [Clean Architecture Guide](/.knowledge-base/technical/application-design/clean-architecture-nestjs.md) | NestJS implementation of Clean Architecture principles |

### Index Documents

For comprehensive documentation, refer to:

- **Standards Index**: [.knowledge-base/technical/standards/index.md](/.knowledge-base/technical/standards/index.md)
- **System Documentation Index**: [.knowledge-base/technical/application-design/index.md](/.knowledge-base/technical/application-design/index.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arch-playground) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
