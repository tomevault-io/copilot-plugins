## violet-vault

> **v2.0 Baseline Complete** - Full TypeScript conversion, Unified Data Model (Envelopes + Transactions), React 19, and modern glassmorphic UI.

# 🌌 Antigravity Config (GEMINI.md)

## 🎯 Current Mission

**v2.0 Baseline Complete** - Full TypeScript conversion, Unified Data Model (Envelopes + Transactions), React 19, and modern glassmorphic UI.

**Next Target**: v2.1 Sentinel Share, Dashboard & Marketing - Cross-app transaction matching and enhanced analytics.

## 🏗️ Architecture Stack (v2.0)

- **Frontend**: React 19 + TypeScript (strict mode) + Vite 7
- **Local DB**: Dexie.js (Offline-first, E2EE via IndexedDB)
- **Cloud DB/Auth**: Firebase Firestore + Auth
- **State Management**:
  - TanStack Query v5 → Server state (Firebase, API)
  - React Context → Auth state (user, session)
  - Zustand → UI state only (modals, forms)
- **Backend (Go)**: `/api/bug-report.go` (GitHub API proxy, secrets handling)
- **Backend (Python)**: `/api/analytics.py` (Financial intelligence, predictions)

## 📜 User Rules

- Create GH issues for bugs (label: `bug`) and features (label: `enhancement`, `roadmap`)
- Commit changes frequently

- **Mandatory 80%+ test coverage for all new/modified code (Front & Back)**
- **Always verify existence of corresponding tests for any files touched; if they don't exist, create them.**
- Data Flow: Firebase → Dexie → TanStack Query
- State: React Context (Auth) / Zustand (UI only)
- **TypeScript strict** - Zero `any` types allowed
- **@ imports only** - Always use `@/` path aliases, never relative imports

## 🔄 Active Task

Consolidating Sync Services and preparing the Polyglot backend structure.

- **Milestone**: v2.1 (Target: March 2026)
- **Epic**: #1463

## ⚠️ Critical Rules

- **E2EE**: Core budgeting data MUST stay client-side encrypted
- **No server data in Zustand**: Use TanStack Query for all Firebase/API data
- **Auth in Context**: Auth state lives in React Context, not Zustand (v2.0+)
- **Test coverage**: All PRs MUST include tests (80%+ for new files)

## 🛠️ Tooling

Use `full_salvo.sh` for multi-language verification.

## 📚 Key Documentation

- [Agent.md](./Agent.md) - Detailed agent instructions
- [.github/agents/violet-vault.yml](./.github/agents/violet-vault.yml) - GitHub Copilot agent config
- [docs/DOCUMENTATION_INDEX.md](./docs/DOCUMENTATION_INDEX.md) - Complete docs index
- [docs/guides/MILESTONES.md](./docs/guides/MILESTONES.md) - 2026 roadmap

---

Last Updated: January 19, 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thef4tdaddy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/thef4tdaddy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
