## 001-symindx-workspace

> APPLY SYMindX architecture standards when working with any file to ensure consistency

# SYMindX Workspace Architecture

## Project Overview

SYMindX is an intelligent AI agent framework built with TypeScript, using Bun as the runtime and package manager. The system provides modular, hot-swappable components for memory, emotion, and cognition with multi-platform capabilities.

### Core Architecture Components

**🧠 Core Runtime System** (`mind-agents/`)
- Event-driven architecture with centralized Event Bus
- Hot-swappable module registry for memory, emotion, cognition
- Multi-agent coordination and lifecycle management
- Extension system for platform integrations

**🌐 AI Portal Architecture** (`mind-agents/src/portals/`)
- Multi-provider AI integration (OpenAI, Anthropic, Groq, xAI, Google)
- Vercel AI SDK v5 implementation
- Unified interface for model switching and optimization

**💾 Memory System** (`mind-agents/src/memory/`)
- SQLite, PostgreSQL, Supabase, Neon providers
- Vector embeddings and semantic search
- Conversation persistence and context management

**❤️ Emotion System** (`mind-agents/src/emotion/`)
- 11 distinct emotions (RuneScape-inspired)
- Dynamic emotional state management
- Contextual emotion weighting and responses

**🎯 Cognition Modules** (`mind-agents/src/cognition/`)
- HTN (Hierarchical Task Network) planner
- Reactive response system
- Hybrid cognitive architectures

**🔌 Platform Extensions** (`mind-agents/src/extensions/`)
- Telegram, Slack, Discord integrations
- RuneLite/RuneScape game integration
- Twitter/X social platform support

### Directory Structure

```
symindx/                          # Root project (Bun workspace)
├── .cursor/                      # Cursor IDE configuration
│   ├── rules/                    # Cursor rules (001-017 core)
│   ├── docs/                     # Quick start, architecture, contributing
│   └── tools/                    # Project analyzer, code generator, debugging
├── mind-agents/                  # Core agent runtime system
│   ├── src/
│   │   ├── core/                 # Runtime, EventBus, ModuleRegistry
│   │   ├── portals/              # AI providers (15+ workspaces)
│   │   │   ├── openai/           # OpenAI portal workspace
│   │   │   ├── anthropic/        # Anthropic portal workspace
│   │   │   ├── groq/             # Groq portal workspace
│   │   │   ├── xai/              # xAI portal workspace
│   │   │   ├── google-vertex/    # Google Vertex workspace
│   │   │   ├── google-generative/ # Google Generative workspace
│   │   │   ├── mistral/          # Mistral portal workspace
│   │   │   ├── cohere/           # Cohere portal workspace
│   │   │   ├── azure-openai/     # Azure OpenAI workspace
│   │   │   ├── ollama/           # Ollama portal workspace
│   │   │   ├── lmstudio/         # LM Studio workspace
│   │   │   ├── openrouter/       # OpenRouter workspace
│   │   │   ├── multimodal/       # Multimodal workspace
│   │   │   ├── kluster.ai/       # Kluster.ai workspace
│   │   │   └── vercel/           # Vercel portal workspace
│   │   ├── modules/              # Memory, emotion, cognition modules
│   │   ├── extensions/           # Platform integrations
│   │   ├── characters/           # Character definitions
│   │   ├── utils/                # Shared utilities
│   │   ├── types/                # TypeScript type definitions
│   │   ├── cli/                  # Command-line interface
│   │   └── __tests__/           # Test files
│   ├── data/                     # Runtime data storage
│   ├── scripts/                  # Build and utility scripts
│   ├── docs/                     # Agent documentation
│   └── dist/                     # Build output
├── website/                      # React web interface (Vite + Tailwind)
│   ├── src/
│   │   ├── components/           # React components
│   │   ├── pages/                # Page components
│   │   └── styles/               # Styling files
│   ├── public/                   # Static assets
│   ├── dist/                     # Build output
│   ├── storybook-static/         # Storybook build
│   └── .storybook/              # Storybook configuration
├── docs-site/                    # Documentation site (separate workspace)
├── redirect-package/             # NPM redirect package
├── testing/                      # Comprehensive test suites
├── monitoring/                   # System monitoring tools
├── config/                       # Configuration management
├── .github/                      # GitHub workflows
├── .claude/                      # Claude AI configuration
├── .gitmodules                   # Git submodules
├── docker-compose.yml            # Multi-service deployment
├── Dockerfile                    # Container configuration
└── package.json                  # Root workspace configuration
```

## Development Standards

### Code Quality
- **TypeScript Required**: All code must use TypeScript with strict mode
- **Bun Runtime**: Use Bun for package management and execution
- **Modular Design**: Components must be hot-swappable and independently testable
- **Event-Driven**: Use EventBus for inter-component communication

### Architecture Principles
- **Single Responsibility**: Each module has one clear purpose
- **Dependency Injection**: Use registry pattern for component management
- **Configuration-Driven**: All behavior configurable via JSON/YAML
- **Platform Agnostic**: Core logic independent of platform implementations

### Performance Requirements
- **Hot-Swapping**: Module replacement without system restart
- **Memory Efficient**: Vector operations optimized for large datasets
- **Real-time**: WebSocket support for live communication
- **Scalable**: Multi-agent coordination capabilities

### Testing Standards
- **Unit Tests**: Jest for component testing
- **Integration Tests**: Full agent lifecycle testing
- **Performance Tests**: Load testing for multi-agent scenarios
- **Platform Tests**: Extension compatibility verification

## Key Constraints

### Memory Management
- Vector embeddings must be chunked for efficiency
- Conversation history with configurable retention policies
- Memory provider abstraction for seamless switching

### AI Portal Integration
- Unified interface across all AI providers
- Error handling and fallback strategies
- Token management and cost optimization

### Platform Extensions
- Stateless design for platform integrations
- Webhook-based event handling where possible
- Platform-specific configuration isolation

### Character System
- JSON-based character definitions
- Personality trait inheritance and override
- Dynamic character switching capabilities

## Development Workflow

1. **Module Development**: Use registry pattern for new components
2. **Configuration**: Update relevant JSON/YAML configs
3. **Testing**: Comprehensive test coverage before integration
4. **Documentation**: Update docs-site with API changes
5. **Integration**: Hot-swap testing in development environment

## Related Rules and Deep Dives

### Core Development Standards
- @003-typescript-standards.mdc - TypeScript strict configuration and Bun optimization
- @004-architecture-patterns.mdc - Modular design principles and hot-swappable patterns
- @008-testing-and-quality-standards.mdc - Testing strategies for all component types

### Component-Specific Guidelines

#### AI Portal Development
- @005-ai-integration-patterns.mdc - AI provider integration standards and patterns
- @012-performance-optimization.mdc - Portal caching and performance optimization
- @010-security-and-authentication.mdc - API key management and secure connections

#### Memory System Development
- @011-data-management-patterns.mdc - Database schemas, migrations, and vector operations
- @012-performance-optimization.mdc - Vector embedding optimization and caching strategies
- @010-security-and-authentication.mdc - Data encryption and privacy protection

#### Extension System Development
- @007-extension-system-patterns.mdc - Platform integration patterns and webhook handling
- @015-configuration-management.mdc - Extension configuration and environment management
- @010-security-and-authentication.mdc - Platform authentication and security protocols

#### Web Interface Development
- @006-web-interface-patterns.mdc - React component patterns and Storybook integration
- @016-documentation-standards.mdc - Component documentation and API reference generation
- @012-performance-optimization.mdc - Frontend performance and bundle optimization

### Operations and Deployment
- @009-deployment-and-operations.mdc - Docker containerization and multi-service deployment
- @015-configuration-management.mdc - Environment configuration and secrets management
- @013-error-handling-logging.mdc - Centralized logging and error monitoring

### Advanced Automation (Cursor v1.2+ Features)
- @018-git-hooks.mdc - Automated commit validation and quality gates
- @019-background-agents.mdc - Parallel task automation and CI/CD integration
- @020-mcp-integration.mdc - External service integration and OAuth flows
- @021-advanced-context.mdc - Context-aware rule activation for different development phases
- @022-workflow-automation.mdc - Multi-agent coordination for complex development workflows

### Development Tools and CLI
- @014-cli-and-tooling-patterns.mdc - Command-line interface design and script automation
- @017-community-and-governance.mdc - Contribution guidelines and open source practices

This architecture serves as the foundation for all specialized component development. Always reference component-specific rules for detailed implementation guidance.

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
