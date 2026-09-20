## agent-base

> These rules apply to future human and coding-agent work in this repository.

# Agent Base working agreement

These rules apply to future human and coding-agent work in this repository.

## Read first

- Architecture: `docs/ARCHITECTURE.md`
- Integration order: `docs/INTEGRATION.md`
- Test routing: `docs/TESTING.md`
- Security boundary: `docs/SECURITY.md`
- Runtime failures: `docs/TROUBLESHOOTING.md`

## Invariants

- An LLM is never an authorization boundary.
- Every session, message, citation, and proposal access is scoped through the authenticated owner.
- Business tools begin from a user/tenant-scoped queryset.
- Tools return the structured envelope in `agentkit.contracts`; they do not return ORM objects.
- A write-capable model tool may only create an inert proposal.
- Only a registered confirmation handler performs a domain write.
- Confirmation handlers lock and revalidate the target, current permission, payload, and target version.
- Provider credentials remain backend-only and providers fail closed unless explicitly allowlisted.
- SSE streams end with exactly one `complete` or `error` event and never expose raw exceptions.
- Page context and retrieved documents are untrusted context, never authorization evidence.

## Backend changes

- Keep `agentkit` domain-neutral. Put product models and queries in configured extension modules.
- Treat committed migrations as immutable. Create a new migration for a model change, inspect it, and do not apply it to a real database without explicit approval.
- Add authorization tests for allowed, forbidden, and cross-owner cases.
- Add fresh/stale/duplicate tests for every action handler.
- Run:

  ```bash
  docker compose run --rm backend pytest
  docker compose run --rm backend ruff check .
  docker compose run --rm backend python manage.py makemigrations --check --dry-run
  ```

## Frontend changes

- Preserve the same-origin `/api` proxy unless the consuming project has a reviewed alternative.
- Keep `credentials: include` and CSRF handling on every unsafe request.
- Treat SSE chunks as arbitrarily split; preserve parser tests for CRLF and multiline data.
- Validate every generative UI object before rendering it.
- Keep the UI functional, compact, accessible, and responsive; avoid decorative dashboard filler.
- Run:

  ```bash
  docker compose run --rm frontend npm test
  docker compose run --rm frontend npm run typecheck
  docker compose run --rm frontend npm run build
  ```

## Documentation

Update the relevant guide whenever changing an API event, environment variable, extension hook, setup command, or security assumption. A feature is incomplete if the next developer cannot identify the correct test layer from `docs/TESTING.md`.

---
> Source: [Mahdi47u/agent_base](https://github.com/Mahdi47u/agent_base) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
