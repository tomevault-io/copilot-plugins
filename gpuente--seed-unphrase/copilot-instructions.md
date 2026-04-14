## seed-unphrase

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Architecture Overview

This is a **pnpm workspace monorepo** for a cryptocurrency seed phrase concealer tool with three main packages:

- **`@seed-unphrase/lib`** - Core cryptographic library using BigInt for precision, BIP39 wordlist validation, and salt-enhanced security
- **`@seed-unphrase/cli`** - Interactive command-line interface using inquirer and commander
- **`@seed-unphrase/web`** - React web application with Vite, Tailwind CSS, and Framer Motion animations

### Key Technologies
- **TypeScript** throughout all packages
- **Jest** for testing with comprehensive coverage
- **BigInt** arithmetic for handling up to 24-word seed phrases (97-digit precision)
- **SHA-256** for cryptographic salt transformations
- **BIP39** compliance for seed phrase validation

## Common Development Commands

### Building
```bash
pnpm build              # Build all packages (lib → cli → web)
pnpm build:lib          # Build library only
pnpm build:cli          # Build CLI only  
pnpm build:web          # Build web app only
```

### Development
```bash
pnpm dev:cli            # Run CLI in development mode with ts-node
pnpm dev:web            # Start Vite dev server for web app
```

### Testing
```bash
pnpm test               # Run all tests (lib + cli)
pnpm test:lib           # Run library tests only
pnpm test:cli           # Run CLI tests only
pnpm --filter "@seed-unphrase/lib" test:coverage  # Generate coverage report
```

### Type Checking
```bash
pnpm typecheck          # Type check all packages
```

### Single Package Commands
Use pnpm filters to work with specific packages:
```bash
pnpm --filter "@seed-unphrase/lib" <command>
pnpm --filter "@seed-unphrase/cli" <command>  
pnpm --filter "@seed-unphrase/web" <command>
```

## Build Dependencies

The packages have a strict build order dependency:
1. **lib** must be built first (core functionality)
2. **cli** depends on lib (uses workspace:* reference)
3. **web** depends on lib (uses workspace:* reference)

## Core Cryptographic Algorithm

The system uses a mathematical approach to conceal seed phrases:

1. **Word-to-Index**: Convert BIP39 words to 0-2047 indices
2. **Salt Enhancement**: Apply SHA-256 to transform indices (prevents brute force)
3. **Number Construction**: Format as "1" + 4-digit padded indices → BigInt
4. **Division**: Divide by cipher key → quotient:remainder pair
5. **Reconstruction**: Reverse process using same cipher key and salt

## Testing Strategy

- **Unit tests** for individual functions
- **End-to-end tests** for complete conceal/reveal cycles  
- **Backward compatibility tests** for algorithm stability
- **Edge case coverage** for various seed phrase lengths
- **CLI integration tests** for user interactions

## Security Considerations

This is an **educational tool** with the following security features:
- Salt-enhanced protection against brute force attacks
- BigInt precision to prevent floating-point errors
- BIP39 wordlist validation
- Cryptographically secure transformations

**Important**: Always verify builds pass and tests succeed before commits.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gpuente)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gpuente)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
