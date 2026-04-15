## frey

> This project uses organized cursor rules stored in the `.cursor/` directory.

# Frey Project - Cursor Rules

This project uses organized cursor rules stored in the `.cursor/` directory.

## Available Rule Files

### Core Framework Rules
- **`.cursor/frey-framework.md`** - Comprehensive Frey framework development guidelines
- **`.cursor/typescript-patterns.md`** - Advanced TypeScript patterns and best practices
- **`.cursor/testing-guidelines.md`** - Multi-runtime testing strategies and patterns
- **`.cursor/deployment-rules.md`** - CI/CD, security, and deployment procedures

## Quick Reference
- **Architecture**: Entity-driven API generation with Fastify
- **Runtimes**: Node.js + Bun compatibility (3x faster with Bun)
- **Testing**: Tests across unit and integration
- **Code Style**: Prettier + TypeScript strict mode
- **File Structure**: Modular CRUD operations in separate files
- **Security**: Zero tolerance for vulnerabilities

## Development Commands
```bash
# Testing
npm run test:run    # Run tests with Vitest (Node.js)
bun test           # Run tests with Bun (3x faster)
npm run test:coverage # Run with coverage reporting

# Code Quality
npm run format:fix   # Format code with Prettier
npm run type-check # TypeScript strict mode check
npm run audit      # Security vulnerability check

# Build & Deploy
npm run build      # TypeScript compilation
npm run build:dist # Production build with declarations
```

## Quality Gates
- ✅ All tests pass on both Node.js and Bun
- ✅ Prettier formatting consistent
- ✅ TypeScript strict mode passes
- ✅ Zero security vulnerabilities
- ✅ Multi-runtime compatibility verified

## Rule File Guidelines
Each rule file focuses on a specific aspect:
- **Framework rules**: Core architecture and patterns
- **TypeScript patterns**: Advanced type safety and ES modules
- **Testing guidelines**: Multi-runtime testing strategies
- **Deployment rules**: CI/CD, security, and release procedures

## Adding New Rules
Create new `.md` files in `.cursor/` directory for additional topics:
- `.cursor/phase2-features.md` - Authentication, validation, caching
- `.cursor/performance-optimization.md` - Performance tuning guidelines
- `.cursor/api-design.md` - REST API design patterns
- etc.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ked57) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
