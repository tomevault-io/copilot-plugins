## project-overview

> This is a **professional TypeScript CLI** with modular architecture supporting multiple email marketing platforms.

# SmartLead CLI - Project Overview

## 🏗️ Architecture

This is a **professional TypeScript CLI** with modular architecture supporting multiple email marketing platforms.

### 📁 Core Structure
- **[src/core/index.ts](mdc:src/core/index.ts)** - Main CLI entry point with module selector
- **[src/core/module-selector.ts](mdc:src/core/module-selector.ts)** - Interactive module switching system
- **[src/core/utils/config.ts](mdc:src/core/utils/config.ts)** - Configuration management (API keys, modules)
- **[src/core/utils/theme.ts](mdc:src/core/utils/theme.ts)** - Theme system with brand colors

### 🧩 Module System
- **[src/modules/smartlead/](mdc:src/modules/smartlead/)** - SmartLead module (80+ commands, available)
- **[src/modules/instantly/index.ts](mdc:src/modules/instantly/index.ts)** - Instantly module (placeholder, coming Q2 2024)
- **[src/types/global.ts](mdc:src/types/global.ts)** - Global TypeScript interfaces and types
- **[src/modules/smartlead/types.ts](mdc:src/modules/smartlead/types.ts)** - SmartLead-specific types

### 🧪 Testing & Quality
- **[tests/](mdc:tests/)** - Jest test suites with 90%+ coverage
- **[jest.config.js](mdc:jest.config.js)** - Jest configuration with TypeScript support
- **[tests/setup.ts](mdc:tests/setup.ts)** - Test environment setup and utilities

### 📦 Build & Distribution
- **[tsconfig.json](mdc:tsconfig.json)** - TypeScript configuration with strict rules
- **[scripts/build.sh](mdc:scripts/build.sh)** - Automated build script
- **[scripts/install.sh](mdc:scripts/install.sh)** - Professional installer with validation
- **[package.json](mdc:package.json)** - npm configuration with comprehensive scripts

### 📚 Documentation
- **[docs/README.md](mdc:docs/README.md)** - Comprehensive project documentation
- **[docs/CONTRIBUTING.md](mdc:docs/CONTRIBUTING.md)** - Development guidelines
- **[docs/ROADMAP.md](mdc:docs/ROADMAP.md)** - Feature roadmap and project direction
- **[docs/CHANGELOG.md](mdc:docs/CHANGELOG.md)** - Version history and changes

## 🎯 Key Concepts

### Module System
Each module implements the `CLIModule` interface and provides:
- Platform-specific commands and functionality
- Branded themes and user experience
- Independent configuration and API management

### Theme System
Supports multiple branded themes:
- **SmartLead**: Professional blue gradient (#2563eb → #0ea5e9 → #06b6d4)
- **Instantly**: Purple gradient (#7c3aed → #a855f7 → #ec4899)

### Configuration
- Global config: `~/.smartlead-cli/config.json`
- Module-specific: `~/.smartlead-cli/{module}.json`
- Environment variables: `SMARTLEAD_API_KEY`, `SMARTLEAD_BASE_URL`

## 🚀 Getting Started

1. **Development**: `npm run dev` - TypeScript watch mode
2. **Build**: `npm run build` - Compile to JavaScript
3. **Test**: `npm test` - Run Jest test suite
4. **Install**: `npm run install-global` - Install globally

---
> Source: [LeadMagic/cold-email-cli](https://github.com/LeadMagic/cold-email-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
