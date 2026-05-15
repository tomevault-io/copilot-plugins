## ex-llm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Version Management

### When to Bump Versions
- **Patch version (0.x.Y)**: Bug fixes, documentation updates, minor improvements
- **Minor version (0.X.0)**: New features, non-breaking API changes, new provider adapters
- **Major version (X.0.0)**: Breaking API changes (after 1.0.0 release)

### Version Update Checklist
1. Update version in `mix.exs`
2. Update CHANGELOG.md with:
   - Version number and date
   - Added/Changed/Fixed/Removed sections
   - **BREAKING:** prefix for any breaking changes
3. Commit with message: `chore: bump version to X.Y.Z`

### CHANGELOG Format
```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added
- New features or providers

### Changed
- Changes in existing functionality
- **BREAKING:** API changes that break compatibility

### Fixed
- Bug fixes

### Removed
- Removed features
- **BREAKING:** Removed APIs
```

## Feature Status

### ✅ Stable Features (Production Ready)
- **Core Chat API**: `ExLLM.chat/3` - Basic chat functionality with all providers
- **Streaming API**: `ExLLM.stream/4` - Real-time response streaming
- **Session Management**: `ExLLM.Core.Session` - Conversation persistence and state
- **Provider Support**: All major providers (OpenAI, Anthropic, Gemini, Ollama, etc.)
- **Function Calling**: Tool use and function calling across providers
- **Cost Tracking**: Token usage and cost calculation
- **Authentication**: API key management and OAuth2 (Gemini)
- **Configuration**: YAML-based model and provider configuration
- **Error Handling**: Comprehensive error handling and recovery
- **Test Caching**: Advanced response caching for 25x faster integration tests

### 🚧 Incomplete Features (Under Development)
- **Context Management**: `ExLLM.Core.Context.truncate_messages/5` - Automatic message truncation for token limits
- **Model Capabilities API**: `ExLLM.Infrastructure.Config.ModelConfig.get_model_config/1` - Programmatic model metadata access
- **Configuration Validation**: Runtime configuration validation utilities

### 📋 Testing Status
- **Core Functionality**: 100% tested and working (8/8 tests pass)
- **Comprehensive Suite**: 80% tested and working (12/15 tests pass)
- **Manual Testing**: All user-facing features verified working end-to-end

**Note**: The incomplete features are advanced/internal APIs that don't affect normal usage. All user-facing functionality in the example app works perfectly.

## Commands

### Development
```bash
# Run the application
iex -S mix

# Install dependencies
mix deps.get
mix deps.compile
```

### Testing

ExLLM includes a comprehensive testing system with intelligent caching and 24 specialized Mix aliases.

#### Basic Testing
```bash
# Run all tests (fast - uses cache when available)
mix test

# Run a specific test file
mix test test/ex_llm_test.exs

# Run tests at a specific line
mix test test/ex_llm_test.exs:42

# Run tests with coverage
mix test --cover
```

#### Streamlined Testing (12 Mix Aliases)
```bash
# === CORE TESTING STRATEGY ===
mix test.fast           # Fast development tests (excludes API calls and slow tests)
mix test.unit           # Unit tests only (pure logic, no external dependencies)  
mix test.integration    # Integration tests (live APIs by default, requires API keys)
mix test.live           # Force live API tests (bypasses cache, refreshes if enabled)
mix test.all            # All tests including slow/comprehensive suites

# === PROVIDER-SPECIFIC TESTING ===
mix test.anthropic      # Anthropic Claude tests
mix test.openai         # OpenAI GPT tests
mix test.gemini         # Google Gemini tests  
mix test.local          # Local providers (Ollama, LM Studio)
mix test.bumblebee      # Bumblebee tests (requires opt-in due to large downloads)

# === SPECIALIZED TESTING ===
mix test.oauth2         # OAuth2 authentication tests
mix test.ci             # CI/CD pipeline tests (excludes flaky/slow)

# === CACHE MANAGEMENT ===
mix cache.clear         # Clear test cache
mix cache.status        # Check cache status
```

#### Test Caching (25x Speed Improvement)

**Integration tests run against live APIs by default.** Test caching is disabled by default and must be explicitly enabled.

```bash
# DEFAULT BEHAVIOR: Integration tests hit live APIs
mix test --include integration     # Calls live provider APIs

# ENABLE CACHING: Use cached responses when available
export EX_LLM_TEST_CACHE_ENABLED=true
mix test --include integration     # Uses cache if fresh, otherwise excludes tests

# FORCE LIVE: Always use live APIs regardless of cache settings  
MIX_RUN_LIVE=true mix test --include integration

# Manage test cache
mix ex_llm.cache stats
mix ex_llm.cache clean --older-than 7d
mix ex_llm.cache clear
mix ex_llm.cache show anthropic

# Enable cache debugging
export EX_LLM_LOG_LEVEL=debug
```

#### Running Tests with Tags
```bash
# Include specific test types
mix test --include live_api
mix test --include requires_api_key
mix test --include oauth2

# Run only specific tags
mix test --only provider:anthropic
mix test --only streaming
mix test --only integration

# Exclude problematic tests
mix test --exclude requires_service
mix test --exclude slow
```

### Code Quality
```bash
# Format code
mix format

# Run linter
mix credo

# Run type checker
mix dialyzer
```

### Debug Logging

ExLLM includes a comprehensive debug logging system that can be configured per environment:

```bash
# Test debug logging example
elixir examples/debug_logging_example.exs
```

**Configuration Options:**
- `log_level`: `:debug`, `:info`, `:warn`, `:error`, or `:none`
- `log_components`: Enable/disable logging for specific components
- `log_redaction`: Control redaction of sensitive data

**Development Environment:** Enhanced logging with all components enabled
**Test Environment:** Minimal logging to reduce noise  
**Production Environment:** Configurable logging with security redaction

### Documentation
```bash
# Generate documentation
mix docs

# Open documentation in browser
open doc/index.html
```

## Model Updates

### Updating Model Metadata

The project uses two methods to keep model configurations up-to-date:

1. **Sync from LiteLLM** (comprehensive metadata for all providers):
   ```bash
   # The wrapper script handles virtual environment automatically
   ./scripts/update_models.sh --litellm
   
   # Or manually with venv
   source .venv/bin/activate
   python scripts/sync_from_litellm.py
   ```
   - Updates configurations for all 56 providers (not just implemented ones)
   - Includes pricing, context windows, and capabilities
   - Requires LiteLLM repository to be cloned alongside this project

2. **Fetch from Provider APIs** (latest models from provider APIs):
   ```bash
   # Activate virtual environment first
   source .venv/bin/activate
   
   # Update all providers with APIs
   source ~/.env && uv run python scripts/fetch_provider_models.py
   
   # Or use the wrapper script (handles venv automatically)
   ./scripts/update_models.sh
   ```
   - Fetches latest models directly from provider APIs
   - Requires API keys in environment variables for some providers
   - Updates: Anthropic, OpenAI, Groq, Gemini, OpenRouter, Ollama

### Important Notes
- **Virtual Environment**: Always activate `.venv` before running Python scripts directly
- **Dependencies**: Python scripts require `pyyaml` and `requests` (installed in `.venv`)
- Always preserve custom default models (e.g., `gpt-4.1-nano` for OpenAI)
- The fetch script may reset defaults, so check and fix them after running
- Both scripts update YAML files in `config/models/`
- Commit changes with descriptive messages about what was updated
- **Use Python Virtual Environment and uv to interact with Python**

## Architecture Overview

ExLLM is a unified Elixir client for Large Language Models that provides a consistent interface across multiple providers. The architecture follows these key principles:

### Core Components

1. **Main Module (`lib/ex_llm.ex`)**: Entry point providing the unified API for all LLM operations. Delegates to appropriate adapters based on provider.

2. **Adapter Pattern (`lib/ex_llm/adapter.ex`)**: Defines the behavior that all provider implementations must follow. Each provider (Anthropic, OpenAI, Gemini, etc.) implements this behavior.

3. **Session Management (`lib/ex_llm/session.ex`)**: Manages conversation state across multiple interactions, tracking messages and token usage.

4. **Context Management (`lib/ex_llm/context.ex`)**: Handles message truncation and validation to ensure conversations fit within model context windows using different strategies (sliding_window, smart).

5. **Cost Tracking (`lib/ex_llm/cost.ex`)**: Automatically calculates and tracks API costs based on token usage and provider pricing.

6. **Test Caching System**: Advanced caching infrastructure for 25x faster integration tests
   - **Cache Storage (`lib/ex_llm/cache/storage/test_cache.ex`)**: JSON-based persistent storage
   - **Cache Detection (`lib/ex_llm/test_cache_detector.ex`)**: Smart detection of cacheable tests
   - **Response Interception (`lib/ex_llm/test_response_interceptor.ex`)**: Transparent HTTP caching
   - **Cache Management (`lib/mix/tasks/ex_llm.cache.ex`)**: Mix task for cache operations

7. **Instructor Integration (`lib/ex_llm/instructor.ex`)**: Optional integration for structured outputs with schema validation and retries.

### Provider Implementations

- **Anthropic Provider (`lib/ex_llm/providers/anthropic.ex`)**: Claude API integration with streaming support
- **OpenAI Provider (`lib/ex_llm/providers/openai.ex`)**: GPT and o1 models with function calling  
- **Gemini Provider (`lib/ex_llm/providers/gemini.ex`)**: Complete Gemini API suite (15 APIs) with OAuth2
- **Groq Provider (`lib/ex_llm/providers/groq.ex`)**: Ultra-fast inference with Llama and DeepSeek
- **OpenRouter Provider (`lib/ex_llm/providers/openrouter.ex`)**: Access to 300+ models
- **Local Providers**: Ollama, LM Studio, and Bumblebee for local model inference

### Configuration System

The library uses a pluggable configuration provider system (`lib/ex_llm/config_provider.ex`) that allows different configuration sources (environment variables, static config, custom providers).

### Type System

All data structures are defined in `lib/ex_llm/types.ex` with comprehensive typespecs for:
- `LLMResponse`: Standard response structure with content, usage, and cost data
- `StreamChunk`: Streaming response chunks
- `Model`: Model metadata including context windows and capabilities
- `Session`: Conversation state tracking

### Application Lifecycle

The `ExLLM.Application` module starts the supervision tree, including the optional `ModelLoader` GenServer for local model management.

## Key Design Patterns

1. **Unified Interface**: All providers expose the same API through the main `ExLLM` module
2. **Provider Pattern**: Each provider implements the `ExLLM.Provider` behavior
3. **Tesla Middleware Architecture**: Modern HTTP client with composable middleware stack
4. **Intelligent Test Caching**: Automatic response caching with smart cache invalidation (25x speedup)
5. **Semantic Test Organization**: Tag-based test categorization with automatic requirement checking
6. **Optional Dependencies**: Features like local models and structured outputs are optional
7. **Automatic Cost Tracking**: Usage and costs are calculated transparently
8. **Context Window Management**: Automatic message truncation based on model limits
9. **Enhanced Streaming**: Real-time responses with direct HTTP.Core integration and error recovery

## HTTP Client Architecture Migration

**Status: Complete Migration to HTTP.Core (2025-06-26)**

ExLLM has successfully migrated from a legacy HTTPClient facade to a modern Tesla middleware-based HTTP architecture:

### ✅ **Completed Migrations**
- **All Core Providers**: OpenAI (55 calls), Anthropic (14 calls), OpenAI-Compatible base (affects Mistral, XAI, OpenRouter, Perplexity), Gemini modules, Ollama, Groq, LMStudio
- **HTTP.Core**: New Tesla-based client with middleware for authentication, error handling, caching, and multipart uploads
- **Provider-Specific Helpers**: Each provider now has optimized request helpers that handle all HTTP methods and streaming
- **Streaming Coordinators**: Both StreamingCoordinator and EnhancedStreamingCoordinator now use HTTP.Core.stream directly
- **ModelFetcher**: Migrated to use HTTP.Core for fetching provider model lists

### 🔧 **Architecture Benefits**
- **Better Error Handling**: Comprehensive retry logic with exponential backoff
- **Improved Authentication**: Provider-specific auth middleware with automatic header management  
- **Enhanced Streaming**: Direct HTTP.Core.stream integration with callback support
- **Modular Design**: Focused middleware components for specific concerns
- **Test Support**: Automatic response caching for 25x faster integration tests

### 📁 **New HTTP Module Structure**
```
lib/ex_llm/providers/shared/http/
├── core.ex              # Main Tesla client and middleware management
├── authentication.ex   # Provider-specific auth headers  
├── error_handling.ex   # Retry logic and error mapping
├── multipart.ex        # File upload handling
└── safe_hackney_adapter.ex # Secure HTTP adapter
```

### ✅ **Legacy Component Cleanup Complete**
- **Stream Parsing Migration**: Successfully replaced all 13 legacy StreamParseResponse modules with modern ParseStreamResponse plugs
- **HTTP.Core Adoption**: All streaming components now use HTTP.Core.stream directly instead of HTTPClient.stream_request
- **Pipeline Modernization**: Updated Core.Chat pipeline configurations to use new plug-based streaming approach
- **Zero Legacy Dependencies**: No remaining references to HTTPClient.stream_request in the codebase

### 📝 **Migration Notes**
- **HTTPClient.post_stream** is now deprecated and serves as a compatibility shim
- New code should use `ExLLM.Providers.Shared.HTTP.Core` directly
- All active providers and coordinators have been migrated to the new architecture

## Test Configuration

**Status: Centralized Configuration Complete (2025-06-25)**

ExLLM uses a centralized test configuration system for consistent behavior across all environments:

### 🎯 **Centralized Config (`ExLLM.Testing.Config`)**
- **Single source of truth**: All test settings defined in one module
- **Semantic categories**: Tests organized by tags (unit, integration, provider, etc.)
- **Dynamic exclusions**: Smart cache-based and environment-based test selection
- **Maintainable**: Easy to add new test categories and strategies

### 📋 **Test Strategies**

#### CI Pipeline (`test.ci`)
- **Fast execution**: Excludes slow and problematic tests (`wip`, `flaky`, `quota_sensitive`, `very_slow`)
- **Reliable**: Focuses on core functionality without external dependencies
- **Used in**: GitHub Actions CI workflow for pull requests and pushes

#### Release Pipeline (`test.integration`) 
- **Comprehensive**: Includes integration and external tests
- **Cached responses**: Uses test caching for faster integration test execution
- **Mock API keys**: Provides mock credentials for integration tests
- **Used in**: GitHub Actions release workflow for validation before publishing

#### Hybrid Strategy (Default)
- **Cache-aware**: Includes integration tests when cache is fresh (<24h)
- **Fallback**: Excludes external tests when cache is stale
- **Override**: `MIX_RUN_LIVE=true` forces live API calls

### 🗂️ **File Organization**
- `lib/ex_llm/testing/config.ex` - Centralized configuration module
- `test/test_helper.exs` - Applies centralized settings
- `test/CENTRALIZED_CONFIG.md` - Detailed documentation
- Config files delegate to centralized module

## Environment Variables

**Status: Centralized Environment Configuration Complete (2025-06-25)**

ExLLM uses centralized environment variable management through the `ExLLM.Environment` module:

### 🎯 **Centralized Module (`ExLLM.Environment`)**
- **Single source of truth**: All environment variables documented in one module
- **Consistent naming**: Standardized variable names across all providers
- **Type-safe access**: Helper functions for accessing provider configuration
- **Auto-discovery**: Check which providers have API keys configured

### 📋 **Key Environment Variables**

#### Provider API Keys
- `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`, etc.
- See [ENVIRONMENT.md](ENVIRONMENT.md) for complete reference

#### Test Environment
- `EX_LLM_TEST_CACHE_ENABLED` - Enable test response caching
- `EX_LLM_LOG_LEVEL` - Control test logging verbosity
- `MIX_RUN_LIVE` - Force live API calls in tests

### 🗂️ **File Organization**
- `lib/ex_llm/environment.ex` - Centralized environment module
- `ENVIRONMENT.md` - Complete environment variable reference
- Provider modules use `ExLLM.Environment` for configuration

## Provider Testing and API Keys

- Anytime you run live API tests, use the `scripts/run_with_env.sh` script to set the api keys
- Environment variables are documented in [ENVIRONMENT.md](ENVIRONMENT.md)

### OAuth2 Testing Requirements

**MANDATORY**: All OAuth2 tests MUST use `ExLLM.Testing.OAuth2TestCase` for consistent token handling.

```elixir
# Required pattern for OAuth2 tests
defmodule MyOAuth2Test do
  use ExLLM.Testing.OAuth2TestCase, timeout: 300_000
  
  test "oauth2 functionality", %{oauth_token: token} do
    # Test logic with automatic token refresh
  end
end
```

**OAuth2 Setup Requirements:**
- Environment variables: `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`
- Initial setup: `elixir scripts/setup_oauth2.exs`
- Token file: `.gemini_tokens` (created by setup script)

**Code Review Guidelines:**
- Verify OAuth2 tests use `OAuth2TestCase` (not `ExUnit.Case`)
- Check for proper error handling with `gemini_api_error?/2`
- Ensure unique resource names with `unique_name/1`
- Confirm automatic cleanup is not manually overridden

See [OAuth2 Testing Guide](test/OAUTH2_TESTING.md) for complete documentation.

## HTTP Client Migration Lessons Learned

**Status: Callback Signature Migration Complete (2025-06-26)**

During the HTTP client migration from HTTPClient to HTTP.Core, we encountered and resolved several critical callback signature mismatches:

### 🔧 **Key Lesson: Callback Signature Compatibility**

**Problem**: HTTP.Core.stream calls callbacks with 1 argument, but legacy StreamingCoordinator expected 2-argument accumulator pattern.

**Root Cause**: 
- Legacy: `callback.(chunk, accumulator)` → `{:cont, new_accumulator}`
- New: `callback.(chunk)` → direct processing

**Solution Pattern**: Use Agent for internal state management in streaming callbacks:

```elixir
# ✅ Correct: Agent-based state management
{:ok, state_agent} = Agent.start_link(fn -> initial_state end)

callback = fn chunk ->
  # Process chunk and update Agent state
  Agent.update(state_agent, fn state -> new_state end)
end

# Later: Retrieve state
final_state = Agent.get(state_agent, & &1)
```

**Anti-Pattern**:
```elixir
# ❌ Incorrect: Variable shadowing in closures
chunks = []
callback = fn chunk ->
  chunks = [chunk | chunks]  # Variable shadowing error!
end
```

### 🎯 **Critical Migration Steps**
1. **Audit all callback signatures** in streaming components
2. **Replace accumulator patterns** with Agent-based state management  
3. **Test both sync and async callback paths** thoroughly
4. **Reset circuit breakers** in test setup to prevent state pollution

### 📋 **Files Updated**
- `lib/ex_llm/providers/shared/streaming_coordinator.ex` - Callback adapter pattern
- `test/ex_llm/providers/shared/streaming_performance_test.exs` - Agent patterns
- `test/ex_llm/providers/shared/http_core_streaming_test.exs` - Variable fixes
- `test/support/shared/provider_integration_test.exs` - Circuit breaker resets

## Memory and Development Tips

### Development Workflow
- Whenever you need to write or edit code, prefer to use aider.
- After new code is written, use zen to review the code.
- **IMPORTANT WORKFLOW TIP**: Before writing or editing code, use zen to analyze it, get consensus on it, and plan it.

---
> Source: [azmaveth/ex_llm](https://github.com/azmaveth/ex_llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
