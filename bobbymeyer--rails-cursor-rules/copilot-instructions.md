## rails-cursor-rules

> This rule file defines the tech stack and conventions for a Rails 8 project.

# Rails 8 Omakase Stack Rules

This rule file defines the tech stack and conventions for a Rails 8 project.

## File Patterns
- `app/**/*.rb`: Ruby files
- `app/**/*.erb`: ERB templates
- `app/javascript/**/*.js`: JavaScript files
- `app/assets/stylesheets/**/*.css`: CSS files
- `config/**/*.rb`: Configuration files
- `test/**/*.rb`: Test files
- `.cursor/rules/docs/*.md`: Documentation files

## Stack Components
- Framework: Ruby on Rails 8 (@docs/rails_core.md)
- Database: PostgreSQL
- Authentication: Rails built-in
- Real-time: Hotwire (@docs/hotwire.md)
- Background Jobs: Solid Queue (@docs/solid_queue.md)
- Testing: Minitest
- Test Coverage: SimpleCov (@docs/simplecov.md)
- Asset Pipeline: Propshaft
- Template Engine: ERB
- Form Builder: Simple Form (@docs/simple_form.md)
- CSS: Plain CSS (no Tailwind)
- JavaScript: Importmaps (@docs/importmaps.md) + Hotwire only
- Pagination: Pagy (@docs/pagy.md)

## Conventions
- Use Rails built-in authentication
- Use Hotwire for real-time features (see @docs/hotwire.md)
- Keep JavaScript minimal and use Hotwire where possible
- Use plain CSS without Tailwind
- Use Simple Form for forms (see @docs/simple_form.md)
- Use Minitest for testing
- Use Solid Queue for background jobs (see @docs/solid_queue.md)
- Use Pagy for pagination (see @docs/pagy.md)
- Use SimpleCov for test coverage (see @docs/simplecov.md)

## File Structure
- Models in `app/models/`
- Controllers in `app/controllers/`
- Views in `app/views/`
- JavaScript in `app/javascript/`
- CSS in `app/assets/stylesheets/`
- Tests in `test/`

## Dependencies
@file .cursor/rules/version.mdc
@file .cursor/rules/docs/*.md 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bobbymeyer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
