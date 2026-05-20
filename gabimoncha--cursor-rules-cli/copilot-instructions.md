## project-structure

> Project structure and file organization guidelines

# Cursor Rules CLI

A command-line tool for bootstrapping and maintaining **Cursor IDE** rule files in any project, offering interactive setup, auditing, and repository summarisation via Repomix.

## Purpose

Provide developers with an easy-to-use CLI that:
1. Installs default and community **Cursor rules** into the current repository.
2. Generates a compact XML snapshot of the codebase using **Repomix** so AI assistants get rich context.
3. Audits and sanitises existing rule files for potential security issues.
4. Offers quality-of-life features such as shell-completion, version checks, and update notifications.

## Key features

- 🚀 **Interactive init** – guided prompts to install rule templates and (optionally) run Repomix.
- 🧩 **Template library** – ships with default rules plus [awesome-cursorrules] templates.
- 🔍 **Audit & Scan** – detect and optionally fix vulnerable or malformed rule files.
- 🪄 **Repomix integration** – create `repomix-output.xml` & `repomix.config.json` with sensible defaults.
- 🖥 **Shell completion** – install/uninstall tab-completion for supported shells.
- 📦 **Workspaces ready** – published as a Bun package, compiled to ESM JavaScript.

## Directory structure

```tree
.
├── .github/                 # GitHub configuration and CI workflows
│   └── workflows/           # Continuous-integration definitions (tests, release)
├── awesome-cursorrules/     # Git submodule with a catalogue of community rule templates
├── cli/                     # Source for the published `@gabimoncha/cursor-rules` package
│   ├── bin/                 # Executable entry file distributed on npm
│   ├── src/                 # TypeScript source code
│   │   ├── audit/           # Regex & Unicode spoofing detection helpers
│   │   ├── cli/             # Command implementations (init, list, repomix, scan, …)
│   │   ├── core/            # Business-logic utilities shared across commands
│   │   ├── shared/          # Logger, constants, error handling, etc.
│   │   └── templates/       # Built-in rule templates copied during `init`
│   ├── package.json         # Manifest for the CLI workspace
│   └── README.md            # Package-level documentation
├── docs/                    # Detailed markdown documentation (guide, commands, contributing)
├── example/                 # Lightweight sample project used in tests & demos
│   ├── parent_folder/       # Nested example showcasing recursive scanning
│   └── single_folder/       # Alternative flat example structure
├── scripts/                 # Development helper scripts (template copy, vulnerability check)
├── .tool-versions           # Toolchain versions (e.g., Yarn)
├── FUTURE_ENHANCEMENTS.md   # Roadmap and upcoming improvements
├── LICENSE                  # MIT license
├── package.json             # Root workspace manifest (Yarn workspaces & scripts)
└── README.md                # High-level project overview and usage instructions
```

## Architecture

The CLI is published as an **ESM Bun package** and organised as a workspace inside a monorepo:

1. **Commander.js** powers the command parser (`cli/src/cli/cliRun.ts`).
2. Each command is implemented as an async function under `cli/src/cli/actions/*` and re-exports utilities from `core` and `shared`.
3. Business logic (rule installation, update checks, etc.) lives in `cli/src/core/*`.
4. **Repomix** is invoked programmatically to generate a compressed view of the repository.
5. Build pipeline compiles TypeScript → ESM JS via `tsc`, aliases paths, and copies markdown templates.
6. Automation (CI runs on Bun inside GitHub Actions) ensures tests & smoke checks pass for every push.

## Usage

```bash
# Install globally with Bun
bun add -g @gabimoncha/cursor-rules

# Initialise rules in the current project
cursor-rules init

# Generate Repomix snapshot only
cursor-rules repomix

# List installed rules
cursor-rules list
```

## Technical implementation

- **TypeScript 5** for type-safe source.
- **Bun** as runtime, package manager & test runner.
- **Commander.js** for CLI ergonomics.
- **@clack/prompts** for interactive terminal UX.
- **Repomix** for repository summarisation.
- **Picocolors** for colourful output.

## Future Enhancements

See `FUTURE_ENHANCEMENTS.md` for the full roadmap, including:
- Rule validation & linting
- Additional specialised rule templates (React, Python, Go, …)
- Web UI for rule management
- Deeper integration with code-analysis tools

---

*After saving this file you might need to refresh the Explorer sidebar or restart Cursor to see it in the tree.*

---
> Source: [gabimoncha/cursor-rules-cli](https://github.com/gabimoncha/cursor-rules-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
