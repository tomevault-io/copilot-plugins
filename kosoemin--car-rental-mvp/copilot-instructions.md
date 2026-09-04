## car-rental-mvp

> Follow these files in this priority order:

# AGENTS.md

## Source of Truth

Follow these files in this priority order:

1. `PROJECT_SPEC.md` — requirements and MVP scope
2. `IMPLEMENTATON_PLAN.md` — implementation order
3. `AGENTS.md` — agent behavior and coding rules

Do not contradict `PROJECT_SPEC.md`.

## Core Rules

- Read relevant files before making changes.
- Inspect the existing code before modifying it.
- Implement only the current requested phase/task.
- Do not implement future phases early.
- Do not add features outside the MVP scope.
- Keep changes small and focused.
- Prefer simple, maintainable solutions.
- Do not rewrite working code unnecessarily.
- Do not install dependencies unless necessary.

## Workflow

For each task:

1. Understand
2. Inspect
3. Plan briefly
4. Implement
5. Test
6. Review changes
7. Report results

If a task has an architectural impact or conflicts with the specification, stop and ask before implementing.

## Security

- Never hard-code secrets.
- Never expose MySQL database credentials or JWT secrets.
- Never expose SMS provider secrets.
- Never commit `.env` files containing secrets.
- Enforce server-side PHP authorization for protected data.
- Enforce critical authorization on the server/database, not only in the UI.

## Database

- Follow the schema defined by `PROJECT_SPEC.md`.
- Use `.sql` migration scripts executed via phpMyAdmin / MySQL CLI.
- Add appropriate constraints and indexes.
- Do not modify production data directly.
- Preserve historical booking values such as distance, price/km, and fare.

## Booking

Booking state changes must be validated server-side.

A booking must never be accepted by two drivers.

The transition:

`PENDING → ACCEPTED`

must be atomic.

Do not rely on frontend checks for concurrency or authorization.

## API / External Services

- Keep Google Maps integration isolated.
- Keep SMS integration server-side.
- Use server-side PHP endpoints for sensitive SMS operations.
- Handle external API failures safely.
- Use mock services when real credentials are unavailable for local development.

## Code Quality

- Use clear naming.
- Keep components focused.
- Avoid unnecessary abstraction.
- Avoid duplicated business logic.
- Keep business rules out of presentation components where practical.
- Add validation for user input.
- Handle loading, empty, success, and error states.

## Testing

After implementation:

- Run relevant tests.
- Run lint if configured.
- Run production build when appropriate.
- Check the changed functionality manually when possible.

Do not claim a test passed unless it was actually run.

## Scope Control

Do NOT implement these in MVP unless explicitly requested:

- Online payment
- Chat
- Ratings/reviews
- Live GPS tracking
- Native mobile apps
- AI recommendations
- Surge pricing
- Advanced analytics

## Communication

Keep responses concise.

After completing a task, report:

- What changed
- Tests/checks performed
- Any known issues

Do not provide long explanations unless requested.

## Stop Conditions

Stop and ask for clarification when:

- Requirements conflict.
- A security-sensitive decision is unclear.
- A database architecture change is required.
- An external service/API decision is required.
- The requested change would expand MVP scope.
- You are unsure which existing behavior should be preserved.

---
> Source: [KoSoeMin/car-rental-mvp](https://github.com/KoSoeMin/car-rental-mvp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
