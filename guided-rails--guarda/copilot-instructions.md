## guarda

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Guarda

Guarda is a lightweight Ruby authorization gem. It uses Rails' `CurrentAttributes` (`Current.person`) for the authenticated user and convention-based policy class discovery.

## Commands

```bash
bundle exec rake test                                    # Run full test suite
bundle exec rake test TEST=test/authorization_test.rb    # Run a single test file
bundle exec rake build                                   # Build the gem
bundle exec rake release                                 # Release to RubyGems
```

## Architecture

The gem has three core components in `lib/`:

- **`guarda.rb`** — Root module. Defines the exception hierarchy: `Guarda::Error`, `NotAuthorizedError`, `AuthorizationNotPerformedError`, `PolicyFinder::NotFoundError`.
- **`guarda/authorization.rb`** — An `ActiveSupport::Concern` included in controllers. Provides `authorize(record=nil)`, `policy(controller, record=nil)`, and `verify_authorization_performed`. The `policy` method is also exposed as a view helper.
- **`guarda/policy_finder.rb`** — Resolves controller paths to policy classes by convention (e.g., `"posts"` → `PostsPolicy`, `"admin/tests"` → `Admin::TestsPolicy`). Uses `safe_constantize`.

## Policy Convention

Policy classes are plain Ruby classes named `{Controller}Policy` with predicate methods matching action names (e.g., `index?`, `show?`, `update?`). They receive an optional record in their constructor. The authenticated user is accessed via `Current.person` inside the policy.

## Test Setup

Tests use Minitest with `ActiveSupport::TestCase`. The `test/test_helper.rb` defines mock objects: a `Controller` class that includes `Guarda::Authorization`, test policy classes (`TestsPolicy`, `Admin::TestsPolicy`), and a `Record` struct.

## Dependencies

- Ruby >= 3.1.0
- `activesupport ~> 7.1` (runtime)
- `minitest`, `minitest-reporters`, `rake` (development)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Guided-Rails) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
