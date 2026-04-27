## financial-tracker-frontend

> This repository is part of a financial tracker, this repo handles the frontend

# Project Overview

This repository is part of a financial tracker, this repo handles the frontend

## Technology overview

- Node 24
- Next.js 16
- Shadcn/ui 4

## Documentation

The documentation resides on @docs

# Important: Commit Policy

1. Commit Frequency  
   Create commits at logical milestones (data model, API endpoint, UI component/group, page/route, refactor).  
   Commits must represent **small, atomic changes**. Avoid mixing unrelated work.

2. Commit Quality  
   Commits must:

- build/compile successfully
- not introduce failing tests
- include only related changes

3. Commit Messages  
   Messages must be **short and descriptive**.

Format:
<type>: short description

Examples:
feat: add transaction data model  
feat: implement create transaction endpoint  
fix: correct balance calculation  
refactor: simplify transaction service

4. Branch Strategy  
   Two branch levels for feature development:

Module branch  
feature/<module-name>  
Example: feature/mvp-module-1

Task branch  
feature/<module-name>-<task>  
Examples:  
feature/mvp-module-1-data-model  
feature/mvp-module-1-create-endpoint

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SocarComunica) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
