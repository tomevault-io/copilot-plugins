## spartan-ai-studio

> This file serves as the primary instructional context for AI agents working on the Spartan Hub 2.0 project. It was generated following a deep audit of the repository on February 10, 2026.

# 🎖️ GEMINI.md - Spartan Hub 2.0 Instructional Context

This file serves as the primary instructional context for AI agents working on the Spartan Hub 2.0 project. It was generated following a deep audit of the repository on February 10, 2026.

## 📋 Project Overview

Spartan Hub 2.0 is a production-grade fitness coaching application integrating advanced AI for injury prediction, readiness forecasting, and an intelligent RAG (Retrieval-Augmented Generation) system.

**Key Technologies:**
- **Frontend**: React 19, TypeScript 5.9, Vite 7.1
- **Backend**: Express 4.18, TypeScript 5.9
- **AI/ML**: ONNX runtime for inference, Qdrant for vector storage
- **Database**: SQLite (via `better-sqlite3`) with PostgreSQL support
- **Testing**: Jest 30.2, ts-jest
- **Infrastructure**: Docker, Docker Compose

## ⚠️ CRITICAL: Codebase Structure & Status

The repository is structured as a **Documentation and Audit Hub**. The actual source code is managed via a Git submodule in the `spartan-hub/` directory.

**Current Environment Note:** 
As of the last audit, the `spartan-hub/` submodule appears **uninitialized or empty** in the current checkout. Ensure submodules are initialized to access source files:
```bash
git submodule update --init --recursive
```

### Expected Directory Layout (when initialized):
- `spartan-hub/` - Frontend root (Vite/React)
- `spartan-hub/backend/` - Backend root (Express API)
- `plans/` - Detailed project roadmaps and technical debt plans
- `backups/` - Database backup storage

## 🚀 Building and Running

Commands should be executed from their respective roots within the `spartan-hub` directory.

### Initialization
```bash
# In spartan-hub/ and spartan-hub/backend/
npm install
```

### Development
```bash
# From spartan-hub/ (starts both frontend and backend)
npm run dev
```

### Build
- **Frontend Build**: `npm run build:frontend` (from `spartan-hub/`)
- **Backend Build**: `npm run build:backend` (from `spartan-hub/`)
- **Complete Build**: `npm run build:all` (from `spartan-hub/`)

## 🧪 Testing and Quality

The project maintains a rigorous testing standard with over 244 test files and an 80% coverage target.

### Key Commands
- **Run All Tests**: `npm test`
- **Coverage Report**: `npm run test:coverage`
- **Security Tests**: `npm run test:security` (Backend)
- **Type Checking**: `npm run type-check` or `tsc --noEmit`

## 🛠️ Development Conventions

### 1. TypeScript Strict Mode (BLOCKING)
- No `any` type allowed (except in mocks).
- Explicit return types for all functions.
- Interfaces for objects, Types for unions.
- Kebab-case for file names (`component-name.tsx`).

### 2. Security Requirements (BLOCKING)
- **Sanitization**: All user inputs MUST be processed via `sanitizeInput()` or `sanitizeHtml()`.
- **API Security**: Rate limiting is mandatory on all routes.
- **Data**: Parameterized queries only (SQL injection prevention).
- **Auth**: JWT for protected routes, OAuth 2.0 for third-party sync.

### 3. Error Handling
- Use custom error classes (`ValidationError`, `NotFoundError`, etc.).
- Never swallow errors; always re-throw or log using `utils/logger.ts`.
- Early returns to minimize nesting.

### 4. Import Organization
1. External dependencies
2. Internal modules (relative)
3. Types/Interfaces
4. Constants

## 📈 Recent Audit Status (Feb 2026)

- **Architecture**: ✅ SOLID - Modular service-based structure.
- **RAG System**: ✅ Phase 7.4 Complete - Advanced query decomposition and re-ranking implemented.
- **Health**: 🟡 Technical debt present in build configs; TypeScript errors in `VideoCapture.test.tsx` require attention.
- **Database**: ✅ All 7 core migrations verified.

---
*For detailed agent guidelines, refer to `AGENTS.md` and `CODEBASE_STRUCTURE_ANALYSIS_2026.md`.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/121378-cell)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/121378-cell)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
