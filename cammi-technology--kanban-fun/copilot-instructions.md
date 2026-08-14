## kanban-fun

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `bin/setup` - Complete development environment setup (installs dependencies, prepares database, starts server)
- `bin/dev` - Start the development server
- `bin/rails server` - Start Rails server directly
- `bin/rails test` - Run all unit tests
- `bin/rails test:system` - Run system tests
- `bin/rubocop -a` - Run linter with auto-correction
- `bundle exec brakeman` - Run security scanner
- `rake zeitwerk:check` - Verify autoloading setup
- `bin/importmap audit` - Check JavaScript dependencies for vulnerabilities

## Application Architecture

This is a Rails 8 application called "Wokecamp" - a kanban-style project management system with multi-tenancy.

### Core Models & Hierarchy
- **User** - Individual users with authentication (has_secure_password, 2FA support)
- **Account** - Multi-tenant organizations owned by users
- **AccountUser** - Join table connecting users to accounts with permissions
- **Project** - Kanban boards belonging to accounts
- **Post** - Cards/items within projects (with ActionText rich content)
- **Comment** - Nested comments on posts (polymorphic commentable concern)

### Key Architectural Patterns
- **Current Attributes**: Thread-local state management via `Current` class
- **Pundit Authorization**: Policy-based authorization - always specify the action being tested explicitly rather than inferring it
- **Phlex 2 Views**: All views, layouts, and components use Phlex 2 instead of ERB templates
- **Multi-tenancy**: Account-scoped resources with `Current.account_user` context
- **Record Association**: Shared polymorphic commenting functionality
- **Noticed Notifications**: Web push notifications for new posts

### Authentication & Security
- Session-based authentication with signed cookies
- Two-factor authentication (TOTP) support
- Sudo mode for sensitive operations
- Email verification with token generation
- Password strength validation via Pwned gem
- Brakeman security scanning in pre-commit hooks

### Frontend Stack
- **Phlex 2**: View layer with Ruby components for layouts, views, and UI components
- **Stimulus Controllers**: JavaScript behavior (mentions, notifications)
- **ActionText**: Rich text editing with Trix
- **Importmap**: ES6 modules without bundling
- **Tribute.js**: @mentions functionality with SGID attachments
- **Web Push**: Browser notifications

### Components
- **CommentForm**: Reusable component for creating and editing comments with mentions support. Accepts `record` (polymorphic) and optional `comment` parameters. Handles both new comment creation and existing comment editing.

### Testing Setup
- Minitest with parallel execution
- System tests with Capybara/Selenium
- Test helpers for authentication and authorization
- Mocha for mocking
- Fixtures-based test data

### Git Hooks (Lefthook)
Pre-commit runs linting, security scans, and autoloading checks. Pre-push runs full test suite and dependency audits.

## Coding Style Guidelines

### Ruby Method Calls Across Multiple Lines

When method calls span multiple lines, use parentheses with the opening parenthesis on the same line as the method name:

```ruby
# Preferred
patch(
  account_project_post_url(@account, @project, posts(:post)),
  params: { post: { title: "Updated Title", content: "Updated content" } }
)

# Not preferred  
patch account_project_post_url(@account, @project, posts(:post)),
      params: { post: { title: "Updated Title", content: "Updated content" } }
```

For single line method calls, parentheses are optional.

### Test Variable Assignment

In test files, avoid creating unnecessary local variables:

```ruby
# Preferred - reuse setup instance variables or inline fixture calls
def test_update
  refute_permit(@account_owner_user, @post, :update)
end

# Preferred - use fixtures directly if only used once
def test_create  
  refute_permit(account_users(:other_user), @post, :create)
end

# Not preferred - unnecessary local variable assignment
def test_update
  account_owner_user = account_users(:account_owner)
  refute_permit(account_owner_user, @post, :update)
end
```

When the same fixture is used multiple times within a test class, assign it to an instance variable in the `setup` method rather than creating local variables in each test method.

---
> Source: [Cammi-Technology/kanban.fun](https://github.com/Cammi-Technology/kanban.fun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
