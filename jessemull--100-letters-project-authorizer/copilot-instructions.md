## 010-architecture

> Authorizer architecture: handler boundaries, packaging, CloudFormation.


# Architecture

Read `docs/ARCHITECTURE.md`.

- Keep auth logic in `src/index.ts` (or clearly named modules under `src/` if split later).
- Do not expand this Lambda into a general API.
- Preserve Webpack → zip → S3 → CloudFormation deploy topology unless migrating with human approval.
- Fail closed on auth errors (`Unauthorized`).

---
> Source: [jessemull/100-letters-project-authorizer](https://github.com/jessemull/100-letters-project-authorizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
