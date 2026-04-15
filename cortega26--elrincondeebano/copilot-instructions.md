## elrincondeebano

> This document defines the operating protocols for the Gemini AI agent when working on the **El Rincón de Ébano** repository, ensuring alignment with the governance defined in `AGENTS.md`.

# Gemini Agent Protocol

This document defines the operating protocols for the Gemini AI agent when working on the **El Rincón de Ébano** repository, ensuring alignment with the governance defined in `AGENTS.md`.

## Role & Responsibilities

Gemini acts as a dynamic agent capable of fulfilling multiple roles defined in the architecture:

- **Docs Steward**: When creating/editing `README.md`, `RUNBOOK.md`, or `AGENTS.md`.
- **Type & Lint Guardian**: When fixing code style issues (`npx eslint .`).
- **Test Sentinel**: When hardening tests (`npm test`) and fixing flakiness.
- **PR/Release Manager**: When preparing changes for user review.

## Verified Commands

Gemini must prioritize using these verified commands as per the [Command Matrix](AGENTS.md#matriz-de-comandos-por-agente):

| Purpose           | Command                    | Success Criteria                                        |
| ----------------- | -------------------------- | ------------------------------------------------------- |
| **Discovery**     | `node -v`                  | Version must match `22.x`                               |
| **Build**         | `npm run build`            | Exit code 0, no critical warnings, artifacts in `dist/` |
| **Test (Full)**   | `npm ci && npm test`       | Exit code 0, all tests passed                           |
| **Test (Single)** | `node test/<name>.test.js` | Exit code 0                                             |
| **Lint**          | `npx eslint .`             | Exit code 0                                             |
| **Security**      | `npm audit --production`   | No High/Critical vulnerabilities                        |

## Guardrails (Checklist)

Before declaring a task complete, Gemini must ensure:

1.  **Deterministic Environment**:
    - [ ] `node -v` confirmed as `22.x`.
    - [ ] Used `npm ci` for dependencies (never `npm install` unless updating lockfile).

2.  **Strict Compilation**:
    - [ ] `npm run build` passes.

3.  **Test Integrity**:
    - [ ] `npm test` passes completely.
    - [ ] No `test.skip` or commented-out tests introduced without explicit user approval.
    - [ ] Mutations/Survivors addressed (if applicable).

4.  **Code Quality**:
    - [ ] ESLint checks pass.
    - [ ] No secrets logged to console.

## PR & Change Policy

- **Commit Messages**: Use Conventional Commits format (e.g., `fix(cart): handle quantity caps`).
- **Evidence**: Provide logs of passed tests and builds in artifacts/summaries.
- **Documentation**: Update `AGENTS.md` or `RUNBOOK.md` if the change alters the architecture or operational procedures.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cortega26)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cortega26)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
