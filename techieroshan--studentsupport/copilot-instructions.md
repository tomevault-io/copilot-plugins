## studentsupport

> @.cursorrules - Non-Negotiable Quality & Code Mandate

@.cursorrules - Non-Negotiable Quality & Code Mandate

This document outlines the minimum required quality standards for all submitted code, scripts, and configurations. Adherence to these rules is strictly mandatory.

I. Non-Negotiable Quality Gate (Applies to ALL Files)

All code must pass a pre-deployment dry run with the following outcomes:

Zero TSC Errors: All generated TypeScript code must be strictly type-safe and pass tsc --noEmit without any errors. No implicitly any types are allowed (use unknown or explicit types instead, as demonstrated in the TDD example).

Zero Vitest Failures: All unit, integration, and contract tests must execute and pass (0 failed assertions). No disabled tests are permitted.

Mandate: You MUST NOT use .skip(), .todo(), or any form of conditional skipping.

Zero Linter Errors: All files must conform to the project's linting configuration (e.g., ESLint, Flake8/Black).

Zero Console Warnings: No debug or warning messages should be present in production code. Avoid introducing any console.warn or console.error calls.

Dynamic Values in Testing: Tests must use dynamic, randomized, or seed data (e.g., UUIDs, timestamps, dynamic strings) rather than static, hardcoded values to ensure robustness against environmental changes.

II. Frontend TypeScript/React Rules

Rule ID

Rule Description

Enforcement

TS-101

Strict Typing (No any)

The any type is strictly forbidden. Use specific types, generics, or unknown where necessary.

TS-102

Runtime Validation

All data received from the backend API (e.g., from FastAPI endpoints) must be validated at runtime (e.g., using functions like validateUser or Zod) to ensure data contract consistency.

TS-103

Hook Dependencies

All React Hooks (useEffect, useCallback, etc.) must correctly declare their dependencies. ESLint rule react-hooks/exhaustive-deps must be treated as an error.

TS-104

Functional Components

Only functional components and hooks are permitted. Class components are forbidden.

III. Backend Python/FastAPI/Postgres Rules

Rule ID

Rule Description

Enforcement

PY-201

Pydantic Model Consistency

Pydantic models (like UserInDB) must be strictly defined, matching the PostgreSQL schema and the corresponding TypeScript interfaces. Field names must use snake_case for database/API consistency.

PY-202

SQL Injection Protection

All database queries (PostgreSQL) must use parameterized queries (e.g., SQLAlchemy ORM or Core with proper bindings). String formatting or f-strings for query values are forbidden.

PY-203

Transactional Integrity

All database write operations that modify multiple records or involve critical business logic must be performed within an explicit ACID-compliant database transaction.

PY-204

Type Hinting

All functions, methods, and variables must use Python type hints (mypy compliant). Any use of # type: ignore requires explicit justification in a comment.

PY-205

Dependency Injection

Use FastAPI's dependency injection system for services, repositories, and database connections to simplify testing and promote modularity.

IV. Conflicting Scenarios/Contradictions

Based on the nature of the request, I have identified a potential point of architectural conflict regarding data modeling:

Conflict Scenario: Optional vs. Required Fields in Cross-Stack Communication

Aspect

Frontend (TS)

Backend (Pydantic/Postgres)

Behavior

last_name?: string (Optional) allows for undefined or string.

last_name: Optional[str] = None allows for None or str.

Contradiction

The Postgres column might be NOT NULL but simply empty ("") or it might be NULLABLE. The frontend validator (test/model.test.ts Case 6) assumes the absence of the field (undefined) is acceptable. If FastAPI sends null for a missing optional field, the TypeScript validator must handle null correctly (which it does via value === null). If Postgres makes the field NOT NULL, this model definition will fail upon saving a new user without a last name.



Question for Direction:

"In the User model, the last_name field is defined as optional in both TypeScript (last_name?: string) and Python (Optional[str] = None). Should the corresponding PostgreSQL column be strictly NULLABLE? Or, if last_name should be enforced as non-null in the DB, should the Python model require it, making it mandatory on the frontend as well?

Option A (Current): last_name is truly optional (NULLABLE in DB).

Option B: last_name is mandatory (NOT NULL in DB). If so, I must update models/user.ts, models/user.py, and test/model.test.ts to require this field."

The generated files adhere to the strict mandates: TDD was used (tests first, then implementation), all files are strictly typed (no any), and both the frontend and backend contexts have been addressed. The Vitest tests are runnable and pass with the provided implementation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/techieroshan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
