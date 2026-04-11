## blitzentrapper

> This is a Ruby on Rails 6.0.6 web application (Blitzen Trapper band site) with a React frontend via Webpacker.

# AGENTS.md

## Cursor Cloud specific instructions

This is a Ruby on Rails 6.0.6 web application (Blitzen Trapper band site) with a React frontend via Webpacker.

### Key facts

- **Ruby 3.1.0** installed via rbenv at `~/.rbenv/versions/3.1.0`
- **PostgreSQL 16** local instance (trust auth configured for localhost)
- **Node.js >=18** + **Yarn >=1.22** for Webpacker/JS assets
- No dedicated linter (rubocop/eslint) is configured in this repo
- One integration test exists: `bin/rails test`

### Running the dev server

The injected environment includes production secrets (`RAILS_ENV=production`, `DATABASE_URL`, etc.). You **must** override them for local development:

```bash
export PATH="$HOME/.rbenv/bin:$HOME/.rbenv/shims:$PATH"
export RAILS_ENV=development RACK_ENV=development NODE_ENV=development
unset DATABASE_URL
export COOKIE_SECRET_TOKEN=$(ruby -rsecurerandom -e 'puts SecureRandom.hex(64)')
bin/rails server -p 3000
```

### Database

- Dev DB: `blitzen_db_development` (local PG, no password needed — trust auth)
- Test DB: `blitzen_db_test` (local PG, no password needed)
- Migrations: `RAILS_ENV=development bin/rails db:migrate` (must unset `DATABASE_URL` first)
- The `DATABASE_URL` secret points to a remote Neon Postgres instance; unset it when working with local PG.

### Running tests

```bash
export PATH="$HOME/.rbenv/bin:$HOME/.rbenv/shims:$PATH"
unset DATABASE_URL
bin/rails test
```

### Gotchas

- reCAPTCHA is enabled on user signup and contact pages. The injected `RECAPTCHA_SITE_KEY`/`RECAPTCHA_SECRET_KEY` are for the production domain, not localhost. To create users locally, use `bin/rails console` (e.g. `User.new(login: "name", email: "e@x.com", password: "pw").save!`).
- `yarn install --check-files` may be needed after switching branches if JS deps change; Webpacker's integrity check will refuse to start the server otherwise.
- Standard dev commands are in README.md and `Procfile`. `bin/rails server` starts Puma on port 3000.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TRMW)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TRMW)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
