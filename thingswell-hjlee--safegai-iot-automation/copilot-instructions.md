## safegai-iot-automation

> SafeGAI protects one industrial process by combining AI camera occupancy with equipment and isolated I/O state. The local Ubuntu gateway performs all safety-adjacent decisions and actions. AWS is not in the safety execution path.

# AGENTS.md

## Product
SafeGAI protects one industrial process by combining AI camera occupancy with equipment and isolated I/O state. The local Ubuntu gateway performs all safety-adjacent decisions and actions. AWS is not in the safety execution path.

## Read first
- SOUL.md
- CLAUDE.md
- AIDLC.md
- START_HERE.md
- Relevant specification and nearest component instruction

## Architecture
- Private monorepo
- Go modular-monolith gateway on linux/amd64
- SQLite WAL
- React/TypeScript single frontend with local/cloud adapters
- AWS CDK TypeScript and minimal serverless backend
- External isolated Modbus I/O

## Safety rules
- Missing or stale camera data is never vacancy.
- Only VACANT_CONFIRMED satisfies vacancy.
- Never implement automatic restart from AI vacancy alone.
- Never create cloud machine-control APIs.
- Never directly switch machine power with general-purpose DO.
- R3 changes require T1 and T2 human approval.

## Work method
- One issue, one purpose, one short branch.
- Contract and failure tests before implementation.
- Simulator before hardware.
- Run make verify-fast during work and make verify before PR.
- Do not push, merge, tag, deploy, or operate real I/O without human approval.

---
> Source: [thingswell-hjlee/safegai-iot-automation](https://github.com/thingswell-hjlee/safegai-iot-automation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
