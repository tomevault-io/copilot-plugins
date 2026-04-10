## bio-esign-elysiajs

> **Bio-eSign-ElysiaJs** is an enterprise-grade biometric attendance system designed for university-scale deployment (1,000+ students). It provides a robust API for managing users, recording biometric-verified attendance, and ensuring high-level security for sensitive biometric data.

# GEMINI.md - Bio-eSign-ElysiaJs Project Context

## Project Overview
**Bio-eSign-ElysiaJs** is an enterprise-grade biometric attendance system designed for university-scale deployment (1,000+ students). It provides a robust API for managing users, recording biometric-verified attendance, and ensuring high-level security for sensitive biometric data.

### Core Tech Stack
- **Runtime & Package Manager:** [Bun](https://bun.sh/)
- **Web Framework:** [ElysiaJS](https://elysiajs.com/) (with TypeScript)
- **ORM:** [Prisma](https://www.prisma.io/)
- **Database:** PostgreSQL
- **Authentication:** JWT (`@elysiajs/jwt`)
- **Security:** AES-256-GCM for biometric template encryption
- **Infrastructure:** Podman (Containerization)

---

## ⚠️ Mandatory Development Guidelines ⚠️

### 1. Tooling & Environment
- **STRICTLY NO DOCKER:** Always use `podman` and `podman-compose` instead of `docker` or `docker-compose`.
- **STRICTLY NO NPM/YARN/PNPM:** Always use `bun` for package management and script execution.
- **Node.js Compatibility:** While Elysia runs on Bun, it uses Node's `crypto` module for specific AES operations in `src/lib/crypto.ts`.

### 2. Biometric Security (Non-Negotiable)
- **Never store biometric templates in plain text.**
- All biometric `template64` data must be encrypted using `encryptTemplate` and decrypted using `decryptTemplate` from `src/lib/crypto.ts` before database operations.
- The encryption method is **AES-256-GCM**.

### 3. Database Operations
- **Singleton Pattern:** Always import the Prisma client from `@/lib/prisma`. Do NOT instantiate `new PrismaClient()` in other files.
- **Syncing:** Use `bunx prisma db push` for development syncs.

### 4. API Authentication
- All attendance, history, and user management routes MUST be protected by the JWT middleware.

---

## Building and Running

| Task | Command |
| :--- | :--- |
| **Start Database** | `podman-compose up -d` |
| **Stop Database** | `podman-compose down` |
| **Sync DB Schema** | `bunx prisma db push` |
| **Start Dev Server** | `bun run dev` |
| **Link Ideas Folder** | `bun run link-ideas` (Requires Admin for Symlinks) |
| **Generate Client** | `bunx prisma generate` |

---

## Project Structure
- `src/index.ts`: API entry point, route groups, and JWT middleware.
- `src/lib/prisma.ts`: Prisma Client singleton.
- `src/lib/crypto.ts`: AES-256-GCM encryption/decryption utilities.
- `prisma/schema.prisma`: Database schema definition (User, Biometric, Attendance models).
- `docs/ideas`: Junction link to an external ideas folder (linked via `bun run link-ideas`).

---

## Development Conventions
- **Naming:** Follow standard TypeScript camelCase for variables/functions and PascalCase for Classes/Interfaces.
- **Error Handling:** Leverage Elysia's built-in error handling and schema validation (`t.Object`).
- **Commits:** Prefer clear, concise commit messages focused on "why" changes were made.
- **Environment:** Ensure `JWT_SECRET`, `BIOMETRIC_ENCRYPTION_KEY`, and `DATABASE_URL` are set in your `.env` file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HazimMaulana)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HazimMaulana)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
