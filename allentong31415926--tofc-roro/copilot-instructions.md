## tofc-roro

> This is a Ruby on Rails 8 project built with Ruby 3.2.2. The application follows RESTful principles and uses PostgreSQL as the database.

# Code Style and Structure (applies to: **/*.rb)
- Write concise, idiomatic Ruby code with accurate examples.
- Follow Rails conventions and best practices.
- Follow Rails naming conventions:
  - Model names are singular (e.g., User, Post).
  - Controller names are plural (e.g., UsersController, PostsController).
- Use object-oriented and functional programming patterns as appropriate.
- Prefer iteration and modularization over code duplication.
- Use CamelCase for class and module names.
- Use descriptive variable and method names (e.g., user_signed_in?, calculate_total).
- Use snake_case for variables, methods, and file names (e.g., current_user, app/models/user.rb).
- Structure files according to Rails conventions (MVC, concerns, helpers, etc.).
- Use Ruby's expressive syntax (e.g., unless, ||=, &.)
- Prefer single quotes for strings unless interpolation is needed.
- Keep lines under 80 characters, per RuboCop defaults.
- Use 2-space indentation, no tabs.

# Routing and Controllers (applies to: app/controllers/*_controller.rb, config/routes.rb)
- Define routes in config/routes.rb using RESTful resources (e.g., resources :posts).
- Keep controller actions lean: limit to 1-2 method calls, delegate complex logic to service objects.
- Use Rails 8’s built-in authentication (e.g., `authenticate` DSL) where applicable.
- Always include before_action filters for authentication or authorization when needed.

# Models (applies to: app/models/*.rb)
- Use ActiveRecord for database interactions.
- Define associations (e.g., has_many, belongs_to) explicitly with foreign_key if non-standard.
- Add validations (e.g., validates :email, presence: true) for required fields.
- Avoid business logic in models; extract shared logic into Concerns and complex logic into Service Objects.

# Concerns (applies to: app/{models,controllers}/concerns/*.rb)
- Use Concerns to share reusable code between models or controllers:
  - Place model concerns in app/models/concerns/ (e.g., app/models/concerns/auditable.rb).
  - Place controller concerns in app/controllers/concerns/ (e.g., app/controllers/concerns/authenticable.rb).
  - Name concerns with a descriptive suffix (e.g., Auditable, Searchable).
  - Include concerns using `include` (e.g., `include Auditable` in a model).

# Service Objects (applies to: app/services/*_service.rb)
- Encapsulate complex business logic in service objects under app/services/.
- Service objects must:
  - Have a single public method named `.call`.
  - Accept arguments in the initializer.
  - Return a result object or raise an exception on failure.

# Jobs (applies to: app/jobs/*_job.rb)
- Use Sidekiq for background processing or time-consuming tasks under app/jobs/.
- Name jobs with a `_job` suffix (e.g., SendWelcomeEmailJob).
- Define a single `perform` method in each job.

# Workflows (applies to: app/workflows/*_workflow.rb)
- Define application workflows in app/workflows/ to handle multi-step processes.
- Name workflows with a `_workflow` suffix (e.g., UserOnboardingWorkflow).
- Structure workflows with a single `.call` method and private step methods.

# Views (applies to: app/views/**/*.erb)
- Use ERB (.erb) templates for views by default.
- Minimize Ruby logic in views; prefer helpers or partials.

# Performance Optimization
- Use database indexing effectively.
- Implement caching strategies (fragment caching, Russian Doll caching).
- Use eager loading to avoid N+1 queries.
- Optimize database queries using includes, joins, or select.

# Security (applies to: app/controllers/**/*.rb, config/initializers/*.rb)
- Implement proper authentication and authorization (e.g., Pundit).
- Use strong parameters in controllers.
- Protect against common web vulnerabilities (XSS, CSRF, SQL injection).
- Restrict API access to trusted clients (web and mobile apps):
  - Require an API key in every request via the "X-API-Key" header.
  - Store the API key securely in Rails credentials (rails credentials:edit).
  - For web clients, avoid hardcoding the API key in JavaScript; use environment variables.
  - For mobile clients, store the API key in secure storage (e.g., Keychain on iOS, Keystore on Android).
  - Verify the API key in a before_action filter in ApplicationController using secure_compare.
  - Optionally, check the Origin header for web requests (e.g., https://mywebapp.com).
  - Return 401 Unauthorized for invalid or missing API keys.
  - Mandate HTTPS for all endpoints to encrypt the API key in transit (set config.force_ssl = true in production).
  - Ensure frontend requests (web and mobile) use HTTPS URLs (e.g., https://myrailsbackend.com).

# Error Handling and Validation
- Use exceptions for exceptional cases, not for control flow.
- Implement proper error logging and user-friendly messages.
- Use ActiveModel validations in models.
- Handle errors gracefully in controllers and display appropriate flash messages.

# Testing (applies to: spec/**/*_spec.rb)
- Write comprehensive tests using RSpec under spec/.
- Follow TDD/BDD practices.
- Write unit tests for models, services, concerns, jobs, and workflows.
- Use factories (FactoryBot) for test data generation.

# CI/CD Workflows (applies to: .github/workflows/*.yml)
- Define GitHub Actions workflows in .github/workflows/.
- Use Ruby 3.2.2 and run `bundle exec rspec` and `rubocop`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/allenTong31415926) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
