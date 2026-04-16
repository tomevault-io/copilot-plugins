## ai-flow

> **IMPORTANT:** Read `AGENT.md` first for complete project context.

# Cursor Configuration for AI Flow

## 📋 Quick Start

**IMPORTANT:** Read `AGENT.md` first for complete project context.

This file contains Cursor-specific preferences and workflows for the ai-flow project.

---

## 🎯 Project Summary

**Name:** AI Flow
**Description:** Interactive CLI tool to build AI-ready projects with comprehensive documentation
**Stack:** Node.js 18+ + TypeScript + Commander.js + Inquirer
**Version:** 2.0.0

---

## 📚 Documentation

Read these files in order:

1. `AGENT.md` - Universal AI configuration (⭐ START HERE)
2. `CLAUDE.md` - Detailed project architecture and development guide
3. `.github/copilot-instructions.md` - GitHub Copilot playbook
4. `README.md` - User-facing documentation
5. `CONTRIBUTING.md` - Contribution guidelines
6. `package.json` - Dependencies and scripts
7. `src/cli.ts` - Main CLI implementation

---

## ❌ NEVER

- Hardcode secrets or API keys
- Use `any` types without explicit justification
- Skip reading files before editing (Read → Edit, never Write over existing)
- Modify templates without understanding their purpose (they generate docs for users)
- Break the Conventional Commits format
- Commit without running tests (`npm test`)
- Use ESM-only packages in this CommonJS project
- Skip TypeScript strict mode checks

---

## ✅ ALWAYS

- Read `AGENT.md` first for complete context
- Read files before editing (use Read → Edit pattern)
- Follow TypeScript strict mode
- Use explicit return types for functions
- Use interfaces for object shapes
- Run `npm run build` before committing
- Run `npm test` before committing
- Use Conventional Commits format: `<type>(<scope>): <subject>`
- Preserve `{{PLACEHOLDER}}` tokens in templates (downstream AI agents expand them)
- Test CLI changes with: `npm run dev init test-folder --ai cursor`
- Use `chalk` for consistent logging (info=cyan/white, warnings=yellow, failures=red)
- Validate cross-platform behavior (Windows keeps script perms untouched)

---

## 🏗️ Architecture

**Pattern:** Single-file CLI with modular helper functions
**Organization:**

- `src/cli.ts` - All CLI logic (Commander.js + Inquirer)
- `templates/` - Document templates (for user projects, not this project)
- `prompts/` - Build prompts (for user projects)
- `dist/` - Compiled output (CommonJS)
- `.github/prompts/` - Development workflow commands

**Module System:** CommonJS (`module: "commonjs"` in tsconfig.json)
**Path Resolution:** Uses `__dirname` relative to `dist/cli.js` to copy assets from installed package

---

## 🎨 Code Style

- **Indentation:** 2 spaces
- **Quotes:** Single quotes for strings
- **Formatter:** None configured (consider adding Prettier)
- **Linter:** ESLint configured (`npm run lint`)
- **File naming:** `kebab-case.ts`
- **Function naming:** `camelCase`
- **Constants:** `UPPER_SNAKE_CASE`
- **TypeScript:** Strict mode enabled

---

## 🧪 Testing

- **Coverage:** Tests in `__tests__/` directory
- **Framework:** Jest
- **Command:** `npm test`
- **Before commit:** Always run `npm test` and `npm run build`

---

## 📝 Commits

Use Conventional Commits format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `ci`, `perf`, `revert`
**Scopes:** `cli`, `templates`, `prompts`, `commands`, `docs`, `deps`, `config`, `readme`

**Examples:**

- `feat(cli): add frontend project type support`
- `fix(templates): correct placeholder syntax`
- `docs(readme): update installation instructions`
- `chore(deps): upgrade chalk to v5.6.2`

**Rules:**

- Imperative mood ("add feature" not "added feature")
- Lowercase start
- No period at end
- Max 72 characters for subject line
- English only

---

## 🔒 Security

- Never hardcode secrets or API keys
- Use environment variables for sensitive data
- Validate all user input
- Keep dependencies updated (`npm audit`)
- Check for hardcoded secrets before committing

---

## 💡 Cursor Tips

- Use `@AGENT.md` to reference project context
- Use `@CLAUDE.md` for detailed architecture
- Use `@src/cli.ts` for CLI implementation details
- Use `@package.json` for dependencies and scripts
- Run `/flow1.commit` for automated conventional commits
- Run `/flow2.check` for pre-PR quality checks
- Run `/flow3.docs` for documentation gap analysis and auto-update
- Run `/flow4.release` for version publishing

---

## 🚀 Development Workflow

### Before Making Changes

1. Read `AGENT.md` for project context
2. Read `CLAUDE.md` for architecture details
3. Understand the feature/bug request
4. Review affected code areas

### While Coding

1. Follow TypeScript strict mode
2. Add JSDoc comments for exported functions
3. Test locally with `npm run dev`
4. Check compilation with `npm run build`
5. Run linter: `npm run lint`

### Testing Changes

**For CLI Changes:**

```bash
npm run build
npm run dev init test-folder --ai cursor
cd test-folder
ls -la .ai-flow
ls -la .cursor/commands
cd ..
rm -rf test-folder
```

**For Template Changes:**

1. Make changes to template files
2. Rebuild: `npm run build`
3. Initialize in test project
4. Verify generated documents

**For Prompt Changes:**

1. Edit `prompts/backend/flow-build.md` or phase files
2. Rebuild and initialize test project
3. Run through full questionnaire flow

### Before Committing

```bash
npm run build
npm test
npm run lint
```

---

## 🔍 Key Implementation Details

**Path Resolution:**

- CLI installed globally but needs to copy files from package
- Uses `__dirname` relative to `dist/cli.js`
- `ROOT_DIR = path.resolve(__dirname, "..")`

**File Operations:**

- Uses `fs-extra` library
- `fs.pathExists()` - Check existence
- `fs.ensureDir()` - Create directories recursively
- `fs.copy()` - Copy files/folders
- `fs.writeJSON()` - Write formatted JSON

**AI Tool Mapping:**

- `cursor` → `.cursor/commands/`
- Commands copied from `prompts/backend/` to `.cursor/commands/`

**Config Structure:**

- Location: `.ai-flow/core/config.json`
- Keys: `version`, `aiTools`, `createdAt`, `projectType`, `backend`, `frontend`

---

**Always reference `AGENT.md` for complete project understanding.**

**Generated for:** AI Flow development
**Last Updated:** 2025-01-XX

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/victorvelazquez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
