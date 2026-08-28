## server-key-injection

> `ski` is a Python package for a service which issues short-lived SSH

# Development guide

## Project purpose

`ski` is a Python package for a service which issues short-lived SSH
certificates and a client which loads the certificate identity into the user's
existing `ssh-agent`. AsyncSSH is the planned SSH implementation library.

The currently implemented surface is intentionally only the package and command
line foundation. Before introducing the issuing protocol, agree on the trust
model, authentication method, principals, validity limits, certificate options,
and audit requirements.

## Layout

- `src/ski/`: importable application package.
- `tests/`: behavioural tests for public interfaces.
- `.agents/`: local copies of requested workflow skill guides.

## Architecture

Read `developer/architecture.md` for an overview of the system architecture, planned functionality, and design decisions.

## Development workflow

Use Python through `uv`; do not invoke a system Python directly for project
tasks. Keep the package layout under `src/` and add tests under `tests/`.

Environment configuration is loaded once at CLI startup with `python-dotenv`.
Search `./.env`, then `$HOME/.ski.env`, then `/home/ski/etc/env`, stopping at the
first existing file. Existing process environment variables take precedence.

Run these commands before handing off a change:

```console
uv run ruff format
uv run ruff check --fix
uv run ty check
uv run pytest
```

Follow `.agents/tdd.md` for test-driven changes: add one behavioural test,
make it pass with the smallest public implementation, then continue. Follow
`.agents/git-commit.md` when creating a commit.

## Guardrails

- Never implement a protocol that transmits, logs, persists, or commits a user's
  SSH private key.
- Treat CA private keys, issued certificates, agent sockets, and authorization
  policy as sensitive material. Keep them out of Git, test fixtures, logs, and
  error messages.
- Require explicit design review before changing identity verification,
  certificate principals, validity periods, critical options, extensions, or
  host/user certificate policy.
- Prefer short certificate lifetimes and least-privilege principals. Do not
  silently broaden access after issuance.
- Validate all untrusted protocol inputs and reject malformed or unsupported key
  types and certificate requests.
- Do not add network listeners, authentication backends, secrets management, or
  external integrations without a documented configuration and threat model.
- Do not bypass the required formatter, linter, type checker, or test suite.

## Specialization workflow

We never generate code ad hoc unless the user specifically requests it. A
direct request is a scoped bypass for debugging, an ad-hoc fix, or an
experiment; Git provides the recovery path for that work.

Otherwise, follow this directory-based workflow for the epic currently being
worked on. Do not modify artifacts belonging to another independent epic or
user story.

1. **User-story step.** Create a directory named
   `developer/<YYYY-MM-DD>-<epic-slug>/`. Put the epic's structured user stories
   in `user-stories.md` and any relevant reviews in that directory, for example
   `security-review.md` or `refactoring-review.md`.
2. **Ticket step.** Commit the current epic's relevant user-story or review
   file before this step. Develop it into actionable tickets in `tickets.md` in
   the same directory, ordered for implementation. Use the TDD skill while
   developing tickets.
3. **Code-generation step.** Commit that epic's `tickets.md` before beginning
   this step. Generate code from the tickets using the TDD skill. When a ticket
   is complete, commit it using the git-commit skill; only then proceed to the
   next ticket.

---
> Source: [isotopp/server-key-injection](https://github.com/isotopp/server-key-injection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
