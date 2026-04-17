## convex-ts

> **convex.ts** is the official TypeScript/JavaScript client library for the [Convex decentralized lattice network](https://convex.world). This is a monorepo containing client libraries and demo applications for interacting with the Convex DLT platform.

# Claude Code Guidelines for convex.ts

## Project Overview

**convex.ts** is the official TypeScript/JavaScript client library for the [Convex decentralized lattice network](https://convex.world). This is a monorepo containing client libraries and demo applications for interacting with the Convex DLT platform.

### Vision

Convex is building fair, inclusive, efficient, and sustainable economic systems based on decentralized technology. The convex.ts library provides developers with idiomatic TypeScript/JavaScript APIs to interact with the Convex network - creating accounts, submitting transactions, querying state, and managing cryptographic keys.

---

## Repository Structure

This is a **pnpm workspace monorepo** containing multiple packages:

```
convex.ts/
├── packages/
│   ├── convex-client/        # Main TypeScript client (@convex-world/convex-ts)
│   ├── convex-react/         # React hooks and components
│   └── demo-site/            # Next.js demo application
├── CLAUDE.md                 # This file
├── README.md                 # User-facing documentation
├── DEPLOY.md                 # Publishing guide
├── package.json              # Monorepo root configuration
└── pnpm-workspace.yaml       # Workspace configuration
```

### Package Details

| Package | npm Name | Purpose | Status |
|---------|----------|---------|--------|
| `convex-client` | `@convex-world/convex-ts` | Core client library | Ready for npm release |
| `convex-react` | `@convex-world/convex-react` | React integration | Development |
| `demo-site` | `@convex-world/demo-site` | Demo application | Development |

---

## Key Technologies

- **TypeScript 5.x** - Type-safe development
- **pnpm** - Fast, disk-efficient package manager with workspace support
- **Node.js 18+** - Runtime environment
- **axios** - HTTP client for peer communication
- **@noble/ed25519** - Ed25519 cryptographic signatures
- **@noble/hashes** - Cryptographic hashing
- **Jest** - Testing framework
- **Next.js 16** - Demo site framework (demo-site package)
- **React 18/19** - UI library (convex-react, demo-site)

---

## Development Setup

### Prerequisites

- **Node.js 18+** (20+ recommended)
- **pnpm 8+** (enable with `corepack enable`)
- **Git**
- **Docker Desktop** (for running local Convex peer during testing)

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/Convex-Dev/convex.ts.git
cd convex.ts

# Install dependencies (all packages)
pnpm install

# Build all packages
pnpm build
```

### Package Management

```bash
# Install dependency in specific package
pnpm --filter @convex-world/convex-ts add <package>

# Run command in specific package
pnpm --filter @convex-world/convex-ts <command>

# Run command in all packages
pnpm -r <command>
```

---

## Building

### Build All Packages

```bash
pnpm build
```

### Build Specific Package

```bash
# Build convex-client
pnpm --filter @convex-world/convex-ts build

# Build convex-react
pnpm --filter @convex-world/convex-react build

# Build demo-site
pnpm --filter @convex-world/demo-site build
```

### Clean Build

```bash
# Clean and rebuild convex-client
cd packages/convex-client
rm -rf dist
pnpm build
```

---

## Testing

Tests require a local Convex peer running in Docker.

### Start Local Convex Peer

```bash
# Start peer
docker-compose up -d

# Verify peer is running
curl http://localhost:8080/api/v1/status

# Stop peer
docker-compose down
```

### Run Tests

```bash
# Run all tests (after starting peer)
pnpm test

# Run tests for specific package
pnpm --filter @convex-world/convex-ts test

# Run with custom peer URL
CONVEX_PEER_URL=http://localhost:8080 pnpm test
```

---

## Package-Specific Guidelines

### convex-client (@convex-world/convex-ts)

**Location:** `packages/convex-client/`

The main client library. This is the primary package for npm publishing.

#### Source Structure

```
src/
├── index.ts          # Main exports
├── convex.ts         # Main Convex class
├── types.ts          # TypeScript interfaces
├── crypto.ts         # Cryptographic utilities
├── keystore.ts       # Key management
├── identicon.ts      # Address identicons
└── __tests__/        # Jest tests (excluded from build)
```

#### Key Conventions

- **Immutable by default** - Never mutate parameters or internal state
- **Async/await** - All network operations are async
- **Error handling** - Throw descriptive errors, wrap axios errors
- **TypeScript strict mode** - All code must pass strict type checking
- **ESM only** - Use `.js` extensions in imports (NodeNext module resolution)

#### Building

```bash
cd packages/convex-client
pnpm build  # Compiles src/ -> dist/
```

**Important:** The `tsconfig.json` excludes `__tests__/` to prevent test files from being compiled into `dist/`.

#### What Gets Published

Only these files are included in the npm package (see `package.json` `files` field):
- `dist/` - Compiled JavaScript and TypeScript definitions
- `CHANGELOG.md` - Version history
- `package.json` - Package metadata

Test files, source TypeScript, and development files are excluded.

### convex-react (@convex-world/convex-react)

**Location:** `packages/convex-react/`

React hooks and components for Convex integration. Depends on `@convex-world/convex-ts`.

**Status:** In development, not yet ready for publishing.

### demo-site (@convex-world/demo-site)

**Location:** `packages/demo-site/`

Next.js application demonstrating convex.ts usage.

#### Running the Demo

```bash
# Development mode
pnpm demo
# or
cd packages/demo-site
pnpm dev
```

**URL:** http://localhost:3000

---

## Code Style and Conventions

### General Principles

1. **Read before modifying** - Always read existing code before making changes
2. **Prefer editing over creating** - Edit existing files rather than creating new ones
3. **Follow existing patterns** - Match the style and structure of existing code
4. **TypeScript strict mode** - All code must satisfy strict type checking
5. **No over-engineering** - Keep solutions simple and focused on requirements

### TypeScript Conventions

- Use **interfaces** for public API types
- Use **type** for unions, mapped types, and utility types
- Always export types used in public API
- Use `.js` extension in imports (required for ESM with NodeNext)

### Naming Conventions

- **Classes:** PascalCase (e.g., `Convex`, `KeyPair`)
- **Functions/methods:** camelCase (e.g., `createAccount`, `submitTransaction`)
- **Interfaces:** PascalCase (e.g., `AccountInfo`, `Transaction`)
- **Constants:** UPPER_SNAKE_CASE for true constants
- **Files:** kebab-case or camelCase (match existing pattern)

### Import Style

```typescript
// External dependencies first
import axios from 'axios';
import { sha256 } from '@noble/hashes/sha256';

// Internal imports with .js extension
import { KeyPair } from './types.js';
import { generateKeyPair } from './crypto.js';
```

---

## Publishing Workflow

See `DEPLOY.md` for detailed publishing instructions.

### Pre-Publishing Checklist

- [ ] All tests pass: `pnpm test`
- [ ] Version bumped in `package.json` (follow semver)
- [ ] `CHANGELOG.md` updated with version and changes
- [ ] Clean build: `rm -rf dist && pnpm build`
- [ ] Verify package contents: `pnpm pack` (check tarball output)
- [ ] All changes committed to git
- [ ] On correct branch (`master`)

### Publishing Command

```bash
cd packages/convex-client
pnpm publish --access=public
```

### Post-Publishing

```bash
# Tag the release
git tag -a v0.1.0 -m "Release v0.1.0"
git push origin master --tags

# Create GitHub release
# https://github.com/Convex-Dev/convex.ts/releases/new
```

---

## Common Tasks

### Add a New Method to Convex Class

1. Add the method to `src/convex.ts`
2. Add type definitions to `src/types.ts` if needed
3. Export types from `src/index.ts` if public
4. Add tests to `src/__tests__/client.test.ts`
5. Update `CHANGELOG.md` under `[Unreleased]`
6. Run tests: `pnpm test`
7. Build: `pnpm build`

### Update Dependencies

```bash
# Check for updates
pnpm outdated

# Update all packages
pnpm update

# Update specific package
pnpm update <package-name>

# Check for security issues
pnpm audit
pnpm audit fix
```

### Debugging Tests

```bash
# Run tests with verbose output
pnpm test --verbose

# Run specific test file
pnpm test client.test

# Run with Node debugging
node --inspect-brk ./node_modules/jest/bin/jest.js --runInBand
```

---

## Important Notes

### Module Resolution

This project uses **NodeNext** module resolution, which requires:
- `.js` extensions in imports (even for `.ts` files)
- `"type": "module"` in package.json
- Proper ESM setup

### Windows Development

- Use `pushd` instead of `cd` for directory changes in bash
- Git line endings: use `core.autocrlf=false` to preserve LF
- Docker Desktop must be running for tests

### Monorepo Workspace Dependencies

The `convex-react` package depends on `convex-client` via workspace protocol:

```json
{
  "dependencies": {
    "@convex-world/convex-ts": "workspace:*"
  }
}
```

When publishing, pnpm automatically converts this to a specific version.

---

## Related Resources

- **Main Repository:** https://github.com/Convex-Dev/convex.ts
- **Convex Network:** https://github.com/Convex-Dev/convex
- **Convex Docs:** https://docs.convex.world
- **npm Package:** https://www.npmjs.com/package/@convex-world/convex-ts
- **Discord:** https://discord.com/invite/xfYGq4CT7v

---

## Branch Strategy

- **`master`** - Main branch, production-ready code
- Feature branches as needed

Keep `master` stable and always buildable.

---

## Getting Help

- Check `README.md` for user documentation
- Check `DEPLOY.md` for publishing guide
- Check existing tests for usage examples
- Check Convex docs for network protocol details

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Convex-Dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
