## cdsecurity-skills

> Instructions for Claude when contributing to this repository.

# CLAUDE.md

Instructions for Claude when contributing to this repository.

## What This Repo Is

A library of Claude Code skills for smart contract security, built by CD Security. Each skill is a self-contained capability in its own directory.

## Structure

```
audit-prep/              # Solidity audit preparation (Foundry/Hardhat)
  SKILL.md               # Orchestrator
  VERSION                # Skill version
  references/            # Agent instructions and shared rules
    agents/              # Per-agent instructions (testing, source, infra)
    shared-rules.md      # Common output format for agents
  evals/                 # Test cases and grading scripts

rust-audit-prep/         # Rust/Solana audit preparation (Anchor/native)
  SKILL.md               # Orchestrator
  references/            # Agent instructions, checklist, shared rules
    agents/              # Per-agent instructions (testing, source, infra)
    checklist.md         # Full check reference
    shared-rules.md      # Common output format for agents
    report-template.md   # Detailed report template
```

## Rules

- One skill, one purpose.
- Keep SKILL.md under 500 lines — use references/ for agent-specific instructions.
- No gas optimization checks — out of scope for audit preparation.
- No vulnerability analysis — agents must not perform security assessments.
- Every FAIL finding must include a specific, actionable fix.
- Do not commit secrets, API keys, or personal data.
- Test changes against both eval projects (Hardhat + Foundry) before submitting.

---
> Source: [CDSecurity/cdsecurity-skills](https://github.com/CDSecurity/cdsecurity-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
