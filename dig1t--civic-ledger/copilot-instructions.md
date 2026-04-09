## civic-ledger

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CivicLedger is a secure document management system for US Government/Defense compliance (FedRAMP/NIST 800-53). Spring Boot 3.3.0 backend with PostgreSQL database and Next.js frontend.

## Commands

### Backend (Spring Boot)
```bash
# Start database
docker-compose up -d

# Build and run
mvn clean install
mvn spring-boot:run

# Tests
mvn test
mvn test -Dtest=ClassName
mvn test -Dtest=ClassName#methodName
```

### Frontend (Next.js)
```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install

# Development server
npm run dev

# Build for production
npm run build

# Run production build
npm start

# Linting
npm run lint
```

## Architecture

**Security-first design:**
- Files must be encrypted before storage (AES-256-GCM)
- Calculate SHA-256 hash for all documents
- Immutable audit logs for all operations
- Use `@Aspect` for audit interceptor on service layer

**RBAC roles:** ADMINISTRATOR, OFFICER (upload/read), AUDITOR (read logs only)

**Backend (Spring Boot):**
- Controllers: API layer only
- Services: Business logic
- Repositories: Data access

**Frontend (Next.js 14+):**
- App Router with TypeScript
- TailwindCSS for styling (high-contrast for accessibility)
- Server Components where applicable
- Section 508 / WCAG 2.1 AA compliance required

**Key entities to implement:**
- AuditLog: `action_type`, `user_id`, `timestamp`, `ip_address`, `resource_id` (write-once)
- Document: `file_hash`, `encryption_iv`, `version_number`, `classification_level`

See DEVELOPMENT_PLAN.md and DEV_CHECKLIST.md for implementation details.

## Task Management

**Always create TODO.md files for tasks inside the `docs/` directory.** When working on features or fixes:
- Create a TODO.md file in `docs/` to track task progress
- Break down complex tasks into actionable items
- Mark items as completed as you finish them

## Documentation

**Always keep documentation up to date.** When making changes to the codebase:
- Update relevant documentation in the `docs/` directory
- Document new features, APIs, and architectural decisions
- Keep API documentation in sync with implementation changes
- Update README and guides when workflows change

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dig1t)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dig1t)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
