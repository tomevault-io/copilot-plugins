## ampext

> - **SEARCH BEFORE CREATING**: Use `dir` or `Get-ChildItem` to find similar files

# AMP CODE REVIEW RULES

## 1. CHECK EXISTING FILES FIRST
- **SEARCH BEFORE CREATING**: Use `dir` or `Get-ChildItem` to find similar files
- **CHECK ALL DIRECTORIES**: `ext/`, `desktop-ui/`, `website/`, root
- **EXTEND, DON'T DUPLICATE**: Find and modify existing implementations
- **USE grep/findstr**: Search for function names and patterns before writing new code

## 2. FOLLOW EXISTING PATTERNS
- **STUDY THE CODEBASE**: Look at how similar problems were solved
- **CHECK git history**: `git log --oneline -10` for recent changes
- **MATCH EXISTING STYLE**: Same indentation, naming, structure
- **USE EXISTING CONSTANTS**: Don't reinvent configuration patterns

## 3. FILE ORGANIZATION
- **ext/**: Extension files (manifest, background, content, popup)
- **desktop-ui/**: Electron app (main.js, renderer.js, index.html)
- **website/**: Product website files
- **Root**: Native messaging manifests, batch files, configs

## 4. NEVER CREATE UNNECESSARY FILES
- **ONE SOLUTION**: Don't create multiple versions or test files
- **CLEAN UP**: Delete temporary files after use
- **NO DUPLICATES**: One batch file, one config, one implementation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AMPLIFAIUI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
