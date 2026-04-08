## laravel-mailtrap

> **Package**: `ronald2wing/laravel-mailtrap`

# Laravel Mailtrap Driver - Development Guide

## Project Overview

**Package**: `ronald2wing/laravel-mailtrap`
**Repository**: <https://github.com/ronald2wing/laravel-mailtrap>
**Maintainer**: Ronald Wong (<ronald2wing@gmail.com>)
**License**: MIT
**Version**: 1.0.0
**Status**: ✅ Stable & Actively Maintained

## Quick Start

```bash
# Clone repository
git clone https://github.com/ronald2wing/laravel-mailtrap.git
cd laravel-mailtrap

# Install dependencies
composer install

# Run tests
composer run test

# Run full quality check
composer run check
```

## Architecture Overview

### Core Components

| Component                   | Location                          | Responsibility                                                     |
| --------------------------- | --------------------------------- | ------------------------------------------------------------------ |
| **MailtrapServiceProvider** | `src/MailtrapServiceProvider.php` | Registers transport, validates config, creates HTTP client         |
| **MailtrapTransport**       | `src/MailtrapTransport.php`       | Symfony Mailer transport implementation, handles API communication |

### Design Patterns

- **Strategy Pattern**: Implements Symfony Mailer's `AbstractTransport`
- **Factory Pattern**: Service provider creates transport instances
- **Adapter Pattern**: Adapts Laravel mail API to Mailtrap REST API
- **Builder Pattern**: Method chaining for payload construction

## Development Workflow

### 1. Setup Development Environment

```bash
# Install dependencies with development tools
composer install --dev

# Configure PHPStan (already configured)
# Configure Laravel Pint (already configured)
```

### 2. Create Feature Branch

```bash
git checkout -b feature/your-feature-name
```

### 3. Implement Changes

Follow existing code patterns:

- Use type hints for all method parameters and return types
- Add PHPDoc blocks for public/protected methods
- Follow PSR-12 coding standards
- Write tests for new functionality

### 4. Run Quality Checks

```bash
# Run all checks (lint, analyse, test)
composer run check

# Or run individually:
composer run lint    # Code formatting check
composer run analyse # PHPStan static analysis (Level 8)
composer run test    # Run test suite
composer run format  # Auto-fix formatting issues
```

### 5. Commit and Push

```bash
git add .
git commit -m "feat: add your feature description"
git push origin feature/your-feature-name
```

## Code Quality Standards

### Static Analysis

- **PHPStan Level 8**: Maximum strictness
- **Type Coverage**: 100% for new code
- **Error Prevention**: All potential issues must be resolved

### Code Formatting

- **Laravel Pint**: Consistent code style
- **PSR-12**: PHP coding standards
- **Auto-fix**: Use `composer run format` to fix formatting

### Testing Requirements

- **Coverage**: 100% for new features
- **Mocking**: Use Mockery for HTTP client
- **Patterns**: Follow existing test structure
- **Data**: Use constants for test data

## Documentation

### Documentation Structure

| Document         | Purpose                                 | Audience           |
| ---------------- | --------------------------------------- | ------------------ |
| **README.md**    | User installation, configuration, usage | End users          |
| **CHANGELOG.md** | Version history, release notes          | Users & developers |
| **AGENTS.md**    | Development guide, workflows            | Developers         |

### Documentation Standards

- **Clarity**: Clear, concise language with examples
- **Completeness**: Cover all features and edge cases
- **Consistency**: Follow established patterns
- **Maintainability**: Keep docs updated with code changes

## Dependency Management

### Version Compatibility

| Laravel Version | Testbench Version | PHP Version |
| --------------- | ----------------- | ----------- |
| 10.x            | ^8.0              | 8.2+        |
| 11.x            | ^9.0              | 8.2+        |
| 12.x            | ^10.0             | 8.2+        |

### Composer Configuration

- `illuminate/mail`: `^10.0|^11.0|^12.0`
- `illuminate/support`: `^10.0|^11.0|^12.0`
- `orchestra/testbench`: `^8.0|^9.0|^10.0`

## CI/CD Pipeline

### GitHub Actions Workflow

- **File**: `.github/workflows/php.yml`
- **Matrix**: PHP 8.2, 8.3, 8.4, 8.5 × Laravel 10, 11, 12
- **Steps**: Setup → Validate → Cache → Install → Test → Analyse → Lint

### Code Coverage

- **Tool**: PCOV for coverage reports
- **Format**: Clover XML for Codecov integration
- **Reports**: HTML for local development
- **Integration**: Automatic upload to Codecov

## Release Process

For detailed release instructions, see the [Release Process section in CHANGELOG.md](CHANGELOG.md#release-process).

## Troubleshooting Development Issues

### Common Issues

1. **PHPStan errors**: Check type hints and PHPDoc blocks
2. **Test failures**: Ensure Mockery expectations match implementation
3. **CI failures**: Check Laravel version compatibility
4. **Coverage drops**: Add tests for new code paths

### Getting Help

- Check existing tests for patterns
- Review similar implementations in codebase
- Run `composer run check` to identify issues
- Consult AGENTS.md for development guidelines

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ronald2wing)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ronald2wing)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
