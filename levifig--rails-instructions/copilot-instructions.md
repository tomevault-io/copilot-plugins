## rails-rules

> Rails Development Rules - The Rails Way with AI Agents

# Rails Development Rules - The Rails Way with AI Agents
***Built for: Solo developer + AI agents, Linear MCP integration, maximum joy***

## Core Philosophy
You are building a Rails 8+ application following The Rails Way™. Code should be:
- Convention over configuration
- Database-first design
- Progressive enhancement with Hotwire
- Zero-build frontend approach
- Test-driven with Minitest
- Optimized for AI agent collaboration
- Lean, readable, and maintainable

## Rails 8+ Stack Preferences
- **Authentication**: Rails built-in (`rails generate authentication`)
- **Background Jobs**: Solid Queue (default Rails 8)
- **Caching**: Solid Cache (database-backed, Redis when needed)
- **WebSockets**: Action Cable with Solid Cable adapter
- **Database**: PostgreSQL with Active Record
- **Frontend**: Hotwire (Turbo + Stimulus) + TailwindCSS
- **Asset Pipeline**: Propshaft (simpler, no-build approach)
- **Testing**: Minitest with fixtures (no RSpec, no factories)
- **Rich Text**: Action Text for content editing
- **File Uploads**: Active Storage with direct uploads
- **Deployment**: Kamal with Docker
- **Code Quality**: StandardRB for linting/formatting
- **Development Tools**: Bullet gem for N+1 detection, Annotate gem for schema docs
- **Error Tracking**: Rails built-in error reporter

## Detailed Guides
This document provides core principles. For detailed implementation guidance, see:

- **[core.md](rails/core.md)** - Rails 8 conventions and patterns
- **[models.md](rails/models.md)** - Active Record patterns and best practices
- **[controllers.md](rails/controllers.md)** - Controller design and RESTful patterns
- **[services.md](rails/services.md)** - Service objects and business logic
- **[testing.md](rails/testing.md)** - Testing philosophy and Minitest patterns
- **[security.md](rails/security.md)** - Security best practices and authorization
- **[performance.md](rails/performance.md)** - Database optimization and caching
- **[api.md](rails/api.md)** - API design and versioning (when needed)
- **[importmaps.md](rails/importmaps.md)** - JavaScript without build steps
- **[hotwire.md](rails/hotwire.md)** - Turbo & Stimulus patterns
- **[views.md](rails/views.md)** - View helpers and rendering
- **[styling.md](rails/styling.md)** - TailwindCSS integration
- **[background-jobs.md](rails/background-jobs.md)** - Solid Queue configuration
- **[deployment.md](rails/deployment.md)** - Kamal deployment guide
- **[mobile.md](rails/mobile.md)** - Hotwire Native for mobile apps (optional)

## File Organization & Naming

### Consistent Directory Structure
```
app/
├── controllers/
│   ├── concerns/
│   └── application_controller.rb
├── models/
│   ├── concerns/
│   └── application_record.rb
├── views/
│   ├── layouts/
│   ├── shared/
│   └── [resource_name]/
├── services/
├── jobs/
├── channels/
├── mailers/
└── helpers/
```

### Naming Conventions for AI Agents
- **Classes**: PascalCase, descriptive (`UserRegistrationService`, `InvoicePaymentProcessor`)
- **Files**: snake_case matching class name (`user_registration_service.rb`)
- **Methods**: snake_case, verb-first for actions (`process_payment`, `calculate_total`)
- **Variables**: snake_case, noun-first (`current_user`, `payment_amount`)
- **Constants**: SCREAMING_SNAKE_CASE (`MAX_RETRY_ATTEMPTS`, `DEFAULT_CURRENCY`)

### AI Agent File Patterns
Always organize files predictably:
```
app/models/user.rb                    # Model: singular
app/controllers/users_controller.rb   # Controller: plural + _controller
app/services/user_registration.rb     # Service: domain + action
app/jobs/send_welcome_email_job.rb    # Job: action + _job
app/views/users/index.html.erb        # View: controller/action
```

### AI-Friendly Documentation
- Include purpose statement for AI comprehension
- Reference Linear ticket context (ID-123)
- Document key dependencies and return values
- Specify potential errors and exceptions
- Keep documentation close to code

## Linear Integration (Project Management)

### Overview
This section contains Linear-specific integration patterns. The same principles can be adapted for other project management tools by replacing Linear ticket formats and magic words with platform-specific equivalents.

### Commit Message Format
- Use Conventional Commits format with issue references
- Structure: `type(scope): description` followed by body and footer
- Include ticket reference in commit body or footer
- Example:
  ```
  feat: add magic link authentication

  Implements passwordless login flow
  Fixes ID-123
  ```

### Issue Linking
- Use semantic keywords to manage issue state through commits
- **Closing keywords**: `close`, `closes`, `closed`, `closing fix`, `fixes`, `fixed`, `fixing`, `resolve`, `resolves`, `resolved`, `resolving`, `complete`, `completes`, `completed`, `completing` (auto-close issues)
- **Reference keywords**: `ref`, `refs`, `references`, `part of`, `related to`, `contributes to`, `toward`, `towards` (link without closing)
- Support multiple issues: `Fixes ID-123, ID-456`

### Branch Strategy
- Follow GitHub Flow with descriptive branch names
- Format: `type/ticket-id/brief-description`
- Example: `feat/id-123/magic-link-login`
- Keep branches short-lived and focused

### Documentation Integration
- Reference tickets in code comments for complex logic
- Include ticket IDs in migration descriptions
- Link issues in test descriptions for context
- Document architectural decisions with ticket references

## Models & Active Record

### Model Design Principles
- Keep models focused on data integrity and business rules
- Use schema annotations (annotate gem) for documentation
- Order model contents consistently: constants, includes, associations, validations, callbacks, scopes, methods
- Implement database constraints to match validations
- Use concerns for shared behavior across models

### Model Best Practices
- Always index foreign keys and frequently queried columns
- Use counter caches for association counts
- Implement scopes for common query patterns
- Keep callbacks minimal - prefer service objects for complex logic
- Use `dependent:` options to maintain referential integrity

### Database Design
- Design normalized schemas by default
- Use appropriate PostgreSQL data types
- Implement database-level constraints
- Create partial indexes for performance
- Document complex queries and decisions

See **[models.md](rails/models.md)** for detailed Active Record patterns.

## Controllers

### Controller Principles
- Keep controllers thin and focused on HTTP concerns
- Use before_action filters for common setup
- Follow RESTful conventions strictly
- Handle multiple response formats (HTML, Turbo Stream, JSON)
- Implement proper error handling with appropriate status codes

### Controller Patterns
- Always use strong parameters for user input
- Prefer redirect after mutations over render
- Keep business logic in models or service objects
- Use concerns for shared controller behavior
- Implement resourceful routes whenever possible

See **[controllers.md](rails/controllers.md)** for detailed controller patterns.

## Service Objects

### When to Use Service Objects
- Complex business logic spanning multiple models
- External API integrations
- Multi-step processes with transactions
- Operations that don't naturally fit in a model
- Background job logic that needs testing

### Service Object Principles
- Keep services focused on a single operation
- Use clear, descriptive names
- Return meaningful results (success/failure)
- Make services easy to test in isolation
- Include proper error handling

See **[services.md](rails/services.md)** for service object patterns.

## Testing Philosophy

### The Rails Way of Testing
- Use fixtures over factories for simplicity and speed
- Write system tests for critical user flows
- Unit test models and services thoroughly
- Test controllers only for complex authorization
- Always test happy path and edge cases

### Testing Best Practices
- Keep tests fast and focused
- Use descriptive test names
- Test behavior, not implementation
- Mock external services appropriately
- Maintain high coverage without obsessing

See **[testing.md](rails/testing.md)** for comprehensive testing patterns.

## Background Jobs

### Job Design Principles
- Make all jobs idempotent and retryable
- Keep jobs small and focused
- Pass simple arguments (IDs, not objects)
- Design for eventual consistency
- Handle failures gracefully

### Solid Queue Configuration
- Use database-backed queuing for simplicity
- Configure workers based on priorities
- Set appropriate concurrency limits
- Monitor queue depth and latency
- Implement proper error handling

See **[background-jobs.md](rails/background-jobs.md)** for Solid Queue patterns.

## Mailers

### Mailer Best Practices
- Keep mailers simple and focused
- Use layouts for consistent email design
- Test email delivery in development
- Implement proper error handling
- Consider delivery performance

### Email Design
- Design for email client limitations
- Provide text alternatives
- Test across email clients
- Keep templates maintainable
- Handle bounces appropriately

## Performance & Optimization

### Query Optimization
- Avoid N+1 queries with proper includes
- Use database-level operations when possible
- Implement appropriate indexes
- Profile before optimizing
- Monitor performance in production

### Caching Strategy
- Use Russian doll caching for nested content
- Implement fragment caching for expensive views
- Cache at the appropriate level
- Use cache keys that auto-expire
- Monitor cache effectiveness

See **[performance.md](rails/performance.md)** for detailed optimization patterns.

## Security Best Practices

### Core Security Principles
- Always use strong parameters
- Implement proper authentication and authorization
- Sanitize user input appropriately
- Use CSRF protection for all forms
- Keep credentials in Rails credentials system

### Security Patterns
- Validate input at multiple levels
- Implement rate limiting for APIs
- Use secure headers in production
- Audit dependencies regularly
- Follow OWASP guidelines

See **[security.md](rails/security.md)** for comprehensive security patterns.

## Error Handling

### Error Handling Strategy
- Use Rails error reporter for centralized tracking
- Implement custom error pages
- Handle exceptions at appropriate levels
- Provide meaningful error messages
- Log errors with sufficient context

### Logging Best Practices
- Use appropriate log levels
- Include structured data in logs
- Avoid logging sensitive information
- Implement request correlation IDs
- Monitor logs for patterns

## Code Style & Conventions

### Method Organization
- Order methods logically: public, protected, private
- Group related methods together
- Use descriptive method names
- Keep methods small and focused
- Document complex logic

### Rails Conventions
- Use `?` suffix for boolean methods
- Use `!` suffix for dangerous methods
- Follow Rails naming patterns strictly
- Prefer Rails helpers over custom solutions
- Keep code idiomatic to Rails

### Code Quality Tools
- Use StandardRB for consistent formatting
- Run Bullet gem to detect N+1 queries
- Keep schema annotations current
- Use pre-commit hooks for quality
- Review code for Rails best practices

## Secrets & Configuration

### Credential Management
- Use Rails credentials for all secrets
- Never commit sensitive data
- Use environment variables for non-sensitive config
- Document credential requirements
- Rotate credentials regularly

### Configuration Best Practices
- Keep configuration DRY
- Use Rails configuration patterns
- Document environment-specific settings
- Validate configuration on boot
- Handle missing configuration gracefully

## AI Agent Collaboration

### Documentation for AI Agents
- Write clear, comprehensive comments
- Use consistent patterns throughout
- Document business logic thoroughly
- Explain non-obvious decisions
- Include examples where helpful

### Predictable Patterns
- Follow Rails conventions religiously
- Use standard file organization
- Keep naming consistent
- Write explicit rather than clever code
- Maintain comprehensive test coverage

### Task Breakdown
- Reference Linear tickets clearly
- Break complex tasks into phases
- Document dependencies between tasks
- Keep scope manageable
- Communicate progress clearly

## Development Workflow

### Project Management Integration
- The workflow supports various project management tools
- Linear integration is current default (see Linear Integration section)
- Adapt ticket references and workflows to your chosen platform
- Maintain consistent commit and branch naming patterns

### Local Development
- Use Rails generators appropriately
- Keep development close to production
- Use Rails console for debugging
- Implement helpful seed data
- Document setup requirements

### Git Workflow
- Follow GitHub Flow for simplicity and clarity
- Create feature branches from main/master
- Keep commits atomic and well-documented
- Open pull requests early for visibility
- Merge after review and CI checks pass

### Continuous Integration
- Run tests automatically on every push
- Deploy to staging via Kamal after main branch updates
- Use branch protection rules for quality gates
- Automate security and dependency checks
- Keep CI fast and focused on essentials

### Code Review
- Check for Rails best practices
- Verify test coverage
- Review security implications
- Ensure performance considerations
- Validate documentation completeness

## Production Considerations

### Deployment Checklist
- Run tests before deployment
- Check migration safety
- Verify environment configuration
- Monitor deployment progress
- Have rollback plan ready

### Production Best Practices
- Use health check endpoints
- Implement proper monitoring
- Configure appropriate timeouts
- Set up error alerting
- Plan for scaling

See **[deployment.md](rails/deployment.md)** for Kamal deployment details.

## Final Reminders

### Always Prefer Rails Conventions
- Trust the framework's decisions
- Use built-in solutions first
- Follow established patterns
- Avoid premature optimization
- Keep solutions simple

### Code Quality Checklist
- [ ] Tests written and passing
- [ ] No N+1 queries detected
- [ ] Code follows Rails conventions
- [ ] Security considerations addressed
- [ ] Performance implications considered
- [ ] Documentation is complete
- [ ] Linear ticket referenced
- [ ] AI agents can understand the code

Remember: Rails provides everything you need. Trust the framework, follow conventions, and focus on delivering value. Write code that's a joy to work with.

Happy coding! 🚂 🚀

---
> Source: [levifig/rails-instructions](https://github.com/levifig/rails-instructions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
