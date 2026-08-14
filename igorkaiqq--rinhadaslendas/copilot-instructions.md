## rinhadaslendas

> RinhaDasLendas is an internal League of Legends platform used to organize matches, players, drafts, statistics and future Discord/Riot integrations.

# AGENTS.md

## Project Overview

RinhaDasLendas is an internal League of Legends platform used to organize matches, players, drafts, statistics and future Discord/Riot integrations.

The project follows a Specification Driven Development (SDD) workflow using Spec Kit.

All implementation decisions must respect:

* Constitution
* Specifications
* Plans
* Tasks
* Architecture documentation

---

# Spec Kit Workflow

Always follow this sequence:

1. Constitution
2. Specify
3. Plan
4. Tasks
5. Implement

Do not skip phases.

Do not start implementation before tasks are approved.

---

# Git Workflow

Before starting any feature:

1. Check the current branch.

2. If the current branch is `main`, create a feature branch.

Branch naming:

```text
feature/<feature-id>-<slug>
```

Example:

```text
feature/003-cadastro-jogadores-rotas
```

3. Never implement features directly on `main`.

4. Commit after each phase:

* specify
* plan
* tasks
* implement

Examples:

```bash
git commit -m "docs: complete feature specification"
git commit -m "docs: complete implementation plan"
git commit -m "docs: generate tasks"
git commit -m "feat: implement player registration"
```

---

# Commit Messages

All commit messages must be written in Brazilian Portuguese.

Preferred patterns:

```text
docs: adicionar diretrizes de arquitetura
docs: atualizar plano da feature cadastro de jogadores

feat: implementar cadastro de jogadores
feat: adicionar atualização de preferências de rotas

fix: corrigir validação de prioridades de rota
fix: corrigir consulta de jogadores inativos

refactor: reorganizar camada de aplicação
refactor: simplificar validações de jogador

test: adicionar testes de cadastro de jogador
test: adicionar testes de preferências de rotas

chore: atualizar dependências
chore: ajustar configuração do devcontainer
```

Do not create commit messages in English.

Avoid messages such as:

```text
feat: implement player registration
fix: update route validation
docs: update architecture docs
```

Always prefer Portuguese commit messages.

---

# Repository Structure

The repository already contains:

```text
BackEnd/
FrontEnd/
.devcontainer/
docs/
specs/
```

All backend code must be created inside:

```text
BackEnd/
```

All frontend code must be created inside:

```text
FrontEnd/
```

Do not create alternative root folders such as:

```text
backend/
frontend/
api/
web/
application/
server/
client/
```

Use the existing project structure.

---

# Architecture Documentation

Always follow the documents located at:

```text
docs/architecture/
```

Current documents:

```text
docs/architecture/DESIGN_PATTERNS.md
docs/architecture/DDD_GUIDELINES.md
docs/architecture/API_STANDARDS.md
docs/architecture/DATABASE_GUIDELINES.md
```

These documents are considered the source of truth for architectural decisions.

---

# Backend Standards

Mandatory technologies:

* .NET 10
* ASP.NET Core Web API
* Entity Framework Core
* PostgreSQL
* FluentValidation
* MediatR
* Dependency Injection

Preferred architecture:

```text
Api
Application
Domain
Infrastructure
Tests
```

Business rules must not be implemented inside Controllers.

Controllers should only:

* receive requests;
* validate input;
* execute use cases;
* return responses.

---

# Development Environment

The .NET 10 SDK and backend tooling are provided by the `app` service in `.devcontainer/docker-compose.yml`. Do not assume `dotnet` is installed in the WSL host.

Before running backend commands:

1. If `dotnet --version` succeeds, the agent is already inside the devcontainer and should run commands directly.
2. Otherwise, run backend commands through the active devcontainer:

```bash
docker compose -f .devcontainer/docker-compose.yml exec -T app dotnet test /workspaces/RinhaDasLendas/BackEnd/RinhaDasLendas.sln --configuration Release
docker compose -f .devcontainer/docker-compose.yml exec -T app dotnet build /workspaces/RinhaDasLendas/BackEnd/RinhaDasLendas.sln --configuration Release
```

3. If the Linux `docker` command is unavailable, try Docker Desktop's Windows CLI with `docker.exe` before reporting a blocker. The VS Code devcontainer project uses these stable names:

```bash
docker.exe start rinhadaslendas_devcontainer-app-1
docker.exe exec rinhadaslendas_devcontainer-app-1 dotnet test /workspaces/RinhaDasLendas/BackEnd/RinhaDasLendas.sln --configuration Release
docker.exe exec rinhadaslendas_devcontainer-app-1 dotnet build /workspaces/RinhaDasLendas/BackEnd/RinhaDasLendas.sln --configuration Release
```

4. Before creating containers, inspect existing devcontainer containers with `docker.exe ps -a --filter "label=com.docker.compose.project=rinhadaslendas_devcontainer"`. Reuse/start them to avoid port conflicts and duplicate PostgreSQL volumes.
5. Run EF Core commands through the same `app` container and use paths under `/workspaces/RinhaDasLendas`.
6. Only report the backend environment as blocked after `dotnet`, `docker`, and `docker.exe` have all been checked. If none is available, request Docker Desktop WSL integration or an active devcontainer.
7. Frontend and Discord bot npm commands may run from the host when Node.js is available; backend verification still uses the devcontainer unless the current shell already has `dotnet`.

---

# Frontend Standards

Mandatory technologies:

* Vue 3
* TypeScript
* Composition API

Frontend responsibilities:

* user interaction;
* state management;
* API consumption.

Backend remains the source of truth.

Business rules must not exist only in frontend code.

---

# Database Standards

Mandatory database:

```text
PostgreSQL
```

Guidelines:

* UUID primary keys
* Explicit foreign keys
* Snake_case naming
* Migrations required
* Relational modeling preferred over JSON

---

# DDD Rules

The Domain layer must contain:

* Entities
* Value Objects
* Domain Rules
* Domain Events
* Domain Exceptions

The Domain layer must not depend on:

* Entity Framework
* PostgreSQL
* HTTP
* Controllers
* DTOs
* External APIs

The domain should be executable without infrastructure dependencies.

---

# CQRS Rules

Commands and Queries must be separated.

Examples:

```text
Commands/
Queries/
```

Do not mix read and write responsibilities.

---

# Validation Rules

Use FluentValidation.

All user input must be validated.

Examples:

* required fields;
* route priorities;
* URL validation;
* business constraints.

Avoid duplicating validation logic across layers.

---

# Internationalization Rules

All user-visible text must be internationalized.

Frontend requirements:

* Use translation keys from `FrontEnd/src/i18n/locales/pt.json` and `FrontEnd/src/i18n/locales/en.json`.
* Do not hardcode labels, buttons, titles, placeholders, tooltips, errors, confirmations, statuses, badges, empty states, toasts or validation messages in Vue components or frontend code.
* Every new key added to `pt.json` must also exist in `en.json`.
* Portuguese text must use correct accents.

Backend requirements:

* All API messages must come from `.resx` resources or an equivalent localization structure.
* Do not hardcode user-facing messages in exceptions, validators, handlers, endpoints, middlewares, responses or domain notifications.
* Every new resource key must exist in Portuguese and English.

Before finalizing any implementation, audit and report:

* Whether frontend hardcoded texts were found.
* Whether backend hardcoded messages were found.
* Whether `pt.json` and `en.json` are synchronized.
* Whether backend resources are updated.
* Whether Portuguese accentuation was reviewed.
* Whether placeholders, buttons, titles, badges, toasts and empty messages were reviewed.
* Whether frontend and backend validations use i18n/resource.
* Whether new files respect this standard.

Final responses for implementation tasks must include an `Auditoria de internacionalização` section. If any item is `Não`, the task is not complete.

---

# Design Patterns

Preferred patterns:

1. Repository
2. Dependency Injection
3. CQRS
4. Strategy
5. Command
6. Observer
7. Adapter
8. Facade
9. Builder

Apply patterns only when they solve a real problem.

Do not introduce unnecessary complexity.

---

# Testing Standards

Mandatory:

* xUnit
* FluentAssertions
* Moq

Minimum coverage:

* Domain Rules
* Validators
* Application Use Cases

Tests should be created together with implementation whenever possible.

---

# API Standards

Always follow:

```text
docs/architecture/API_STANDARDS.md
```

Requirements:

* REST
* Swagger
* DTOs
* Proper HTTP Status Codes
* Consistent error responses

Never expose entities directly.

---

# Documentation Rules

Whenever a significant decision is made:

Create or update documentation under:

```text
docs/
```

Examples:

```text
docs/architecture/
docs/domain/
docs/decisions/
```

Document first, implement second.

---

# Implementation Priority

Always prioritize:

1. Simplicity
2. Readability
3. Maintainability
4. Testability
5. Extensibility

Avoid premature optimization.

Avoid overengineering.

---

# UI and Design System

Always follow the documents located at:

docs/design/

Current documents:

- docs/design/DESIGN_SYSTEM.md
- docs/design/DESIGN_TOKENS.md
- docs/design/UI_GUIDELINES.md

All frontend implementations must follow these standards.

Do not invent colors, spacing scales, typography scales or component styles.

Use the documented design tokens whenever possible.

---

# Frontend Design Rules

Before implementing any frontend feature:

1. Read docs/design/DESIGN_SYSTEM.md
2. Read docs/design/DESIGN_TOKENS.md
3. Read docs/design/UI_GUIDELINES.md

Frontend code must respect:

- Colors
- Typography
- Spacing
- Components
- Responsive behavior

defined in the design documentation.

Do not create new design tokens unless explicitly approved.

---

# Current Feature Context

<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
at specs/028-corrigir-nucleo-ciclo-draft/plan.md
<!-- SPECKIT END -->

---
> Source: [Igorkaiqq/RinhaDasLendas](https://github.com/Igorkaiqq/RinhaDasLendas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
