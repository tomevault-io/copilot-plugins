## exodia

> - Alwaysin plan mode to make a plan

# Exodia - Outil moderne pour appels à projets

## Plan & Review & TDD

### Before starting work

- Alwaysin plan mode to make a plan
- After get the plan, make sure you Write the plan to .claude/tasks/TASK_NAME.md
- Use checkboxes to mark tasks as completed and incomplete and headings to structure the plan.

- The plan should be a detailed implementation plan and the reasoning behind them, as well as tasks broken down.
- If the task require external knowledge or
  certain package, also research to get latest knowledge (Use Task tool for research)
- Don't over plan it, always think MVP.
- Once you write the plan, firstly ask me to
  review it. Do not continue until I approve the plan.
- If the plan is approved, you can start implementing.

### While implementing

- You shoul update the plan as your work.
- After you complete tasks in the plan, you should update and append detailed descriptions of the changes you made, so following tasks can be easily hand over to other engineers.

### Before implementing

- You shall not write code until you have written a failing test.
- You shall only write one new failing unit test at a time.
- You shall not write more code than is necessary to make the failing test pass.

The approach to developing in TDD is as follows:

- Write a test that fails because the relevant code does not exist
- Write just enough code to validate the new test and the previous ones
- Optimize the code without altering its behavior

### Test Coverage

- Use pnpm test:coverage to run tests with coverage
- Use pnpm test:ui to run tests with ui

## Project Overview

Create a modern, innovative, and highly intuitive tool for designing and automating call for projects responses with advanced AI features and a modern design inspired by Google Notebook, ChatGPT Canvas.

- Centralization of necessary information and documents
- Automation of repetitive tasks (document generation, tracking, notifications)
- Simple and efficient user interface
- Modular architecture to facilitate scalability
- Security and compliance (authentication, access management)

The goal is to transform call for projects management into a seamless, fast, and collaborative process, leveraging tech and UX best practices.

## Architecture & Technology Stack [`.claude/docs/architecture.md`](.claude/docs/architecture.md)

## Development Commands [`package.json`](package.json)

## Core Features

### 1. Gestion de projet

_See [`.claude/docs/user-management.md`](.claude/docs/user-management.md)_

- CRUD des organisations
- Gestion des membres
- CRUD des projets
- Onboarding des projets

### 2. Automatisation & vectorisation des documents (RAG)

_See [`.claude/docs/rag-system.md`](.claude/docs/rag-system.md) & [`.claude/docs/document-processing.md`](.claude/docs/document-processing.md)_

- Upload de documents
- Vectorisation des documents
- Recherche de documents
- Suppression de documents
- Génération automatisée de documents

### 3. Collaboration & gestion des accès

_See [`.claude/docs/collaboration.md`](.claude/docs/collaboration.md)_

- Accès basé sur les rôles
- Cloisonnement sécurisé des données par projet
- Gestion des documents partagés
- Journaux d'audit

### 4. Notifications & suivi

_See `.claude/docs/collaboration.md`_

- Notifications d'assignation de tâches
- Mises à jour de statut de projet
- Rappels d'échéances
- Fil d'activité

### 5. Sécurité & conformité

_See [`.claude/docs/auth.md`](.claude/docs/auth.md) & [`.claude/docs/database.md`](.claude/docs/database.md)_

- Authentification
- Application du contrôle d'accès
- Chiffrement des données
- Reporting conformité

## User Map

### Epics by Theme

#### Gestion de projet

- **CRUD des organisations** : Créer, lire, modifier et supprimer des organisations
- **Gestion des membres** : Inviter, gérer les rôles et permissions des membres [`.claude/docs/invitation-system.md`](.claude/docs/invitation-system.md)
- **CRUD des projets** : Créer, organiser et gérer les projets d'appels à offres
- **Onboarding des projets** : Guide d'aide pour démarrer efficacement un nouveau projet [`.claude/docs/user-management.md`](.claude/docs/user-management.md)

#### Automatisation & vectorisation des documents

- **Upload de documents** : Interface de glisser-déposer pour les fichiers PDF, DOCX, TXT [`.claude/docs/document-processing.md`](.claude/docs/document-processing.md)
- **Vectorisation des documents** : Traitement automatique et conversion en embeddings [`.claude/docs/document-processing.md`](.claude/docs/document-processing.md)
- **Recherche de documents** : Recherche sémantique et par mot-clé dans la base documentaire [`.claude/docs/rag-system.md`](.claude/docs/rag-system.md)
- **Suppression de documents** : Gestion du cycle de vie et suppression sécurisée [`.claude/docs/document-processing.md`](.claude/docs/document-processing.md)
- **Génération automatisée de documents** : IA pour créer des réponses personnalisées [`.claude/docs/project-generation.md`](.claude/docs/project-generation.md)

#### Collaboration & gestion des accès

- **Accès basé sur les rôles** : Système de permissions granulaires (admin, member) [`.claude/docs/user-management.md`](.claude/docs/user-management.md)
- **Cloisonnement sécurisé des données par projet** : Isolation complète entre projets [`.claude/docs/database.md`](.claude/docs/database.md)
- **Gestion des documents partagés** : Contrôle fin du partage et des autorisations [`.claude/docs/collaboration.md`](.claude/docs/collaboration.md)
- **Journaux d'audit** : Traçabilité complète des actions utilisateurs [`.claude/docs/user-management.md`](.claude/docs/user-management.md)

#### Notifications & suivi

- **Notifications d'assignation de tâches** : Alertes en temps réel pour les nouvelles tâches `.claude/docs/collaboration.md`
- **Mises à jour de statut de projet** : Suivi des avancées et changements d'état [`.claude/docs/collaboration.md`](.claude/docs/collaboration.md)
- **Rappels d'échéances** : Système d'alertes automatiques pour les deadlines [`.claude/docs/collaboration.md`](.claude/docs/collaboration.md)
- **Fil d'activité** : Timeline chronologique de toutes les activités du projet [`.claude/docs/collaboration.md`](.claude/docs/collaboration.md)

#### Sécurité & conformité

- **Authentification** : Magic Link email sécurisé via Supabase Auth [`.claude/docs/auth.md`](.claude/docs/auth.md)
- **Application du contrôle d'accès** : Enforcement des permissions via RLS [`.claude/docs/database.md`](.claude/docs/database.md)
- **Chiffrement des données** : Protection des données sensibles en transit et au repos [`.claude/docs/database.md`](.claude/docs/database.md)

## Environment Variables

_See [`.claude/docs/environment.md`](.claude/docs/environment.md)_

The project uses [`@t3-oss/env-nextjs`](https://env.t3.gg/docs/nextjs) for type-safe environment variable management with Zod validation.

## UI Design System & Interface

_See [`.claude/docs/interface.md`](.claude/docs/interface.md)_

**⚠️ HIGH UPDATE FREQUENCY** - This documentation requires frequent updates during UI development.

The interface documentation serves as both specification and task tracking system with checkboxes for implementation status. It maps directly to:

### File Relationships

- **`src/components/ui/`** - shadcn/ui component implementations (13 components)
- **`tailwind.config.js`** - Design system configuration and theme tokens
- **`src/app/globals.css`** - CSS custom properties and dark/light theme definitions
- **[shadcn/ui docs](https://ui.shadcn.com/docs/components)** - External component reference

### Maintenance Requirements

- Update immediately when UI components are added/modified
- Track implementation progress with checkboxes
- Document deviations from design specifications
- Sync design tokens between [`globals.css`](src/app/globals.css) and documentation
- Weekly reviews during active UI development

### Implementation Status

The interface.md file tracks:

- Design system foundation (colors, typography, spacing)
- Component library status (existing, missing, custom)
- Page implementation progress with detailed checklists
- Subscription tier UI differentiation
- Responsive design and accessibility compliance

## Database Schema (Supabase)

The database is hosted on Supabase, and the database schema is defined in the [`.claude/docs/database.md`](.claude/docs/database.md) file. Use Supabase MCP `.mcp.json` to check the database status and logs. Ensures that the [`.claude/docs/database.md`](.claude/docs/database.md) file is kept up to date with the latest database changes.

## Deployment (Vercel)

Automatic deployment to Vercel on main branch: Use Vercel CLI to check the deployment status and logs.

## Contributing

1. Create feature branch from `main`
2. Run `pnpm lint` and `pnpm typecheck` before commits
3. Write tests for new features
4. Add documentation (<250 lines) as needed [`.claude/docs/`](.claude/docs/) and update [`CLAUDE.md`](CLAUDE.md) in english

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leobrival) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
