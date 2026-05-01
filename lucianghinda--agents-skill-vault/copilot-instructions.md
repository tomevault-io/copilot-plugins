## agents-skill-vault

> A Ruby gem for managing a local vault of GitHub resources (repositories, folders, or files). Supports sparse checkout, sync management, and skill validation.

# AgentsSkillVault

A Ruby gem for managing a local vault of GitHub resources (repositories, folders, or files). Supports sparse checkout, sync management, and skill validation.

## Quick Reference

```bash
# Run tests
bundle exec rake test

# Run linter
bundle exec rake rubocop

# Run both (default task)
bundle exec rake

# Interactive console
bin/console
```

## Architecture

### Core Components

- **Vault** (`lib/agents_skill_vault/vault.rb`) - Main interface for managing resources. Handles add, sync, remove, validate operations.
- **UrlParser** (`lib/agents_skill_vault/url_parser.rb`) - Parses GitHub URLs into components (username, repo, branch, path, type).
- **GitOperations** (`lib/agents_skill_vault/git_operations.rb`) - Git commands wrapper for clone, sparse checkout, pull.
- **Resource** (`lib/agents_skill_vault/resource.rb`) - Data object representing a tracked resource.
- **Manifest** (`lib/agents_skill_vault/manifest.rb`) - JSON manifest persistence for tracking resources.
- **SkillScanner** (`lib/agents_skill_vault/skill_scanner.rb`) - Scans directories for SKILL.md files.
- **SkillValidator** (`lib/agents_skill_vault/skill_validator.rb`) - Validates SKILL.md file structure.
- **SyncResult** (`lib/agents_skill_vault/sync_result.rb`) - Result object for sync operations.
- **Errors** (`lib/agents_skill_vault/errors/`) - Custom error classes.

### Key Patterns

**Label Format**: Labels follow `username/repo/skill_name` format for skills and folders, `username/repo` for repositories.

**Storage Structure**: Resources are stored at `<VAULT>/username/repo/relative_path`.

**URL Types**:
- `:repo` - Full repository (`https://github.com/user/repo`)
- `:folder` - Directory via tree (`https://github.com/user/repo/tree/main/path`)
- `:file` - File via blob (`https://github.com/user/repo/blob/main/path/SKILL.md`)

**Autoloading**: Uses Zeitwerk for autoloading. File names must match class names (snake_case to CamelCase).

## Testing

- **Framework**: Minitest
- **Mocking**: Mocha (`mocha/minitest`)
- **Test files**: `test/*_test.rb`
- **Helper**: `test/test_helper.rb` provides `TestHelper.temp_vault_path` for temp directories

### Test Conventions

```ruby
class MyClassTest < Minitest::Test
  def setup
    @storage_path = TestHelper.temp_vault_path
    FileUtils.mkdir_p(@storage_path)
  end

  def teardown
    FileUtils.rm_rf(@storage_path)
  end

  def test_something
    # Use stubs for GitOperations to avoid real git calls
    AgentsSkillVault::GitOperations.stubs(:clone_repo)
    AgentsSkillVault::GitOperations.stubs(:sparse_checkout)
  end
end
```

## Code Style

- Ruby 3.4+ required
- Double quotes for strings (configured in `.rubocop.yml`)
- `frozen_string_literal: true` in all Ruby files
- YARD-style documentation for public methods
- Max method length: 20 lines
- Max class length: 150 lines

## Dependencies

- `addressable` - URL parsing with proper encoding
- `zeitwerk` - Autoloading
- `mocha` - Test mocking (development)

## Git Requirements

Requires Git 2.25.0+ for sparse checkout support. The gem validates git availability on Vault initialization.

---
> Source: [lucianghinda/agents_skill_vault](https://github.com/lucianghinda/agents_skill_vault) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
