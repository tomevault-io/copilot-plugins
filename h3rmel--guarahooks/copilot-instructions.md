## pnpm-only

> Enforcing PNPM as the package manager


# Mandatory PNPM Usage

This project **MANDATORILY** uses PNPM as package manager. Using `npm` or `yarn` is **FORBIDDEN**.

## Security Configuration

The project has configurations that prevent the use of other managers:

### [.npmrc](mdc:.npmrc)

```txt
engine-strict=true
```

### [package.json](mdc:package.json) - Engines

```json
{
  "engines": {
    "node": ">=18",
    "pnpm": ">=9.15.3"
  }
}
```

### Verification Script

```json
{
  "scripts": {
    "preinstall": "node ./scripts/check-node.mjs && node ./scripts/check-pnpm.mjs && pnpm dlx only-allow pnpm"
  }
}
```

## Correct Commands

### ✅ Dependency Installation

```bash
# Install all dependencies
pnpm install

# Install specific dependency
pnpm add package-name

# Install development dependency
pnpm add -D package-name

# Install global dependency
pnpm add -g package-name
```

### ✅ Run Scripts

```bash
# Development mode
pnpm dev

# Build project
pnpm build

# Linting
pnpm lint
pnpm lint:fix

# Formatting
pnpm format:write
pnpm format:check
```

### ✅ Workspace (Monorepo)

```bash
# Install dependency in specific workspace
pnpm add package-name --filter @guarahooks/cli

# Run script in specific workspace
pnpm --filter @guarahooks/cli build

# Run command in all workspaces
pnpm -r build
```

## ❌ FORBIDDEN Commands

### DO NOT use npm

```bash
# ❌ NEVER DO THIS
npm install
npm run dev
npm add package-name
```

### DO NOT use yarn

```bash
# ❌ NEVER DO THIS  
yarn install
yarn dev
yarn add package-name
```

## Automatic Verifications

The project has automatic scripts that verify:

1. **[scripts/check-node.mjs](mdc:scripts/check-node.mjs)**: Minimum Node.js version
2. **[scripts/check-pnpm.mjs](mdc:scripts/check-pnpm.mjs)**: Minimum PNPM version
3. **only-allow pnpm**: Prevents npm/yarn usage

## PNPM Benefits

- 🚀 **Performance**: Faster installation
- 💾 **Space**: Package sharing between projects
- 🔒 **Security**: Symbolic links avoid conflicts
- 📦 **Monorepo**: Native workspace support

## Troubleshooting

### If you get npm/yarn errors

```bash
# Clean node_modules and old lockfiles
rm -rf node_modules package-lock.json yarn.lock

# Reinstall with pnpm
pnpm install
```

### If you don't have pnpm installed

```bash
# Install pnpm globally
npm install -g pnpm@latest

# Or via corepack (recommended)
corepack enable
corepack prepare pnpm@latest --activate
```

---
> Source: [h3rmel/guarahooks](https://github.com/h3rmel/guarahooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
