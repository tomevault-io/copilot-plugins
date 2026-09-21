## integration-lab

> - Work in dev. prod contains validated milestones only.

# Development rules
- Work in dev. prod contains validated milestones only.
- Make small, informative commits (feat:, fix:, test:, docs:, chore:).
- No author mentions, signatures or Co-authored-by trailers in commit messages.
- Keep existing Git identity; do not modify identity configuration.
- Java owns execution, assertions, validation and persistence.
- React uses JavaScript. Use PostgreSQL migrations, not automatic schema generation.
- Secrets come from environment variables; never commit .env or live webhook credentials.
- Run backend tests, frontend build and relevant end-to-end checks before promoting prod.
- Do not claim unexecuted checks passed.
- No microservices, Kubernetes or AI dependency in MVP.
- Default to local-only exposure and explicit allowed target origins.

---
> Source: [Dilunder/integration-lab](https://github.com/Dilunder/integration-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
