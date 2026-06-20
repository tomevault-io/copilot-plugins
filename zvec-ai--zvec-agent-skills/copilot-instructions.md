## zvec-agent-skills

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Overview

This repository contains AI Agent skills for Zvec vector database. It provides complete technical guidance for building vector search applications with Zvec, supporting Python and Node.js development environments.

**Package Name**: `@zvec/agent-skills`

**GitHub**: https://github.com/zvec-ai/agent-skills

## Commands

```sh
# Install dependencies
bun install

# Build skills (compiles templates + code examples into skills/ output)
bun run build

# Pre-publish build (runs automatically before npm publish)
npm run prepublishOnly
```

## Architecture

### Build System

The build process (`scripts/build-skills.ts`) transforms source files into distributable skills:

1. **Source location**: `src/zvec/` contains templates, code examples, and general docs
2. **Output location**: `skills/zvec/` contains the built skill files
3. **Template processing**: Templates in `src/zvec/templates/*.md` contain `{{CODE:snippet-name}}` placeholders
4. **Code snippets**: Code files in `src/zvec/code/{typescript,python}/` define snippets with `@snippet:name` and `@end` markers
5. **Build output**: Each template generates two files: `skills/zvec/<topic>/python.md` and `typescript.md`

### Directory Structure

```
zvec-agent-skills/
├── src/zvec/                  # Source files
│   ├── SKILL.md               # Skill main description
│   ├── templates/             # Markdown templates with {{CODE:*}} placeholders
│   ├── code/
│   │   ├── typescript/        # .ts files with @snippet markers
│   │   └── python/            # .py files with @snippet markers
│   └── general/               # Language-agnostic docs
├── skills/zvec/               # Built output (generated, don't edit directly)
├── scripts/
│   └── build-skills.ts        # Build script
├── .github/workflows/         # CI/CD workflows
├── package.json               # Package config (@zvec/agent-skills)
└── README.md                  # User documentation
```

### Code Snippet Format

In TypeScript files:
```typescript
// @snippet:example-name
const code = "here";
// @end
```

In Python files:
```python
# @snippet:example-name
code = "here"
# @end
```

Reference in templates: `{{CODE:example-name}}`

## Adding a New Topic

1. Create a new template in `src/zvec/templates/<topic-name>.md`
2. Add code snippets to `src/zvec/code/python/examples.py` and `src/zvec/code/typescript/examples.ts`
3. Reference snippets in template using `{{CODE:snippet-name}}`
4. Run `bun run build` to generate output
5. Verify output in `skills/zvec/<topic-name>/`

## Release Process

1. Create a new Release on GitHub with tag (e.g., `v0.0.1`)
2. GitHub Actions automatically:
   - Extracts version from tag
   - Updates package.json version
   - Builds skills
   - Publishes to npm

## Code Style

- **Python**: PEP 8 style
- **TypeScript**: Strict mode with ESNext modules
- **Markdown**: Use clear headings and code blocks

## Important Notes

- `skills/zvec/` directory is auto-generated, don't edit directly
- Version in package.json is managed by CI (set to `0.0.0` as placeholder)
- License: Apache-2.0

---
> Source: [zvec-ai/zvec-agent-skills](https://github.com/zvec-ai/zvec-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-19 -->
