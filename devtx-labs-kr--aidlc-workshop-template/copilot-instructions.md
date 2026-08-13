## aidlc-workshop-template

> AIDLC (AI-Driven Development Life Cycle) workshop project. Provides structured software development workflow with Inception, Construction, and Operations phases.

# AGENTS.md

## Project Overview

AIDLC (AI-Driven Development Life Cycle) workshop project. Provides structured software development workflow with Inception, Construction, and Operations phases.

## Language Preference

- Respond in Korean except for technical terms
- Technical terms remain in English (e.g., API, function, class)

## Workflow Instructions

### AIDLC Workflow

Follow the AIDLC workflow defined in `.kiro/steering/aws-aidlc-rules/core-workflow.md`:

1. **Inception Phase**: Requirements analysis, design, planning
   - Workspace Detection (always)
   - Reverse Engineering (if brownfield)
   - Requirements Analysis (always)
   - User Stories (conditional)
   - Workflow Planning (always)
   - Application Design (conditional)
   - Units Generation (conditional)

2. **Construction Phase**: Detailed design and code generation
   - Functional Design (conditional, per-unit)
   - NFR Requirements (conditional, per-unit)
   - NFR Design (conditional, per-unit)
   - Infrastructure Design (conditional, per-unit)
   - Code Generation (always, per-unit)
   - Build and Test (always)

3. **Operations Phase**: Deployment and operations (future expansion)

### Critical Rules

- **User approval required at every step**: Must get user confirmation before proceeding
- **Load detailed rules**: Read corresponding rule files from `.kiro/aws-aidlc-rule-details/` before executing each stage
- **Load common rules**: At workflow start, load:
  - `common/process-overview.md`
  - `common/session-continuity.md`
  - `common/content-validation.md`
  - `common/question-format-guide.md`
- **Content validation**: Validate Mermaid diagrams, ASCII art before file creation
- **Audit logging**: Record all user inputs and AI responses in `aidlc-docs/audit.md`

## Directory Structure

```
aidlc-workshop/
├── .kiro/
│   ├── agents/                     # Custom agent config (CLI)
│   ├── steering/                   # AIDLC workflow rules
│   └── aws-aidlc-rule-details/     # Detailed rule docs
├── AGENTS.md                       # This file
└── README.md
```

## Code Generation Guidelines

### TDD Option

Choose TDD (Test-Driven Development) approach during code generation:
- **TDD**: Test-first, higher quality, 1.5-2x time/tokens
- **Standard**: Normal code generation, baseline time

### Application Code Location

- **Application code**: Generate in workspace root (never in `aidlc-docs/`)
- **Documentation**: Generate only in `aidlc-docs/` directory

## Testing Instructions

- Build and test instructions generated in `aidlc-docs/construction/build-and-test/`
- Refer to documents in that directory for per-unit test execution

## Session Continuity

- Check `aidlc-docs/aidlc-state.md` when resuming existing session
- Load artifacts and decisions from previous stages
- Review previous conversation context from audit log

## Security Considerations

- Never include sensitive info (API keys, passwords) directly in code
- Use environment variables or AWS Secrets Manager

---
> Source: [devtx-labs-kr/aidlc-workshop-template](https://github.com/devtx-labs-kr/aidlc-workshop-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
