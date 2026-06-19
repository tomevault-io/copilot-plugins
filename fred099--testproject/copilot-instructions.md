## testproject

> This project uses PRPs (Project Requirements & Planning) for structured development.

## PRP Workflow

This project uses PRPs (Project Requirements & Planning) for structured development.

### Key Commands

- `/execute-prp <file>` - Execute a PRP with validation
- `/generate-prp <draft>` - Create detailed PRP

### Execution Rules

1. Run validation after EVERY code change
2. Never mock tests - fix root causes
3. Use ULTRATHINK before implementing
4. Follow patterns from existing code
5. Complete ALL validation gates in PRP

PRPs in `PRPs/` contain implementation plans with mandatory validation steps.
Auto-validation hooks run after file changes.

---
> Source: [fred099/testproject](https://github.com/fred099/testproject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
