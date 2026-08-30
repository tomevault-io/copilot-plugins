## endsideout

> This application is a custom LMS software for EndsideOut, a Baltimore-based non-profit focused on wellness education for

# EndsideOut

This application is a custom LMS software for EndsideOut, a Baltimore-based non-profit focused on wellness education for
underprivileged groups.

The goal of this LMS is to be as simple as possible for the organization and students.
See [Readme](README.md) for additional context.

This is a Rails Application using the Solid stack and the built-in Rails Authentication generator.
There are 2 separate sets of models using authentication:

- Users (traditional email/password) used for EndsideOut Employees to access the admin interface
- Students (non-traditional Magic Links) used for Students to access their specific content

## Skills

- Look in the [Skills](./skills) Folder for relevant skills

## Conventions

- Always use rails generators when they exist rather than creating files from scratch
- Don't introduce new gems or javascript packages without informing the user and getting permission
- Ensure valuable test coverage is maintained when adding features. When possible, use TDD.

## ADRs

- See [ADRs](./docs/adrs/README.md) for important architectural decisions

## Commands

- `bin/setup` is idempotent setup for the dev environment
- `bin/rails test` runs the backend tests
- `bin/ci` runs the full local CI suite. Run this when you think you're "done"
- `bin/rubocop` runs linting ruby checks
- `npm run lint` runs herb linting for ERB.

---
> Source: [rubyforgood/endsideout](https://github.com/rubyforgood/endsideout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
