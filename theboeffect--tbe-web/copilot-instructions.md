## tbe-web

> This project uses Yarn as its package manager. All package management operations

# Rule: Package Manager
description: >
  This project uses Yarn as its package manager. All package management operations
  must be performed using Yarn to maintain consistency and avoid conflicts.

do:
  must:
    - "Use yarn for all package installations: yarn add [package]"
    - "Use yarn for all development package installations: yarn add -D [package]"
    - "Use yarn for running scripts: yarn [script-name]"
    - "Commit yarn.lock file changes"
    - "Use yarn workspaces for monorepo management if needed"
  
  avoid:
    - "NEVER use npm commands (npm install, npm run, etc.)"
    - "NEVER commit package-lock.json"
    - "NEVER mix yarn and npm in the same project"
    - "NEVER delete yarn.lock unless absolutely necessary"

example:
  good: |
    yarn add package-name
    yarn remove package-name
    yarn install
    yarn build
  bad: |
    npm install package-name
    npm run build
    npm ci

conventions:
  versioning:
    - "Use exact versions in package.json when needed (no ^ or ~)"
    - "Let yarn.lock manage sub-dependency versions"
  
  scripts:
    - "Define all project scripts in package.json"
    - "Use yarn run for non-standard script names"

  dependencies:
    - "Use yarn add for production dependencies"
    - "Use yarn add -D for development dependencies"
    - "Use yarn workspace commands for monorepo operations"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/theBoEffect) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
