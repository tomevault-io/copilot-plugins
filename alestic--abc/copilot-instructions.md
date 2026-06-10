## abc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

abc (AI Bash Command) is a command-line tool that translates natural language descriptions into shell commands using LLMs. It uses a plugin architecture to support multiple LLM providers.

## Architecture

### Core Components
- **abc_cli**: Main package containing command generation logic
  - `abc_generate.py`: CLI entry point that handles command generation
  - `llm_provider.py`: Abstract base class that all providers must implement
  - `abc_setup.py`: Installation script for shell integration
  - Shell scripts (`abc.sh`, `abc.tcsh`) provide the `abc` shell function

### Provider Plugins
- **abc_provider_anthropic**: Anthropic Claude provider
- **abc_provider_aws_bedrock**: AWS Bedrock provider
- **abc_provider_openai**: OpenAI GPT provider
- Providers are discovered via Python entry points defined in pyproject.toml

### Key Design Patterns
1. The `abc` command is a shell function (not a direct Python script) that calls `abc_generate`
2. Providers implement the `LLMProvider` abstract base class
3. Configuration uses INI format with sections for different providers
4. Shell integration must be idempotent (safe to run multiple times)

## Development Commands

### Installation
```bash
# Development install (using Makefile - recommended)
make install-dev

# Manual development install
pipx install -e .
pipx inject abc-cli -e ./abc_provider_anthropic
pipx inject abc-cli -e ./abc_provider_aws_bedrock
pipx inject abc-cli -e ./abc_provider_openai
abc_setup --no-prompt

# Regular install from source
make install

# Nix install (alternative)
make install-nix
```

### Testing
```bash
# Run all tests (using Makefile - creates venv automatically)
make test

# Run tests with coverage report
COVERAGE=1 make test

# Manual testing (requires setting up venv first)
python -m pytest

# Run with coverage
python -m pytest --cov=abc_cli

# Run specific test file
python -m pytest abc_cli/tests/test_abc_generate.py

# Test specific provider
python -m pytest abc_provider_anthropic/tests/
python -m pytest abc_provider_aws_bedrock/tests/
python -m pytest abc_provider_openai/tests/
```

### Development Utilities
```bash
# Show project file tree
make tree

# Clean build artifacts and cache
make clean

# Re-run shell setup (if abc command stops working)
make setup
```

### Uninstall
```bash
# Complete uninstall (using Makefile)
make uninstall

# Manual uninstall
abc_setup --uninstall --no-prompt
pipx uninstall abc-cli

# Nix uninstall
make uninstall-nix
```

## Critical Implementation Details

1. **Python Compatibility**: Must support Python ≥ 3.8
2. **Shell Function**: The `abc` command users type is a shell function that:
   - Calls `abc_generate` with the user's description
   - Presents the generated command in an editable prompt
   - Adds executed commands to shell history
3. **Configuration Priority**:
   - Primary: `~/.config/abc/config` (XDG standard)
   - Legacy: `~/.abc.conf` (for backward compatibility)
4. **Danger Level Evaluation**: Commands are evaluated for potential harm before presentation
5. **Version Updates**: When releasing, update ALL version strings to the same date (YYYY.MM.DD format).
   To find all version strings: `git grep -i 'version.*20'`
6. **Testing Structure**: Tests use pytest with markers (unit, integration, slow) and separate test suites for each provider

## Working with Providers

When implementing or modifying providers:
1. Inherit from `abc_cli.llm_provider.LLMProvider`
2. Implement required methods: `configure()`, `generate_command()`, `is_configured()`
3. Register via entry point in pyproject.toml
4. Support both environment variables and config file for API keys
5. Handle danger level evaluation in generated commands

## Key File Locations

- **Shell Integration**: `abc_cli/abc.sh` and `abc_cli/abc.tcsh` - shell functions that users actually run
- **Main CLI**: `abc_cli/abc_generate.py` - core command generation logic
- **Provider Base**: `abc_cli/llm_provider.py` - abstract base class for all providers
- **Setup Script**: `abc_cli/abc_setup.py` - handles shell integration installation
- **Config Template**: `abc_cli/abc.conf.template` - example configuration

<!-- [Created by AI: Claude Code] -->

---
> Source: [alestic/abc](https://github.com/alestic/abc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
