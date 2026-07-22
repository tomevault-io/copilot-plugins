## repo-reconnoiter-rails

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RepoReconnoiter is an **API-only backend service** that analyzes GitHub trending repositories using AI to provide developers with context-aware recommendations. The system fetches trending repos every 20 minutes, uses AI to categorize and analyze them, and exposes a RESTful API for frontend clients to consume. The only UI is Mission Control (`/admin/jobs`) for admin job monitoring.

## Core Principles

1. **Cost Control**: Keep AI API costs under $10/month through automatic tracking, caching, and smart model selection
2. **Code Consistency**: All code follows strict organization standards with alphabetized methods and clear section headers
3. **Service Pattern**: Use "Doer" naming (no "Service" suffix) for all service classes
4. **Automatic Tracking**: The `OpenAi` service automatically tracks all API costs - never call OpenAI directly
5. **Prompt as Code**: AI prompts are versioned ERB templates in `app/prompts/`, not hardcoded strings
6. **Multi-Query Strategy**: Use 2-3 GitHub queries for comprehensive results when needed
7. **Observability**: Use strategic logging in background jobs and services for debugging - production logs stay clean and focused

## Logging Philosophy

**Development Environment:**

- Log level: `:debug` (verbose SQL queries, full stack traces, detailed debugging info)
- Broadcast logger: Writes to both STDOUT and `log/development.log` for `bin/dev` visibility
- Framework logging: Enabled (SQL, ActiveRecord, ActionCable, etc.)
- Custom logger statements: Encouraged in background jobs to track async operations

**Production Environment:**

- Log level: `:info` (request/response, errors, warnings only)
- Framework logging: Minimal (no SQL queries, concise output)
- Custom logger statements: Use sparingly - focus on business-critical events and errors
- Exception tracking: Errors automatically logged by Rails + Sentry

**Philosophy**: Strategic logging in background jobs and async operations aids debugging without cluttering logs. Development broadcasts to console for real-time monitoring via `bin/dev`. Production logs focus on business events, errors, and warnings only.

## Tech Stack

- **Framework**: Rails 8.1 (API-only mode, no frontend)
- **Database**: PostgreSQL 17 (production: Render managed)
- **Background Jobs**: Solid Queue (database-backed, no Redis needed)
- **Job Scheduling**: Solid Queue recurring tasks (see `config/recurring.yml`)
- **AI Provider**: OpenAI (gpt-5-mini for categorization, gpt-5 for comparisons and deep analysis)
- **Authentication**:
  - **Two-Layer API Auth**:
    - Layer 1 (App-to-App): API Keys via `Authorization: Bearer <API_KEY>` header - required for ALL endpoints
    - Layer 2 (User-Specific): JWT tokens via `X-User-Token` header - for user-specific actions
    - Generate API key: `bin/rails api_keys:generate NAME="Insomnia Testing"` (see `lib/tasks/api_keys.rake`)
    - Exchange GitHub OAuth token for JWT: `POST /api/v1/auth/exchange`
  - **Mission Control**: Devise + OmniAuth GitHub (invite-only whitelist)
  - **Session Exchange**: JWT → Rails session for Mission Control access via `/session_exchange?token=JWT`
- **Rate Limiting**: Rack::Attack (25/day per user, 5/day per IP)
- **Deployment**: Render.com (Starter plan - $14/month)
- **Hosting**:
  - Production API: <https://api.reporeconnoiter.com/v1/> (subdomain routing)
  - Development API: <http://localhost:3001/api/v1/> (path-based routing)
  - Mission Control: `/admin/jobs` (admin-only UI for job monitoring)
  - PostgreSQL 17 database (1GB storage, 97 connections)
  - Web Service (512MB RAM, always-on, shell access)
- **Ruby Version**: 3.4.7

## Development Commands

### Setup

```bash
bin/setup                 # Initial setup: install dependencies, setup database
bin/dev                   # Start development server (runs Puma + Solid Queue + Tailwind watcher)
```

### Database

```bash
bin/rails db:create       # Create database
bin/rails db:migrate      # Run migrations
bin/rails db:seed         # Seed database (populates initial categories)
bin/rails db:reset        # Drop, create, migrate, and seed database
```

### Testing

```bash
bin/rails test            # Run all tests (290 tests, API + backend only)
bin/rails test test/models/repository_test.rb  # Run specific test file
```

### Linting & Security

```bash
bin/rails ci:all          # Run all CI checks (security, lint, tests) - mirrors GitHub Actions
bin/rails ci:security     # Run security scans only (Brakeman, Bundler Audit, Importmap)
bin/rails ci:lint         # Run RuboCop linter only
bin/rails ci:test         # Run all tests only (unit + system)

# Individual commands (if needed)
bin/rubocop               # Run RuboCop linter
bin/brakeman              # Run security vulnerability scanner
bin/bundler-audit         # Check for vulnerable gem versions
```

### Background Jobs

```bash
bin/rails solid_queue:start    # Start Solid Queue worker
bin/rails solid_queue:stop     # Stop Solid Queue worker
```

### API Keys

```bash
bin/rails api_keys:generate NAME="Insomnia Testing"              # Generate new API key
bin/rails api_keys:generate NAME="Production" EMAIL="user@example.com"  # Generate key for specific user
bin/rails api_keys:list                                          # List all API keys
bin/rails api_keys:revoke ID=123                                 # Revoke an API key
bin/rails api_keys:stats                                         # Show usage statistics
bin/rails api_keys:cleanup                                       # Delete old revoked keys (90+ days)
```

### WebSocket Testing (Insomnia)

**ActionCable WebSocket URL:**
- Production: `wss://api.reporeconnoiter.com/cable`
- Development: `ws://localhost:3001/cable`

**Required Headers (Insomnia WebSocket request):**
- `Origin: https://reporeconnoiter.com` (bypasses browser-only CSRF protection)
- `Authorization: Bearer <API_KEY>` (if channel requires authentication)

**Note:** Origin checking only prevents browser-based attacks. Non-browser clients (Insomnia, curl) can set any Origin header. Real security comes from API key + user token authentication, not Origin validation.

### Production (Render.com)

```bash
# Access via Render Dashboard → Shell tab
bin/rails console         # Open Rails console on production
bin/rails db:migrate      # Run migrations (done automatically on deploy)

# Deploy via Git push (auto-deploys on push to main)
git push origin main      # Triggers automatic deployment

# Monitoring
# View logs: Render Dashboard → Logs tab
# View jobs: https://reporeconnoiter.com/admin/jobs (admin only)
```

## Code Organization Standards

All services and models MUST follow this consistent structure:

### Service/Model Organization

```ruby
class ExampleService
  #--------------------------------------
  # CUSTOM EXCEPTIONS
  #--------------------------------------

  class CustomError < StandardError; end
  class AnotherError < StandardError; end

  #--------------------------------------
  # CONSTANTS
  #--------------------------------------

  DEFAULT_LIMIT = 10
  MAX_RETRIES = 3

  attr_reader :query, :options

  #--------------------------------------
  # PUBLIC INSTANCE METHODS
  #--------------------------------------

  def initialize
    # initialization logic
  end

  def method_a
    # Methods alphabetized within this section
  end

  def method_b
    # ...
  end

  #--------------------------------------
  # CLASS METHODS
  #--------------------------------------

  class << self
    # Use 'class << self' - NOT 'def self.method_name'

    def class_method_a
      # Methods alphabetized within this section
    end

    def class_method_b
      # ...
    end
  end

  private

  #--------------------------------------
  # PRIVATE METHODS
  #--------------------------------------

  def private_method_a
    # Methods alphabetized within this section
  end

  def private_method_b
    # ...
  end
end
```

### Key Rules

1. **Section Order**:
   - Custom exceptions (if any)
   - Constants (if any)
   - attr_reader/accessor/writer
   - Public instance methods
   - Class methods
   - Private methods
2. **Section Headers**: Use `#--------------------------------------` separators with section names for:
   - CUSTOM EXCEPTIONS (if defined)
   - CONSTANTS (if defined)
   - PUBLIC INSTANCE METHODS
   - CLASS METHODS
   - PRIVATE METHODS
3. **Class Methods**: ALWAYS use `class << self`, NEVER use `def self.method_name`
4. **Alphabetization**: Methods MUST be alphabetized within each section (except `initialize` which comes first)
5. **Models**: Same rules apply - ASSOCIATIONS, VALIDATIONS, CALLBACKS, SCOPES, then methods
6. **RuboCop Style**: Follow RuboCop conventions (run `bin/rails ci:lint` before committing)
   - Array brackets: `[ "item" ]` not `["item"]` (space inside brackets)
   - CI will fail on style violations

## Service Naming Convention ("Doer" Pattern)

Services use action-oriented names WITHOUT "Service" suffix:

- ✅ `Prompter` (renders AI prompts)
- ✅ `UserQueryParser` (parses user queries)
- ✅ `RepositoryAnalyzer` (analyzes repositories)
- ✅ `Github` (GitHub API wrapper)
- ✅ `OpenAi` (OpenAI API wrapper)
- ❌ ~~`PromptService`~~ (too verbose)
- ❌ ~~`QueryParserService`~~ (too verbose)

## Architecture Overview

### Frontend/Backend Separation

This is an **API-only backend service**. All user-facing features are consumed via the REST API (`/api/v1/*`). The only exception is Mission Control, which provides an admin UI for job monitoring.

**Frontend Integration:**
- Frontend clients (e.g., Next.js) authenticate users via GitHub OAuth
- Exchange GitHub token for JWT via `/api/v1/auth/exchange`
- Use JWT for all subsequent API requests (`X-User-Token` header)

**Mission Control Access:**
- Admin users can access `/admin/jobs` for job monitoring
- Accessed via session exchange: `/session_exchange?token=JWT&redirect=/admin/jobs`
- Requires admin role (defined by `ALLOWED_ADMIN_GITHUB_IDS` env var)
- Session exchange converts JWT → Rails session for Mission Control UI

### Data Flow

1. **GitHub API Sync**: Solid Queue recurring job fetches trending repos from GitHub API
2. **Tier 1 Processing (Cheap)**: gpt-5-mini categorizes repos using metadata + description
3. **Tier 2 Processing (Expensive)**: gpt-5 performs deep analysis on-demand (README + issues)
4. **Tier 3 Processing (Comparison)**: Multi-query GitHub search, merge/dedupe, AI-powered comparison with real-time progress tracking
5. **Real-Time Progress Updates**: ActionCable broadcasts comparison creation progress to API clients via Solid Cable
   - **WebSocket URL**: `wss://api.reporeconnoiter.com/cable?token=<JWT>` (JWT required in URL)
   - **Ownership Verification**: Users can only subscribe to their own session_ids
   - **Channels**: ComparisonStatusChannel, ComparisonProgressChannel, AnalysisProgressChannel
6. **Caching Strategy**: Aggressive caching to minimize AI API costs - repos only re-analyzed if README changes or significant activity detected

### Core Services

#### OpenAi Service (`app/services/open_ai.rb`)

Transparent wrapper for OpenAI API that automatically tracks costs and enforces model whitelisting.

**Key Features:**

- **Model Whitelisting**: Only allows `gpt-5-mini` and `gpt-5` with explicit pricing
- **Automatic Cost Tracking**: Every API call updates `ai_costs` table with daily rollup
- **Transparent API**: Returns same response object as `OpenAI::Client`
- **Usage Tracking**: Logs model, tokens, and cost for every request

```ruby
# Always use OpenAi service instead of calling OpenAI directly
ai = OpenAi.new
response = ai.chat(
  messages: [
    { role: "system", content: "You are a helpful assistant" },
    { role: "user", content: "Hello!" }
  ],
  model: "gpt-5-mini",
  temperature: 0.3,
  track_as: "description_of_what_this_does"  # Optional: helps with debugging
)

# Response is standard OpenAI::Client response
content = response.choices[0].message.content
tokens = response.usage.prompt_tokens
```

**Pricing (as of 2025):**

- `gpt-4o-mini`: $0.150/1M input, $0.600/1M output
- `gpt-4o`: $2.50/1M input, $10.00/1M output

#### Prompter Service (`app/services/prompter.rb`)

Renders AI prompt templates from `app/prompts/` directory using ERB.

**Key Features:**

- **Template Rendering**: Renders `.erb` files with variable interpolation
- **Prompt Injection Prevention**: `sanitize_user_input()` method prevents attacks
- **Convention**: System prompts end in `_system.erb`, user prompts in other names

```ruby
# Render a system prompt (no variables)
system_prompt = Prompter.render("user_query_parser_system")

# Render with variables
user_prompt = Prompter.render("repository_analyzer_build",
  repository: repo,
  available_categories: categories
)

# Sanitize user input to prevent prompt injection
safe_query = Prompter.sanitize_user_input(user_input)
```

**Prompt Directory Structure:**

```
app/prompts/
  ├── README.md                              # Documentation
  ├── user_query_parser_system.erb           # System prompt for query parsing
  ├── repository_analyzer_system.erb         # System prompt for repo analysis
  └── repository_analyzer_build.erb          # User prompt with variables
```

**Creating New Prompts:**

```ruby
# Generate a new system prompt
Prompter.create("my_new_prompt", system: true)
# Creates: app/prompts/my_new_prompt_system.erb

# Generate a regular prompt
Prompter.create("my_prompt")
# Creates: app/prompts/my_prompt.erb
```

#### UserQueryParser Service (`app/services/user_query_parser.rb`)

Parses natural language queries into structured GitHub search parameters.

**Key Features:**

- **Multi-Query Support**: Can return 2-3 GitHub queries for comprehensive coverage
- **JSON Response**: Returns structured data with validation
- **Query Strategy**: Indicates "single" or "multi" query approach

```ruby
parser = UserQueryParser.new
result = parser.parse("I need a Rails background job library")

result[:github_queries]    # ["background job rails stars:>100", "sidekiq rails stars:>100"]
result[:query_strategy]    # "multi"
result[:tech_stack]        # "Rails, Ruby"
result[:problem_domain]    # "Background Job Processing"
result[:constraints]       # ["production ready", "retry logic"]
result[:valid]             # true
```

#### RepositoryAnalyzer Service (`app/services/repository_analyzer.rb`)

Analyzes and categorizes repositories using AI (formerly `RepositoryCategorizationService`).

**Methods:**

- `analyze(repository)` - Tier 1 analysis using metadata + description
- `deep_dive_analysis(repository)` - Tier 2 analysis using README + issues (not yet implemented)

```ruby
analyzer = RepositoryAnalyzer.new
result = analyzer.analyze(repo)

result[:categories]      # [{ category_id: 1, confidence: 0.95, reasoning: "..." }]
result[:summary]         # "Modern background job processor..."
result[:use_cases]       # ["Email sending", "Report generation"]
result[:input_tokens]    # 150
result[:output_tokens]   # 300
```

#### Github Service (`app/services/github.rb`)

Wrapper for GitHub API using Octokit gem.

```ruby
# Search repositories
results = Github.search("background job rails stars:>100", per_page: 30)

# Search trending repos
trending = Github.search_trending(days_ago: 7, language: "ruby", min_stars: 10)

# Instance methods also available
gh = Github.new
results = gh.search("query")
authenticated = gh.authenticated?
```

### Database Schema

**Core Tables:**

- **repositories**: GitHub repo data, cached README content, metadata
- **analyses**: Versioned AI-generated insights (Tier 1/Tier 2) with token/cost tracking
- **categories**: Categorization taxonomy (Problem Domain, Architecture Pattern, Maturity Level)
- **comparisons**: User queries with AI-generated repo comparisons (Tier 3)
- **users**: Authenticated users via GitHub OAuth (email, github_id, github_username, etc.)
- **whitelisted_users**: Invite-only access control (github_id, reason, added_by)

**Join Tables:**

- **repository_categories**: Many-to-many with confidence scores, assignment method (ai/manual/github_topics)
- **comparison_repositories**: Links comparisons to repos with rank and score
- **comparison_categories**: Links comparisons to inferred categories

**Processing:**

- **queued_analyses**: Queue for batch Tier 1/Tier 2 analysis (priority, retry logic, scheduling)
- **ai_costs**: Daily rollup of AI API spending by model and user (auto-updated by OpenAi service)

### Comparison Search

**Comprehensive Multi-Field Search with Fuzzy Matching:**

The `Comparison.search(search_term, fuzzy: true)` scope provides intelligent search with synonym expansion and fuzzy matching:

**Search Fields:**

- **user_query**: Original search query entered by user
- **tech_stack**: Parsed technologies (e.g., "Rails, Ruby", "Python")
- **problem_domain**: Identified problem area (e.g., "Background Job Processing")
- **categories**: Associated category names via join (e.g., "Ruby", "Authentication")

**Key Features:**

1. **Synonym Expansion** (`SearchSynonymExpander`):
   - "auth" → ["auth", "authentication", "authorize", "authorization"]
   - "job" → ["job", "jobs", "queue", "worker"]
   - "node" → ["node", "nodejs", "javascript", "js"]
   - 50+ synonym mappings for common tech terms

2. **Fuzzy Matching** (PostgreSQL `pg_trgm` WORD_SIMILARITY):
   - Handles plural/singular variations ("job" matches "jobs")
   - Threshold 0.45 balances recall vs precision
   - Catches partial matches ("scien" matches "Scientific Computing")
   - Prevents false positives (floating point precision handling)

3. **Search Modes:**
   - `fuzzy: true` (default) - Uses WORD_SIMILARITY for intelligent matching
   - `fuzzy: false` - Falls back to ILIKE for exact substring matching

4. **Performance:**
   - Efficient EXISTS subquery for category matching
   - SQL injection protection via `sanitize_sql_like`
   - Case-insensitive by default
   - Returns all comparisons for blank/nil search terms

**Usage Example:**

```ruby
# Fuzzy search (default) - handles synonyms and variations
results = Comparison.search("jobs")
# Matches:
# - "Rails background job library" (fuzzy: job ≈ jobs)
# - "Background worker processing" (synonym: worker → job)
# - Comparisons with "Background Job Processing" category

# Exact search - strict substring matching
results = Comparison.search("exact term", fuzzy: false)

# Via SearchComparisonsPresenter (used in UI)
presenter = SearchComparisonsPresenter.new(params)
comparisons = presenter.comparisons # Automatically applies fuzzy search
```

**Adding New Synonyms:**

Edit `app/services/search_synonym_expander.rb`:

```ruby
SYNONYMS = {
  "new_term" => [ "new_term", "synonym1", "synonym2" ],
  # ...
}
```

**Testing & Validation:**

```bash
bin/rails search:validate           # Test 21+ search scenarios (fuzzy + synonym)
bin/rails search:test["ruby"]       # Test specific search with match details
bin/rails search:coverage           # Verify field population (should be 100%)
bin/rails search:benchmark          # Performance testing (100 iterations)
```

**Performance Optimization (Future):**

When search becomes slow (10k+ comparisons), add GIN trigram indexes:

```ruby
add_index :comparisons, :tech_stack, using: :gin, opclass: :gin_trgm_ops
add_index :comparisons, :problem_domain, using: :gin, opclass: :gin_trgm_ops
```

Currently not needed - WORD_SIMILARITY is fast enough for small datasets.

### Authentication & Authorization

**Two-Layer API Authentication:**

All API endpoints (`/api/v1/*`) require TWO layers of authentication:

1. **Layer 1 - API Key (App-to-App Authentication)**:
   - Header: `Authorization: Bearer <API_KEY>`
   - Required for ALL API endpoints (enforced by `BaseController#authenticate_api_key!`)
   - Generate: `bin/rails api_keys:generate NAME="Insomnia Testing"`
   - Keys are BCrypt-hashed, raw key shown only once
   - Manage with rake tasks: `api_keys:list`, `api_keys:revoke`, `api_keys:stats` (see `lib/tasks/api_keys.rake`)

2. **Layer 2 - User Token (User-Specific Authentication)**:
   - Header: `X-User-Token: <JWT>`
   - Required for user-specific endpoints (e.g., creating comparisons)
   - Obtain via `POST /api/v1/auth/exchange` (exchange GitHub OAuth token for JWT)
   - JWT valid for 24 hours (configurable in `JsonWebToken::EXPIRATION_TIME`)
   - Create manually in console: `token = JsonWebToken.encode(user_id: user.id); puts token`

**Invite-Only Whitelist System:**

- Users authenticate via GitHub OAuth (Devise + OmniAuth)
- `WhitelistedUser` model controls who can access the app
- `User.from_omniauth` checks whitelist before allowing sign in
- Raises error if GitHub user not in whitelist (handled gracefully with redirect)

**Access Control:**

- Unauthenticated users: Can view homepage and existing comparisons (read-only)
- Authenticated users: Can create comparisons (rate limited to 25/day)
- Admin users: Unrestricted access + Mission Control dashboard access

**Rate Limiting (Rack::Attack):**

- 25 comparisons per day per authenticated user
- 5 POST requests per day per IP address (defense-in-depth security backstop)
- 10 OAuth login attempts per 5 minutes per IP
- Note: Only authenticated users can create comparisons (Devise blocks anonymous access)
- Throttling prevents abuse and controls AI costs

**Admin Access:**

- Mission Control Jobs dashboard: `/admin/jobs` (restricted to admins)
- Admin stats dashboard: `/admin/stats` (restricted to admins)
- Admin status determined by `ALLOWED_ADMIN_GITHUB_IDS` env var (GitHub IDs)
- Helper method: `current_user.admin?` for view-level checks
- Fail-closed security: Empty admin IDs means no admins (denies all access)

### Security Features

**Prompt Injection Prevention (OWASP LLM01:2025):**

- Multi-layered defense in `Prompter.sanitize_user_input`
- Context-aware filters (15+ patterns for credential extraction, system info leaks)
- System prompt security constraints in all AI prompts
- Output validation with suspicious pattern monitoring
- Applied to all user inputs before AI processing

**Content Security Policy (CSP):**

- Strict CSP enforced via `config/initializers/content_security_policy.rb`
- Nonce-based inline script/style protection (Turbo + Tailwind compatible)
- Microsoft Clarity whitelisted (CSP-friendly analytics)
- Blocks eval(), inline scripts, and untrusted sources
- Enforcing mode enabled (not report-only)

**HTTP Security Headers:**

- X-Frame-Options: DENY (clickjacking prevention)
- X-Content-Type-Options: nosniff (MIME sniffing prevention)
- X-XSS-Protection: 1; mode=block (legacy XSS filter)
- Referrer-Policy: strict-origin-when-cross-origin
- Permissions-Policy: Disables geolocation, camera, microphone, payment APIs
- HSTS (production only): max-age=31536000; includeSubDomains
- Force SSL enabled in production with secure cookies

**Security Scanning:**

- Brakeman: Static security analysis (0 warnings)
- Bundler Audit: Vulnerable gem detection (clean)
- All credentials encrypted in `config/credentials.yml.enc`
- Secrets never committed to git (`.gitignore` enforced)

**Input Validation:**

- User queries: 500 character max, whitespace prevention
- Model-level validations with custom validators
- Controller-level guard clauses with early returns
- Flash messages for validation errors

### Cost Optimization Strategy

The app implements several strategies to keep AI API costs under $10/month:

1. **Automatic Cost Tracking**: `OpenAi` service automatically tracks all API calls to `ai_costs` table with daily rollup
2. **Model Whitelisting**: Only allow pre-approved models with known pricing to prevent cost surprises
3. **Selective Processing**: Only analyze repos that pass metadata filters (stars > 100, active within 30 days, relevant language)
4. **Tiered AI Models**: Use cheap models (gpt-5-mini ~$0.001/repo) for categorization, expensive models only for deep dives
5. **Aggressive Caching**: Track `readme_sha` to detect changes; don't re-analyze unless content changed or 7+ days passed
6. **Batch Processing**: Queue repos during the day, process in nightly batches (limit 50/day)
7. **Multi-Query Strategy**: Use 2-3 GitHub queries to get comprehensive results, reducing need for expensive AI filtering
8. **Query Caching with pg_trgm**: Fuzzy query matching (0.8 threshold) prevents duplicate AI comparisons (~99% accuracy)
9. **Hard Limits**: Max 15 repos per comparison (prevents runaway costs)
10. **Cost Transparency**: Display cost estimate on homepage ($0.05 per search)

### Key Model Methods Pattern

Models should implement smart caching logic:

```ruby
class Repository < ApplicationRecord
  def needs_analysis?
    return true if last_analyzed_at.nil?
    return true if readme_changed?
    return true if last_analyzed_at < 7.days.ago
    return true if stargazers_count > last_analysis.stargazers_at_analysis * 1.5
    false
  end

  def readme_changed?
    current_sha != readme_sha
  end
end
```

### Background Job Pattern

Jobs should use services that automatically track costs:

```ruby
class AnalyzeRepositoryJob < ApplicationJob
  queue_as :default

  def perform(repository_id)
    repo = Repository.find(repository_id)
    return unless repo.needs_analysis?

    # Use RepositoryAnalyzer service which uses OpenAi internally
    analyzer = RepositoryAnalyzer.new
    result = analyzer.analyze(repo)

    # OpenAi service already tracked costs to ai_costs table
    # Just store the analysis results
    repo.analyses.create!(
      analysis_type: "tier1_categorization",
      summary: result[:summary],
      use_cases: result[:use_cases],
      input_tokens: result[:input_tokens],
      output_tokens: result[:output_tokens],
      model_used: "gpt-5-mini",
      is_current: true
    )

    # Create category associations
    result[:categories].each do |cat|
      repo.repository_categories.create!(
        category_id: cat[:category_id],
        confidence_score: cat[:confidence],
        assigned_by: "ai"
      )
    end
  end
end
```

**Important Rules:**

1. ALWAYS use `OpenAi` service, NEVER call `OpenAI::Client` directly
2. Cost tracking is automatic - no need to manually calculate or save costs
3. The `OpenAi#chat` method returns standard OpenAI response object
4. Use `track_as:` parameter to label what the API call is for (helps debugging)

## Rails 8 Specific Features

- **Solid Queue**: Database-backed job processing (no Redis required). Job queues configured in `config/queue.yml`
- **Solid Cache**: Database-backed caching configured in `config/cache.yml`
- **Solid Cable**: Database-backed Action Cable for WebSocket connections
  - **IMPORTANT**: Uses `solid_cable` adapter (not `async`) for cross-process broadcasting
  - Allows background jobs to broadcast real-time updates to connected browsers
  - Development and production both use `solid_cable` for consistency
- **Multi-Database Setup**: Single PostgreSQL database with separate schema files for primary, cache, queue, and cable
  - `db/schema.rb` - Primary application data
  - `db/cache_schema.rb` - Solid Cache tables
  - `db/queue_schema.rb` - Solid Queue tables
  - `db/cable_schema.rb` - Solid Cable tables
  - All connections share same `DATABASE_URL` in production (Render requirement)

## Configuration Files & Directories

**Configuration:**

- `config/recurring.yml`: Solid Queue recurring task definitions for scheduled jobs
- `config/queue.yml`: Solid Queue configuration
- `config/cable.yml`: Solid Cable configuration (uses `solid_cable` adapter for cross-process broadcasting)
- `config/initializers/devise.rb`: Devise authentication configuration (GitHub OAuth)
- `config/initializers/rack_attack.rb`: Rate limiting configuration
- `config/initializers/content_security_policy.rb`: CSP configuration with Microsoft Clarity
- `config/initializers/mission_control.rb`: Mission Control Jobs authentication
- `.env.example`: Environment variable documentation (copy to `.env` for development)

**AI Prompts:**

- `app/prompts/`: ERB templates for AI prompts (rendered by Prompter service)
- `app/prompts/README.md`: Documentation for prompt templates

**Services:**

- `app/services/`: All service classes following "Doer" naming pattern
  - `open_ai.rb`: OpenAI API wrapper with automatic cost tracking
  - `prompter.rb`: AI prompt template renderer
  - `user_query_parser.rb`: Natural language query parser
  - `repository_analyzer.rb`: Repository AI analysis
  - `github.rb`: GitHub API wrapper
  - `comparison_creator.rb`: Orchestrates comparison creation with progress broadcasting
  - `comparison_progress_broadcaster.rb`: ActionCable broadcasting for real-time progress updates
  - `repository_fetcher.rb`: Fetches and prepares repositories from GitHub

**Rake Tasks:**

- `lib/tasks/ci.rake`: CI/CD tasks that mirror GitHub Actions
  - `ci:all`: Run all CI checks (security, lint, tests)
  - `ci:security`: Run security scans (Brakeman, Bundler Audit, Importmap)
  - `ci:lint`: Run RuboCop linter
  - `ci:test`: Run all tests (unit + system)
- `lib/tasks/api_keys.rake`: API key management (for testing with Insomnia/Postman)
  - `api_keys:generate` (NAME='...' [EMAIL='...']): Generate new API key (optionally for specific user)
  - `api_keys:list`: List all API keys with usage stats
  - `api_keys:revoke` (ID=123): Revoke an API key
  - `api_keys:stats`: Show API key usage statistics
  - `api_keys:cleanup`: Delete old revoked keys (90+ days)
- `lib/tasks/analyze.rake`: Repository analysis tasks
  - `analyze:basic` (REPO='owner/name'): Run basic analysis on a single repo (Tier 1, gpt-5-mini)
  - `analyze:deep` (REPO='owner/name'): Run deep analysis on a single repo (Tier 2, gpt-5, cheaper with 272K context)
- `lib/tasks/comparisons.rake`: Comparison management tasks
  - `comparisons:create` (QUERY='...'): Create a new comparison (parse → fetch → analyze → compare)
  - `comparisons:search["query"]`: Search for existing comparisons matching a query
- `lib/tasks/query.rake`: Query parser testing
  - `query:parse["query"]`: Test query parser with a natural language query
- `lib/tasks/github.rake`: GitHub API exploration and manual sync
  - `github:explore`: Explore GitHub API and display available data
  - `github:trending[days,language,min_stars]`: Search trending repos with custom parameters
  - `github:search[query]`: Search GitHub repositories with any query string
  - `github:sync[days,min_stars,per_page]`: Sync trending repositories to database
- `lib/tasks/whitelist.rake`: User whitelist management (invite-only system)
  - `whitelist:add[github_id,github_username,email,notes]`: Add user to whitelist
  - `whitelist:list`: List all whitelisted users
  - `whitelist:remove[github_username]`: Remove user from whitelist
- `lib/tasks/db.rake`: Database management
  - `db:sync_from_production`: Pull production database and load it locally (dev only)
- `lib/tasks/category_seeds.rake`: Category seed file management
  - `categories:dump_seeds`: Dump current categories to seeds file

**Documentation:**

- `README.md`: Project overview and getting started guide (root level)
- `TODO.md`: Current development status and next steps (root level)
- `CLAUDE.md`: This file - coding standards and architecture guide (root level)
- `docs/OVERVIEW.md`: Detailed project concept, database schema, and cost optimization strategies
- `docs/PLAN.md`: Phased build order from foundation to deployment
- `docs/SCHEMA.md`: Database schema documentation
- `docs/GITHUB_SEARCH_RESEARCH.md`: GitHub API search research and golden queries
- `docs/SECURITY_REVIEW.md`: Security audit summary and compliance documentation
- `docs/RENDER_DEPLOYMENT.md`: Complete Render.com deployment guide (step-by-step)

## Production Deployment

**Status**: ✅ Live in production at <https://reporeconnoiter.com>

**Deployment Process:**

1. Push to `main` branch on GitHub
2. Render auto-deploys (build command runs: `bundle install && rails assets:precompile && rails db:migrate`)
3. Web service restarts with new code
4. Monitor via Render Dashboard → Logs tab

**Initial Setup (One-Time):**

- See `docs/RENDER_DEPLOYMENT.md` for complete step-by-step guide
- PostgreSQL database created and schemas loaded
- Environment variables configured (DATABASE_URL, SECRET_KEY_BASE, RAILS_MASTER_KEY, GitHub OAuth, OpenAI)
- SSL auto-provisioned via Let's Encrypt
- Force SSL enabled with HSTS

**Post-Deployment Tasks:**

- Whitelist admin user via Rails console
- Test OAuth flow and comparison creation
- Verify security headers at <https://securityheaders.com/>
- Check Mission Control Jobs dashboard

**Monitoring:**

- Logs: Render Dashboard → Logs tab
- Jobs: <https://reporeconnoiter.com/admin/jobs> (admin only)
- Stats: <https://reporeconnoiter.com/admin/stats> (admin only)
- Analytics: Microsoft Clarity dashboard
- Costs: Check `ai_costs` table via Rails console

## Testing

**Test Suite:**

- 242 tests, 615 assertions (all passing)
- 229 unit tests + 13 API integration tests + system tests
- Run with: `bin/rails test` (runs in parallel)

**Test Coverage:**

- Security tests (OAuth whitelist, rate limiting, Mission Control access)
- Cost control tests (fuzzy cache matching with pg_trgm)
- Data integrity tests (repository deduplication)
- Model tests (Repository analysis logic, needs_analysis? behavior)
- Service tests (RepositoryFetcher, correct method names)
- Presenter tests (homepage stats, trending, categories)
- API tests (pagination, filtering, search, OpenAPI schema validation)
- System tests (homepage UI for authenticated/unauthenticated users)

**OpenAPI Schema Validation:**

- `committee` gem automatically validates API responses against `docs/openapi.yml`
- Every API test includes `assert_schema_conform` to prevent documentation drift
- Tests fail if response doesn't match OpenAPI specification
- Configuration in `test/test_helper.rb` (see Committee section)

**Test Philosophy:**

- Test custom logic, not framework features
- Focus on security, cost control, and data integrity
- Fail-closed security (empty admin IDs should error, not allow all)
- Use realistic fixture data instead of empty stubs
- Create dedicated fixtures (e.g., `:no_analyses`) instead of destroying data in tests
- Parallel test safety: never use `destroy_all` or other database-wide operations

## Environment Variables

**Required for Production:**

- `DATABASE_URL` - PostgreSQL connection string from Render
- `SECRET_KEY_BASE` - Rails encryption key (generate with `bin/rails secret`)
- `RAILS_MASTER_KEY` - Master key to decrypt credentials (from `config/master.key`)
- `GITHUB_CLIENT_ID` - GitHub OAuth App ID
- `GITHUB_CLIENT_SECRET` - GitHub OAuth App Secret
- `OPENAI_ACCESS_TOKEN` - OpenAI API key
- `ALLOWED_ADMIN_GITHUB_IDS` - Comma-separated GitHub IDs for admin access

**Optional:**

- `CLARITY_PROJECT_ID` - Microsoft Clarity analytics project ID
- `COMPARISON_SIMILARITY_THRESHOLD` - Query fuzzy matching threshold (default: 0.8)
- `COMPARISON_CACHE_DAYS` - Cache TTL in days (default: 7)
- `RAILS_LOG_LEVEL` - Logging verbosity (default: info)

See `.env.example` for complete documentation.

---
> Source: [jimmypocock/repo-reconnoiter-rails](https://github.com/jimmypocock/repo-reconnoiter-rails) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
