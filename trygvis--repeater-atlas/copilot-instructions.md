## repeater-atlas

> - Purpose: Public repeater directory and map with club-managed data.

# Repeater Atlas: Agent Notes

## Quick Context

- Purpose: Public repeater directory and map with club-managed data.
- Users: Public readers; club admins/editors manage repeater entries.
- Deployment: Single-instance service (no horizontal scaling in MVP).
- Data access: Public read; admin editing only on the main site domain.
- Club views: CNAME-based read-only views; embeddable minimal layout.

## Stack

- Rust + Axum (SSR) + Askama templates.
- HTMX for progressive enhancement.
- PostgreSQL and Diesel (including migrations).
- Auth: signed/encrypted session cookies; in-memory session store.
- Map: Leaflet + OSM tiles (provider swappable).

## Conventions

- Generate migrations via `diesel migration generate`, not by hand.
- Before closing an issue, include a note on what was done. Summarize code and
  design changes and anything related that was done.
- `schema.rs` should never be edited manually. Run the database migrations with
  `just db-init`, it will reinitialize the database and run the migrations.
  Diesel will generate `schema.rs` automatically.
- There are pre-commit hooks installed that will be executed automatically. If
  these fail, there is an issue that has to be fixed before continuing. DO NOT
  run with `--no-verify` as a workaround.
- _NEVER_ commit any code without explicit instructions. Ask if a commit should
  be made, only if everything builds and all tests pass.
  - Between stating the code and doing the actual commit, run `prek run`. This
    will reformat all code and do other tasks to ensure consistency. Include
    whatever changes it does into the commit.
- Commit messages:
  - All commit messages should be put into a file called `commit.tmp`, and used
    with the `-F` flag to `git commit`. After putting the message into the file,
    run `bin/prettier commit.tmp` to get it properly formatted.
  - The commit message should be a very short summary of what changed. It SHOULD
    be less than 50 characters, MUST NOT be more than 75. After the first line,
    there should be a blank line, then the rest of the messages.
- Before working, make a note of the current Git commit SHA. Might be useful
  later if asked to review incoming changes.
- When working on bugs:
  - Review the bug to make sure that everything is clear. Have a low bar to ask
    for more details.
  - Make a plan for yourself. It SHOULD include these steps
    - A testing step to run all tests
    - A step to update the documentation under `/docs`
  - When working on an epic, each bug should follow this process.
- After editing files, ensure that they are consistent with the project's rules
  by running `prek run --files $FILES` where `$FILES` is the files that has been
  edited.
- Follow these steps when changing code:
  1. Run `just all` to make sure that everything compiles. This must pass before
     continuing.
  2. Run `just test` to make sure that all tests pass before continuing.
  3. Check that the database migrations work with `just db-init`. This will
     clean the database and run all migrations
  4. Check that the data loading still works with `just db-setup`. This will do
     `db-init` again, but also load the data.

## Documentation

These must be read before doing any work.

- `docs/code.md`
- `docs/data-model.md`
- `docs/design/README.md`
- `docs/testing.md`
- `docs/design/pages/README.md` — index of page-level specs; read the relevant
  page spec(s) before working on any page.

## CLI tools and commands

- Run `just` to get the overview of available recipes and agent notes.
- This project uses a CLI ticket system for task management. Run `tk help` when
  you need to use it. Use `tk` commands to manage tickets. The ticket body
  (description, design, acceptance criteria) may be edited directly with the
  Edit tool on `.tickets/<id>.md` when updating the body content; use
  `tk add-note` (via stdin) for timestamped notes, and `tk close`/`tk reopen`
  for status changes.
  - When using `tk add-note` always pass the note as input to stdin.
- Generate migrations via `diesel migration generate`, not by hand.
  - Requires `diesel_cli` installed
    (`cargo install diesel_cli --no-default-features --features postgres`).
  - Apply the migration with `diesel migration run`, it will automatically
    update `src/schema.rs`.

## Testing Notes

- Access the database via `psql`. Use `repeater_atlast` as database and username
  for normal database inspections.
- Integration tests use a dedicated local Postgres via `DATABASE_URL`.
- Fixture loader can drop/recreate the test DB and run migrations.
- Tests commit changes (no transaction rollback).

---
> Source: [trygvis/repeater-atlas](https://github.com/trygvis/repeater-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
