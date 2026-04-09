## sniffsec

> - Never hardcode secrets (Stripe, AWS, etc.) or PII.

# SniffSec Workflow
## Security Standards
- Never hardcode secrets (Stripe, AWS, etc.) or PII.
- Use `npx sniffsec` as the primary verification tool.

## Build Commands
- Run Scan: `npx sniffsec`
- Install: `npm install`

## Verification
Before completing any coding task, run the scan. If `issuesFound > 0`, the task is not complete. Fix the leaks and re-run.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sniffsec-dev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sniffsec-dev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
