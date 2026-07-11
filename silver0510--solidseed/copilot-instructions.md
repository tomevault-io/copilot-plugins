## solidseed

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**SolidSeed** is a modern CRM platform designed specifically for real estate professionals (realtors, agents, and loan officers). It serves as a replacement for FollowUpBoss, focusing on:

- **Client Hub**: Centralized client management with profiles, documents, notes, tasks, and tag-based organization
- **Email Marketing**: Integrated communication tracking and automation (planned)
- **Mobile-First Design**: Optimized for on-the-go access during property showings
- **Data Portability**: Agents retain their client data when switching brokers
- **SAAS Model**: Subscription tiers (trial, free, pro, enterprise) with 14-day trial period

### Current Status

The project is in early development with two primary features in planning:

1. **User Authentication** - Email/password + OAuth (Google, Microsoft) with Better Auth library and Supabase
2. **Client Hub** - Client management platform with GDPR compliance

## Architecture

### Technology Stack

**Database & Storage:**

- **Supabase** - PostgreSQL database hosting and management
- Supabase CLI for migrations (`supabase/migrations/` directory)
- Connection via `SUPABASE_DATABASE_URL` environment variable

**Authentication:**

- **Better Auth** library with Supabase PostgreSQL adapter
- JWT tokens for session management (3-day default, 30-day with "remember me")
- OAuth 2.0 integration (Google Cloud Platform, Microsoft Azure AD)
- Bcrypt password hashing (cost factor 12)
- Email verification required for registration

**Security Features:**

- Account lockout after 5 failed login attempts (30-minute lock)
- Rate limiting: 10 login attempts/min per IP, 3 password resets/hr per email
- Trial period: 14 days from email verification
- Soft delete with `is_deleted` flag

### Database Schema

**Authentication tables** (5 tables):

- `users` - Main user accounts with subscription tiers and trial tracking
- `oauth_providers` - Social login provider mappings
- `password_resets` - Password reset tokens (1-hour expiration)
- `email_verifications` - Email verification tokens (24-hour expiration)
- `auth_logs` - Security audit trail (7-day retention)

**Client Hub tables** (5 tables):

- `clients` - Client profiles with soft delete
- `client_tags` - Flexible tagging system for organization
- `client_documents` - Document storage with chronological sorting (no categories)
- `client_notes` - Activity and interaction notes
- `client_tasks` - Task management with due dates and priorities

## Project Management System (CCPM)

This repository uses Claude Code Project Management (CCPM) - a markdown-based project management system integrated into the `.claude/` directory.

### Directory Structure

```
.claude/
├── prds/                    # Product Requirements Documents
│   ├── client-hub.md
│   └── user-authentication.md
├── epics/                   # Technical implementation plans
│   └── user-authentication/
│       ├── epic.md          # Technical plan and architecture
│       ├── 001.md           # Task: Database schema
│       ├── 002.md           # Task: Better Auth integration
│       └── ...              # Additional tasks
├── commands/                # Custom slash commands
│   ├── pm/                  # Project management commands
│   ├── context/             # Context management
│   └── testing/             # Test execution
├── rules/                   # Development guidelines
│   ├── datetime.md          # ISO 8601 datetime standards
│   ├── frontmatter-operations.md
│   ├── path-standards.md
│   ├── github-operations.md
│   └── standard-patterns.md
├── skills/                  # Reusable skill definitions
├── agents/                  # Specialized agent configurations
└── ccpm.config              # GitHub repository configuration
```

### Key PM Commands

**Creating Work:**

```bash
/pm:prd-new <name>           # Create Product Requirements Document
/pm:prd-parse <name>         # Convert PRD to technical epic
/pm:epic-decompose <name>    # Break epic into tasks
```

**Viewing Status:**

```bash
/pm:status                   # Project dashboard
/pm:epic-show <name>         # View epic and all tasks
/pm:prd-list                 # List all PRDs
/pm:next                     # Show next priority tasks
```

**GitHub Sync (Optional):**

```bash
/pm:epic-sync <name>         # Push epic and tasks to GitHub Issues
/pm:sync                     # Bidirectional sync with GitHub
```

### Local-Only Mode

CCPM works entirely offline without GitHub integration. All project management is done through local markdown files with YAML frontmatter. See `LOCAL_MODE.md` for complete local workflow.

### File Frontmatter Standards

All markdown files use YAML frontmatter with ISO 8601 timestamps:

```yaml
---
name: feature-name
status:
  open # PRDs: backlog/in-progress/complete
  # Epics: backlog/in-progress/completed
  # Tasks: open/in-progress/closed
created: 2026-01-06T08:26:55Z
updated: 2026-01-06T08:58:07Z
depends_on: [001, 002] # Task dependencies
parallel: true # Can run in parallel
---
```

**Important:** Always use real system datetime (`date -u +"%Y-%m-%dT%H:%M:%SZ"`), never placeholders. See `.claude/rules/datetime.md`.

## Development Guidelines

### Path Standards

**Always use relative paths** in documentation and code:

- ✅ `../project-name/internal/auth/server.go`
- ✅ `internal/processor/converter.go`
- ❌ `/Users/username/project-name/...` (exposes local structure)

See `.claude/rules/path-standards.md` for complete specification.

### Supabase Setup

**Environment Variables Required:**

```bash
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
SUPABASE_DATABASE_URL=postgresql://postgres:[password]@db.your-project.supabase.co:5432/postgres
GOOGLE_CLIENT_ID=xxx
GOOGLE_CLIENT_SECRET=xxx
MICROSOFT_CLIENT_ID=xxx
MICROSOFT_CLIENT_SECRET=xxx
JWT_SECRET=xxx
APP_URL=http://localhost:3000
```

**Migration Workflow:**

```bash
# Initialize Supabase locally
supabase init

# Create new migration
supabase migration new create_table_name

# Apply migrations to remote
supabase db push

# View database in Supabase Studio
open https://app.supabase.com
```

### Better Auth Configuration

```typescript
export const authConfig = {
  database: {
    adapter: postgresAdapter({
      connectionString: process.env.SUPABASE_DATABASE_URL,
    }),
  },
  // ... additional config in .claude/epics/user-authentication/epic.md
};
```

### GitHub Operations

The repository includes protection against accidentally creating issues in template repositories. All GitHub operations check remote origin and enforce using the correct repository. See `.claude/rules/github-operations.md`.

## Context Management

The `.claude/context/` directory contains living documentation about the project state:

```bash
/context:create              # Initialize project context
/context:prime               # Load context for new session
/context:update              # Update context after changes
```

Context files include:

- `project-brief.md` - Scope, goals, objectives
- `tech-context.md` - Dependencies and tools
- `progress.md` - Current status and next steps
- `project-structure.md` - Directory organization

## Development Workflow

1. **Start Session:** `/context:prime` to load current project state
2. **Create Requirements:** `/pm:prd-new feature-name` for new features
3. **Technical Planning:** `/pm:prd-parse feature-name` converts to implementation epic
4. **Task Breakdown:** `/pm:epic-decompose feature-name` creates individual task files
5. **Implementation:** Work on tasks in `.claude/epics/feature-name/###.md`
6. **Update Context:** `/context:update` at end of session

## Key Principles

1. **Mobile-First**: All UI optimized for 375px+ width screens
2. **Security-First**: Authentication and data protection are foundational
3. **Data Portability**: Users own their data
4. **GDPR Compliant**: Privacy by design with export capabilities
5. **Subscription Tiers**: Trial (14 days) → Free → Pro → Enterprise
6. **No Over-Engineering**: Simple solutions over premature abstractions
7. **Markdown Everything**: All documentation, PRDs, epics, and tasks in markdown

## Important Files to Reference

- `.claude/rules/` - Development standards and patterns
- `.claude/prds/` - Product requirements for features
- `.claude/epics/` - Technical implementation plans
- `LOCAL_MODE.md` - Complete local workflow guide
- `.claude/ccpm.config` - GitHub repository configuration

---
> Source: [silver0510/solidseed](https://github.com/silver0510/solidseed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
