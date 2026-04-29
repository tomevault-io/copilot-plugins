## worktree-cli

> - src/index.ts: Main entry point that sets up the CLI commands and orchestrates the command handlers.


# Worktree CLI

## Key Files

- src/index.ts: Main entry point that sets up the CLI commands and orchestrates the command handlers.
- src/commands/new.ts: Implements logic to create new Git worktrees, with options for branch creation, dependency installation, and opening in an editor.
- src/commands/list.ts: Provides functionality to list existing Git worktrees in the repository.
- src/commands/remove.ts: Handles the removal of Git worktrees, including support for force deletion.
- package.json: Project configuration file detailing dependencies, scripts, and publishing settings.
- tsconfig.json: TypeScript configuration for compiling the codebase.
- .github/workflows/publish.yml: GitHub Actions workflow for building and publishing the package to npm.
- CHANGELOG.md: Logs version updates, bug fixes, and feature enhancements.

## Core Features

1. **Git Worktree Management**:
   - Create new worktrees with the option to automatically create and checkout new branches.
   - List existing worktrees associated with the repository.
   - Remove worktrees, with support for a force flag to delete directories regardless of state.

2. **Editor Integration**:
   - Automatically opens the new worktree in a specified editor (default is Cursor).
   - Allows customization of the editor command via CLI options.

3. **Dependency Installation**:
   - Optionally install project dependencies using a specified package manager during worktree creation.

4. **User-Friendly CLI**:
   - Intuitive command structure built using Commander.
   - Provides clear, colored console feedback for success and error messages.

## Main Components

- **CLI Interface**: Utilizes Commander for parsing commands and handling options.
- **Worktree Handlers**: Dedicated modules that implement the logic for new, list, and remove operations.
- **Process Execution**: Leverages Execa to execute Git commands and other external processes.
- **Configuration Management**: Governed by package.json and tsconfig.json, ensuring consistency across development and release workflows.

## Development Workflow

- **TypeScript-Based**: Code is written in TypeScript and compiled using tsc.
- **Watch Mode**: Supports development with a watch mode for real-time compilation (via the "dev" script).
- **CI/CD Pipeline**: Configured with GitHub Actions to automate builds and publishing using semantic-release.
- **Semantic Versioning**: Version management is automated with semantic-release, ensuring clear and consistent version updates.

---

ow Releases Work Now:
From now on, when you push commits with messages like fix: ..., feat: ..., or include BREAKING CHANGE: in the footer to the main branch, the publish.yml workflow will run.
semantic-release will analyze these commits, determine the correct version bump, update package.json (only within the CI environment for the release process, it won't commit this back with the @semantic-release/npm plugin), generate release notes, create a Git tag (e.g., v2.1.0), publish the package with the correct version to npm (including the dist directory specified in files), and create a corresponding GitHub Release.
Your original issue of the dist directory not being included should be resolved with the next release triggered by this new setup, as semantic-release will use the package.json which now explicitly includes "files": ["dist"]. After the next successful release triggered by a feature or fix commit, you should be able to install the CLI globally using pnpm i -g @johnlindquist/worktree.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/johnlindquist) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
