## spellbooks-avatar

> - `app/` holds domain code: models, controllers, jobs, mailers, views, and Stimulus controllers under `app/javascript/controllers`.

# Repository Guidelines

## Project Structure & Module Organization
- `app/` holds domain code: models, controllers, jobs, mailers, views, and Stimulus controllers under `app/javascript/controllers`.
- `config/` stores application settings; environment-specific files live in `config/environments/`.
- `db/` tracks migrations, schema snapshots, and seed data in `db/seeds.rb`.
- `test/` contains the Minitest suite with fixtures under `test/fixtures/`; use `docs/` for long-form notes and `public/` for static assets.

## Build, Test, and Development Commands
- `bin/setup` installs gems, prepares the database, and compiles assets; run after cloning or pulling major changes.
- `bin/rails server` boots the app at `http://localhost:3000` for local development.
- `bin/rails test` runs the entire suite; append a path (e.g., `bin/rails test test/models/avatar_test.rb`) to scope runs.
- `bundle exec rubocop` enforces Rails Omakase style; use `bundle exec rubocop -A` for safe auto-corrections.
- `bundle exec brakeman` performs static security analysis before shipping controller or model changes.

## Coding Style & Naming Conventions
- Follow Ruby conventions: two-space indentation, `snake_case` for methods and variables, and `CamelCase` for classes and modules.
- Keep controllers, jobs, and mailers focused; move business logic into POROs under `app/models` or `app/services` (create the folder if absent).
- Align Stimulus controller filenames with class names (e.g., `app/javascript/controllers/avatar_controller.js` exports `AvatarController`).
- Run RuboCop locally before committing to maintain consistent formatting.

## Testing Guidelines
- Use the default Minitest framework; add files as `<subject>_test.rb` with classes named `<Subject>Test`.
- Place system tests in `test/system/` and rely on fixtures from `test/fixtures/` or factories you introduce.
- Target regressions with focused tests and keep high coverage on models and controllers handling business rules.

## Commit & Pull Request Guidelines
- Existing history favors short, imperative summaries (e.g., `Initital doc commit`); keep subjects ≤ 50 characters with details in the body.
- Reference issues with `Fixes #ID`, list schema changes, and include validation steps or screenshots for UI-impacting work.
- Ensure lint (`bundle exec rubocop`) and tests pass locally before opening a PR; request review before merging.

## Security & Configuration Tips
- Manage secrets through Rails credentials (`bin/rails credentials:edit`) and avoid committing plaintext keys or environment files.
- The default `sqlite3` database suits development; configure production via environment variables and update `config/database.yml` accordingly.
- Validate file uploads for size and content type when introducing Active Storage attachments to prevent abuse.

---
> Source: [procload/spellbooks-avatar](https://github.com/procload/spellbooks-avatar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
