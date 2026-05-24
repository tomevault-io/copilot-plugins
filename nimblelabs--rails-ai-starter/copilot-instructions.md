## rails-ai-starter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Tech Stack

- **Backend**: Rails 8.1.1 with PostgreSQL
- **Frontend**: Vue 3 with Vite, served via vite_rails
- **Styling**: Tailwind CSS 4.x
- **Authentication**: Devise
- **LLM Integration**: ruby_llm gem (~> 1.9.1) for OpenAI/Anthropic APIs
- **Email**: bootstrap-email, ahoy_email (tracking), mailkick (unsubscribe management)
- **Background Jobs**: Solid Queue (Rails 8 default)
- **Caching**: Solid Cache
- **Action Cable**: Solid Cable
- **Rich Text**: Action Text with Trix editor
- **File Storage**: Active Storage
- **URL Slugs**: FriendlyId

## Development Commands

### Setup
```bash
bin/setup              # Install dependencies, prepare DB, start server
bin/setup --skip-server # Setup without starting server
```

### Running the Application
```bash
bin/dev                # Start all services (Rails, Vite, Tailwind CSS)
bin/rails server       # Rails server only (port 3000)
bin/vite dev           # Vite dev server only
```

### Database
```bash
bin/rails db:prepare   # Create/migrate/seed database
bin/rails db:migrate   # Run pending migrations
bin/rails db:test:prepare # Prepare test database
```

### Testing
```bash
bin/rails test                    # Run all tests
bin/rails test:system             # Run system tests only
bin/rails test test/models/user_test.rb  # Run single test file
```

### Code Quality
```bash
bin/rubocop           # Run linter (omakase style)
bin/rubocop -a        # Auto-fix linter issues
bin/brakeman          # Security vulnerability scan
bin/importmap audit   # JavaScript security scan
bin/rails annotaterb:annotate # Update model schema annotations
```

## Architecture

### Dual Vue Applications

The project has **two separate Vue 3 SPAs**:

1. **User App** (`/app/*`)
   - Entrypoint: `app/javascript/entrypoints/application.js`
   - Root component: `app/javascript/app/App.vue`
   - Router: `app/javascript/app/router.js`
   - Mounted on: `#app` div in `app/views/static/app.html.erb`

2. **Admin App** (`/admin/*`)
   - Entrypoint: `app/javascript/entrypoints/admin.js`
   - Root component: `app/javascript/admin/AdminApp.vue`
   - Router: `app/javascript/admin/router.js`
   - Mounted on: `#app` div in `app/views/static/admin.html.erb`
   - Uses Trix editor for rich text

Both apps share a model object exposed as `window.starter` for Rails integration.

### Rails Routes Structure

- Static pages: Root `/`, About, Privacy, Terms
- Devise auth: `/sign-in`, `/register`, `/logout`
- Resources: `/articles`, `/contacts`, `/email-templates`
- API: `/api/v1/users/*` (JSON format)
- Vue apps: `/app/*` and `/admin/*` (catch-all routes to respective SPAs)

### Key Models

**User**
- Uses Devise for authentication
- FriendlyId slugs based on name
- Has `role` enum (user: 0, admin: 1)
- Mailkick subscriptions (`has_subscriptions`)
- After create: sends welcome email, subscribes to newsletter

**Article**
- FriendlyId slugs based on title
- Action Text rich content (`has_rich_text :content`)
- Active Storage featured image (`has_one_attached :featured_image`)
- Fields: title, description, author, category, published, featured, published_at

**Contact**
- FriendlyId slugs based on name
- Mailkick subscriptions
- Email validation with URI::MailTo::EMAIL_REGEXP
- Tracks Ahoy messages for email campaign tracking
- After create: subscribes to newsletter

**Chat/Message System**
- Uses ruby_llm gem: `acts_as_chat` and `acts_as_message`
- Messages track tokens (input/output), role, model_id
- ToolCall model for function calling support

### Email Architecture

**Template System**
- `EmailTemplate` model with `has_rich_text :content` for HTML emails
- `SendEmailTemplateJob` for bulk sending to contacts
- Bootstrap-email for responsive email styling
- Ahoy Email for open/click tracking
- Mailkick for unsubscribe management

**User Journey**
- Welcome email sent 2 seconds after user creation
- Auto-subscribe to "Newsletter" on signup
- Contacts track "first outreach email" with custom queries

### Environment Configuration

Required environment variables (use .env in development via dotenv-rails):
- `OPENAI_API_KEY` - for ruby_llm OpenAI integration
- `ANTHROPIC_API_KEY` - for ruby_llm Anthropic integration
- `DATABASE_URL` - PostgreSQL connection (auto-configured in database.yml)

### Deployment

- Docker support via Dockerfile
- Kamal deploy configuration in .kamal/ and config/deploy.yml
- Thruster for HTTP caching/compression
- CI via GitHub Actions: brakeman, importmap audit, rubocop, tests

## Important Patterns

### FriendlyId Usage
Most models use FriendlyId with `use: [:slugged, :finders]`. This means:
- URLs use slugs instead of IDs (e.g., `/articles/my-article-title`)
- Finders work with both slugs and IDs
- Models need a `slug` column (string, indexed)

### Ruby LLM Integration
Configure in `config/initializers/ruby_llm.rb`. Models using `acts_as_chat` and `acts_as_message` automatically get chat functionality with token tracking and tool calling support.

### Vite + Rails Integration
- Vite config base path: `/app/` (see vite.config.js)
- Custom elements: `ion-*` and `trix-*` tags whitelisted for Vue
- Asset paths use Vite helpers: `vite_javascript_tag`, `vite_client_tag`

### Schema Annotations
Models are annotated with schema info via annotaterb. Run `bin/rails annotaterb:annotate` after migrations to keep comments up to date.


## Feature Workflow

Pull features from the production yourdomainhere.com Features API to manage development workflow.

**Setup:**
1. Get your API token from: Production Admin → Users → Your User → Copy API Token
2. Add to `.env`: `STARTER_API_TOKEN=<your-token>`

**Commands:**
```bash
# Rake tasks
rake feature:list                    # List planned features from production
rake feature:show[slug]              # Show details for a specific feature
rake feature:start[slug]             # Create branch, update status to in_progress
rake feature:complete                # Complete current feature (detects from branch)
rake feature:current                 # Show current feature being worked on

# Shell wrappers (quick access)
bin/feature list                     # Alias for rake feature:list
bin/feature start <slug>             # Alias for rake feature:start
bin/feature complete                 # Alias for rake feature:complete
bin/feature current                  # Alias for rake feature:current
```

**Workflow:**
1. `rake feature:list` - See available features
2. `rake feature:start[slug]` - Creates `feature/<slug>` branch, marks as in_progress
3. Work on the feature, commit changes
4. `rake feature:complete` - Marks as completed, optionally creates PR

## API Token Authentication

Users have an `auth_token` column for stateless API authentication.

**ApplicationController Methods:**
- `authenticate_with_token` - Token-only auth via `x-api-token` header
- `authenticate_user_or_token` - Token OR Devise session auth
- Both set `@current_user` and work with `current_user` helper

**Usage in controllers:**
```ruby
before_action :authenticate_with_token      # Token only
before_action :authenticate_user_or_token   # Token or session
```

**User model:**
```ruby
user.auth_token                  # Get token
user.regenerate_auth_token       # Generate new token
User.find_by_auth_token(token)   # Find by token
```

### Final Points
- Write every prompt that you are given to a PROMPTS.md file so we can keep track of them over time.

---
> Source: [NimbleLabs/rails-ai-starter](https://github.com/NimbleLabs/rails-ai-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
