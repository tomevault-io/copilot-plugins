## lcms-core

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Technology Stack

- **Backend**: Ruby on Rails 8.1.1, Ruby 3.4.7
- **Database**: PostgreSQL with `pg_search` for full-text search
- **Search**: Elasticsearch 8.x
- **Frontend**: Hotwire (Turbo, Stimulus), React 16.9, Bootstrap 5.3
- **Asset Pipeline**: esbuild for JavaScript, Sass for CSS
- **Background Jobs**: Resque with Redis, Solid Queue
- **Authentication**: Devise
- **PDF Generation**: Grover (Puppeteer-based, uses Chromium in Docker)
- **Testing**: RSpec
- **Containerization**: Docker, Docker Compose

## Docker Architecture

This project runs entirely in Docker containers. All commands must be executed inside containers using `docker compose run --rm`.

### Docker Services

- **db**: PostgreSQL 17.6 on port 5432
- **redis**: Redis 7 on port 6379
- **rails**: Main Rails application on port 3000
- **resque**: Background job workers
- **css**: CSS asset watcher
- **js**: JavaScript asset builder
- **test**: Test runner

### Docker Image

- Base image: `learningtapestry/lcms-core:dev` (built from `Dockerfile.dev`)
- Includes Ruby 3.4.7, Node.js 22, Yarn, PostgreSQL client, Chromium for PDF generation
- Uses volumes: `bundle`, `postgres-17.6`, `redis-7`

## Development Commands

All commands run inside disposable Docker containers with `--rm` flag.

### Setup

```bash
# Build the Docker image
docker build -f Dockerfile.dev -t learningtapestry/lcms-core:dev .

# Start all services
docker compose up

# Install dependencies
docker compose run --rm rails bundle install
docker compose run --rm rails yarn install

# Database setup
docker compose run --rm rails rails db:create
docker compose run --rm rails rails db:migrate
docker compose run --rm rails rails db:seed
```

### Development

```bash
# Start Rails server
docker compose up rails

# Start all services (Rails, Resque, CSS/JS watchers)
docker compose up

# Start specific services
docker compose up rails resque

# Rails console
docker compose run --rm rails rails console

# Rails commands
docker compose run --rm rails rails routes
docker compose run --rm rails rails db:migrate
docker compose run --rm rails rails db:rollback
```

### Asset Compilation

```bash
# Build JavaScript
docker compose run --rm js yarn build

# Build CSS once
docker compose run --rm rails yarn build:css

# Watch CSS for changes
docker compose up css
```

### Testing

**IMPORTANT**: The `.rspec` file contains a custom `--pattern` that includes plugin specs. When running individual spec files, you MUST override this pattern to avoid "No examples found" errors.

```bash
# Run all tests (uses default pattern from .rspec)
docker compose run --rm test bundle exec rspec

# Run specific test file (MUST override --pattern with the file path)
docker compose run --rm test bundle exec rspec --pattern 'spec/path/to/file_spec.rb' spec/path/to/file_spec.rb

# Run specific test by line (MUST override --pattern with the file path)
docker compose run --rm test bundle exec rspec --pattern 'spec/path/to/file_spec.rb' spec/path/to/file_spec.rb:42

# Setup test database
docker compose run --rm -e RAILS_ENV=test rails rails db:create
docker compose run --rm -e RAILS_ENV=test rails rails db:migrate
```

### Code Quality

```bash
# Run Rubocop
docker compose run --rm rails bundle exec rubocop

# Auto-fix style issues
docker compose run --rm rails bundle exec rubocop -a

# Security scans
docker compose run --rm rails bundle exec brakeman
docker compose run --rm rails bundle exec bundler-audit

# Run all pre-commit checks (Rubocop, Brakeman, YAML syntax, etc.)
docker compose run --rm rails overcommit --run

# Run pre-push checks (RSpec tests)
docker compose run --rm rails overcommit --run pre_push
```

### Git Hooks

Git hooks run checks automatically before commit and push. Since all tools run inside Docker, install the hooks that delegate to Docker containers:

```bash
# Install pre-commit and pre-push hooks
ln -sf ../../script/hooks/pre-commit .git/hooks/pre-commit
ln -sf ../../script/hooks/pre-push .git/hooks/pre-push
```

The hooks will run:
- **pre-commit**: Rubocop, Brakeman, ShellCheck, YAML syntax, trailing whitespace checks
- **pre-push**: RSpec tests

If Brakeman check fails, run interactive mode to review warnings:
```bash
docker compose run --rm -it rails bundle exec brakeman -I
```


### Background Jobs

```bash
# Start Resque workers (via docker-compose)
docker compose up resque

# Manual Resque worker
docker compose run --rm rails env QUEUE=* bundle exec rake resque:work

# Resque scheduler
docker compose run --rm rails bundle exec rake resque:scheduler
```

### Utility Commands

```bash
# Shell access to Rails container
docker compose run --rm rails bash

# Check Ruby version
docker compose run --rm rails ruby --version

# Check syntax of Ruby files
docker compose run --rm rails ruby -c app/helpers/some_helper.rb

# Database console
docker compose run --rm rails rails dbconsole
```

## Application Architecture

### Core Domain Models

**Documents and Materials**
- `Document`: Lesson documents imported from Google Docs, with hierarchical curriculum structure
- `Material`: Supporting materials for lessons (PDFs, worksheets, etc.)
- `DocumentPart` and `MaterialPart`: Rendered content in different contexts (gdoc, PDF)
- Both use `Partable` concern for multi-format content rendering

**Resources and Curriculum**
- `Resource`: Hierarchical curriculum structure (grades, modules, units, lessons)
- Uses `closure_tree` gem for tree navigation
- Stores `hierarchical_position` for ordering

**Standards**
- `Standard`: Educational standards that can be attached to documents/materials

### Core Domain Specifications
- Lesson specifications: see `docs/core/lesson-metadata-specs.md` for the full lesson-metadata specification.

### Service Objects Pattern

Services are located in `app/services/` and follow these patterns:

**Import Services**
- `ImportService`: Base class for importing content
- `StandardsImportService`: Imports educational standards
- All import services typically inherit from base `ImportService`

**Document Processing**
- `BundleGenerator`: Creates bundled PDFs of multiple documents
- `EmbedEquations`: Processes mathematical equations
- Located in `lib/document_exporter/` and `lib/document_renderer/`

**Google Drive Integration**
- `Google::ScriptService`: Interacts with Google Apps Script
- Uses `lt-google-api` and `google-apis-drive_v3` gems

### Background Job Architecture

Jobs are in `app/jobs/` and use Resque with ActiveJob:

**Document Processing Jobs**
- `DocumentParseJob`: Parses imported Google Docs
- `DocumentGenerateJob`: Main job that orchestrates generation
- `DocumentGeneratePdfJob`: Generates PDF versions
- `DocumentGenerateGdocJob`: Generates Google Doc versions

**Material Processing Jobs**
- `MaterialParseJob`: Parses material content
- `MaterialGenerateJob`: Orchestrates material generation
- `MaterialGeneratePdfJob`: PDF generation
- `MaterialGenerateGdocJob`: Google Doc generation

All jobs inherit from `ApplicationJob` with retry logic via `activejob-retry`.

### Template System (lib/doc_template)

The `DocTemplate` module handles document templating and parsing:

- **Configuration**: Loads from `config/lcms.yml`
- **Tag Processing**: Custom markup tags in documents (e.g., `[section: name]`)
- **Tables**: Different table renderers (`DocTemplate::Tables::*`)
- **Context Types**: Multiple output formats (gdoc, PDF)
- **XPath Functions**: Custom functions for document parsing

Key regex pattern for tags: `FULL_TAG = /\[([^\]:\s]*)?\s*:?\s*([^\]]*?)?\]/mo`

### Value Objects and Presenters

- `app/value_objects/`: Immutable data structures
- `app/presenters/`: View-layer presentation logic
- `app/queries/`: Complex query objects (e.g., `AdminDocumentsQuery`, `AdminMaterialsQuery`)
- `app/forms/`: Form objects using `simple_form`

### Controller Structure

**Public Controllers**
- `DocumentsController`: Public document viewing and export
- `MaterialsController`: Public material previews (PDF, Google Doc)
- `WelcomeController`: Landing page and OAuth callbacks

**Admin Namespace** (`app/controllers/admin/`)
- Full CRUD for resources, documents, materials, users, standards
- Batch operations (bulk delete, reimport)
- Settings management

**API Namespace** (`app/controllers/api/`)
- RESTful JSON API for resources

### Frontend Architecture

**JavaScript** (`app/javascript/`)
- Entry point: `app/javascript/application.js`
- Built with esbuild, supports JSX for React components
- Uses jQuery, Lodash, Bootstrap for UI components

**Stylesheets** (`app/assets/stylesheets/`)
- Main: `application.bootstrap.scss`
- PDF-specific: `pdf.scss`, `pdf_plain.scss`
- Uses Bootstrap 5.3 with SCSS customizations

**Stimulus Controllers**: 7Hotwire Stimulus for JavaScript behavior

## Testing Strategy

- **Factories**: `spec/factories/` using FactoryBot
- **Request specs**: `spec/requests/`
- **Feature specs**: `spec/features/` using Capybara
- **Model specs**: `spec/models/`
- **Service specs**: `spec/services/`
- **Support files**: `spec/support/` for shared examples and helpers

Use `database_cleaner-active_record` for test database management.

## Git Commit Guidelines

**IMPORTANT**: All commit messages MUST be written in English.

### Commit Message Format

First line is the subject — a short summary starting with a capital letter. After a blank line, add details about changes using a bullet list.

### Git Commit Rules

- Always use `git commit -s` flag to add Signed-off-by line
- NEVER add Co-Authored-By line to commits

### Example

```
Add user authentication module

- Implement JWT token generation and validation
- Add login and logout endpoints
- Create middleware for protected routes
- Add password hashing with bcrypt
```

## Code Style Guidelines

**IMPORTANT**: All generated Ruby code MUST follow Rubocop rules configured in `.rubocop.yml`.

This project uses `rubocop-rails-omakase` style guide. Key rules to follow:

- **Double quotes for strings**: Always use `"string"` not `'string'`
- **Percent literal delimiters**: Use parentheses for `%w()`, `%i()`, `%W()`, `%I()`
- **Keyword alignment**: Align `end` with the keyword that opens the block (`if`, `def`, `class`, etc.)
- **New cops enabled**: All new Rubocop cops are enabled by default

Before committing code, run Rubocop to check for style violations:
```bash
docker compose run --rm rails bundle exec rubocop
docker compose run --rm rails bundle exec rubocop -a  # Auto-fix
```

## Important Patterns and Conventions

### Concerns
- `Filterable`: Adds scope-based filtering to models
- `Partable`: Multi-format content rendering (gdoc/pdf)
- Located in `app/models/concerns/`

### Metadata Storage
Documents and materials store curriculum metadata as JSONB:
```ruby
where_metadata(:subject, "math")  # Query JSONB metadata
```

### Queue Configuration
- Queue adapter: Resque (configured in `config/application.rb`)
- Access Resque web UI at `/queue` (requires authentication)

### Asset Paths
Custom asset paths configured for:
- FontAwesome webfonts: `node_modules/@fortawesome/fontawesome-free/webfonts`
- Bootstrap icons: `node_modules/bootstrap-icons/font/fonts`

## Configuration Files

- `config/lcms.yml`: DocTemplate configuration
- `.ruby-version`: Ruby 3.4.7
- `.node-version`: Node 22.12.0
- `config/routes.rb`: Routes with admin namespace and API
- `docker-compose.yml`: Docker services configuration
- `Dockerfile.dev`: Development Docker image
- `.env.docker`, `.env.development`: Environment variables

## Plugin System

The application supports a plugin architecture for extending functionality. Plugins are developed as git submodules in `lib/plugins/`.

### Key Concepts

- **Full access**: Plugins have direct access to all application models, services, and helpers
- **Single test suite**: Plugin tests run as part of the main application test suite
- **Minimal conflicts**: Plugin system uses separate files to minimize merge conflicts for forks

### Plugin Structure

```
lib/plugins/<plugin_name>/
  Gemfile                       # Plugin gem dependencies (auto-loaded by main Gemfile)
  lib/
    <plugin_name>.rb          # Main entry point with setup! method
    <plugin_name>/            # Service classes
  app/
    models/<plugin_name>/     # Namespaced models
    controllers/<plugin_name>/
    views/<plugin_name>/
  config/
    routes.rb                 # Plugin routes
  db/migrate/                 # Plugin migrations
  spec/                       # Plugin tests
```

### Working with Plugins

```bash
# Add a plugin as submodule
git submodule add https://github.com/org/plugin.git lib/plugins/plugin_name

# Clone with all plugins
git clone --recursive https://github.com/learningtapestry/lcms-core.git

# Update plugins after clone
git submodule update --init --recursive

# Run plugin tests
docker compose run --rm test bundle exec rspec lib/plugins/
```

### Documentation

See `docs/plugin-system.md` for complete plugin development guide.

## Dependencies Worth Noting

- **File Upload**: CarrierWave with AWS S3 support
- **Pagination**: WillPaginate with Bootstrap styling
- **Filtering**: Ransack for advanced search
- **Monitoring**: Airbrake for error tracking
- **Performance**: Bullet (dev) for N+1 query detection
- **Linting**: Rubocop with Rails Omakase style guide

---
> Source: [learningtapestry/lcms-core](https://github.com/learningtapestry/lcms-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
