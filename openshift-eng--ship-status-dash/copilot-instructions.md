## docs

> Keep documentation up-to-date alongside code changes


* When your change alters setup steps, usage instructions, or architecture,
  update the relevant README in the same PR.
* When API endpoints are added, removed, or modified, update
  `API_ENDPOINTS.md`.
* When configuration options, environment variables, or CLI flags change,
  update the relevant section of the root `README.md`.
* When new conventions, workflows, or tooling are introduced, consider
  adding or updating an `.apm/instructions/` file so that AI coding
  assistants stay aligned with the project's practices.
* When authentication flows, credential handling, or deployment security
  change, update `.apm/instructions/security.instructions.md`.
* Documentation and code belong in the same PR; never treat a docs update
  as a follow-up task.
* Do not use em dashes when writing docs. Use commas, parentheses, or periods instead.

---
> Source: [openshift-eng/ship-status-dash](https://github.com/openshift-eng/ship-status-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-08 -->
