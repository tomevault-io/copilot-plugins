## aurelio-living

> This is a Spree Commerce 5.1.7 application for Aurelio Living, an e-commerce platform for premium home & living products.

# Aurelio Living - Spree Commerce Project Rules

## Project Context
This is a Spree Commerce 5.1.7 application for Aurelio Living, an e-commerce platform for premium home & living products.

## Technology Stack
- Ruby 3.3.0
- Rails 8.0.3
- Spree Commerce 5.1.7
- PostgreSQL (Neon database)
- WSL environment

## Development Environment
- Server runs on port 3001 (WSL compatible)
- Admin panel: http://localhost:3001/admin
- Store front: http://localhost:3001
- Database: Neon PostgreSQL

## Branding Requirements
- Replace "Spree" with "Aurelio Living" throughout the application
- Remove all "upgrade to Enterprise Edition" messages
- Maintain Spree functionality while customizing branding

## Code Style
- Follow Rails conventions
- Use descriptive variable names
- Comment complex logic
- Maintain Spree gem compatibility

## Database
- Uses Neon PostgreSQL
- Environment variables: DATABASE_URL, DATABASE_DIRECT_URL
- Run migrations with: bin/rails db:migrate

## Common Commands
- Start server: bin/rails server
- Clear cache: bin/rails tmp:clear
- Run migrations: bin/rails db:migrate
- Reset admin: ruby reset_admin_user.rb

## Troubleshooting
- CSRF errors: Clear browser cache, use incognito mode
- Port conflicts: Use port 3001 instead of 3000
- Gem errors: Check Gemfile for missing dependencies
- Database issues: Verify DATABASE_URL in .env file

## File Structure
- app/views/spree/ - Spree view overrides
- app/assets/stylesheets/spree/ - Custom Spree styles
- app/assets/javascripts/spree/ - Custom Spree JavaScript
- config/initializers/ - Rails configuration
- db/migrate/ - Database migrations

## Security
- Never commit .env files
- Use environment variables for secrets
- Keep SECRET_KEY_BASE secure
- Validate all user inputs

## Performance
- Use Rails caching where appropriate
- Optimize database queries
- Minimize asset compilation time
- Use background jobs for heavy tasks

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pathtoresiliencebv) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
