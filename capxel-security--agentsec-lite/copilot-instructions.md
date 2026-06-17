## agentsec-lite

> Scan OpenClaw workspaces and skills for exposed secrets, weak permissions, unsafe scripts, network exposure, insecure config, memory leakage, git hygiene issues, and risky runtime settings. Use when Codex needs to audit an agent setup, review a skill before installation, or generate a prioritized security report with actionable fixes.


# AgentSec Lite

## Quick Start

- Run `agentsec-lite scan` to audit the current workspace.
- Run `agentsec-lite scan --path <workspace>` to inspect a different root.
- Run `agentsec-lite scan --checks secrets,skills,network` to focus on specific categories.
- Run `agentsec-lite scan --json` when another tool needs machine-readable output.
- Run `agentsec-lite audit-skill <path-or-url>` before installing a third-party skill.

## Workflow

1. Detect the workspace root or accept the user-provided path.
2. Scan files for the eight security categories.
3. Prioritize `critical` and `high` findings first.
4. Recommend concrete fixes: rotate exposed credentials, tighten permissions, bind services to `127.0.0.1`, remove risky scripts, and move secrets into environment variables.

## Interpretation

- Treat `A` and `B` grades as healthy but still review findings.
- Treat `C` and `D` grades as needing remediation before wider use.
- Treat `F` grades or any `critical` finding as a stop-ship condition.

## Commands

```bash
agentsec-lite scan
agentsec-lite scan --json
agentsec-lite scan --checks secrets,permissions,skills
agentsec-lite audit-skill ./skills/example
agentsec-lite audit-skill https://github.com/example/example-skill
```

---
> Source: [Capxel-Security/agentsec-lite](https://github.com/Capxel-Security/agentsec-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
