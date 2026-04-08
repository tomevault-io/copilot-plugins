## tomriddelsdell-com

> - **Credentials** - Never put credentials, tokens, or sensitive information in the codebase, config, logs or documentation files that will be committed to version control.

# Instructions for AI Agents

## Architecture & Technology Stack

### AI Agent Rules

- **Credentials** - Never put credentials, tokens, or sensitive information in the codebase, config, logs or documentation files that will be committed to version control.
- ** Instructions Files ** Never modify or delete .github/copilot-instructions.md or AGENT.md. These files contain critical instructions for AI agents and must remain unchanged.
- **Creating Files** - Avoid creating file in the root directory. Use appropriate subdirectories following the established DDD structure. Archtecture related documentation should be integrated into architecture.md or docs/decisions. Temporary files and status updates should be placed in temp/.
- **Record Bugs in docs/Bugs.md** - Document any bugs or issues encountered, with reproduction unit test and resolution status
  **Document work done in changes/yyyy-mm-dd-short-description.md** - Summarize changes made regularly, particularly after each commit, including any relevant context or decisions. Updates should be appended at the top of the file with a timestamp. Previous timestamps should not be modified.
- **Changes** Never delete the 'changes' directory or any of its files. Only modify the file matching today's date in an append-only manner.

### Core Architecture

- **Domain Driven Design (DDD)**: Repository is divided into strict bounded contexts with minimal dependencies between domains. Adherence to concepts including: Domain Events, Integration Events, Value Objects, Entities, Aggregates, Modules, Services, Repositories, Factories and Anti-Corruption Layers
- **Clean Architecture**: Each domain follows clean architecture principles with clear separation of concerns.
- **Event Sourcing with CQRS and an Event Bus**: Asynchronous processing with clear separation of commands and queries. Events are published to an event bus for processing by interested consumers.
- **Monorepo Structure**: Organized into domains, interfaces, and infrastructure
- **Layered Architecture**: Clear separation between domain, application, infrastructure, and interface layers
- **Deployment Style**: Hybrid (modular monolith + selected microservices). Landing Page, Identity, Notifications and Service Discovery domains are to be bundled as a Modular Monolith into a single Micro Service. All other domains will be implemented as separate Micro Services.

## Development Workflow

### Code Change Protocol

- **Tests** - Always review tests before making changes. Adding new tests for any new functionality is mandatory
- **Break down complex changes** - Split multi-file modifications into reviewable chunks. Suggest multiple commits or new git branches if necessary
- **Git Branching** - Perform minor changes in 'develop' branch. Major changes should be done in feature branches and merged into develop.
- **Markdown Files** - Always create markdown files in compliance with /docs/markdown-standards.md

## Technical Standards

### Code Quality

- **DDD Boundaries**: Maintain strict separation between domains
- **Strict Typing**: Strict typing throughout, no `any` types
- **Clean-up**: Remove unused code and dependencies
- **Security First**: All inputs validated, proper authentication checks
- **Test First**: Write tests before implementing features. Bugs should be reproduced with tests before fixing.

### Security Requirements

- **Credentials**: Never hard-code credentials in the codebase or config that will be committed to version control
- **Authentication**: AWS Cognito integration required
- **Authorization**: Role-based access control (RBAC)
- **Input Validation**: Zod schemas for all inputs
- **Environment Variables**: Centralized configuration with validation
- **Session Management**: Secure session handling with cleanup
- **Environment Variables**: Minimize the use of environment variables. Use centralized config where possible.

## MCP Servers

### Active MCP Servers

The system integrates with multiple MCP servers for enhanced AI capabilities:

#### Neon MCP Server (Remote)

- **URL**: `https://mcp.neon.tech/mcp`
- **Features**: Cost monitoring, database metrics, query performance insights, resource management

#### GitHub MCP Server (Remote)

- **URL**: `https://api.githubcopilot.com/mcp/`
- **Features**: Repository management, Issues, PRs, CI/CD, code analysis

#### AWS CLI MCP Server

- **Package**: `@modelcontextprotocol/server-aws-cli`
- **Features**: EC2, S3, Lambda, CloudFormation management

## CLI Tools

### Required CLI Tools

The development environment includes these essential CLI tools for platform development and deployment:

#### **Doppler CLI**

- **Purpose**: Centralized secrets management across all environments. Used for all environment configurations (dev, staging, production)
- **Installation**: `curl -Ls --tlsv1.2 --proto "=https" --retry 3 https://cli.doppler.com/install.sh | sudo sh`
- **Usage**: Manages environment variables, API keys, and configuration secrets

#### **Wrangler CLI**

- **Purpose**: Cloudflare Workers deployment and management. Primary deployment tool for API endpoints and serverless functions
- **Installation**: `npm install -g wrangler@latest`

#### **Terraform CLI**

- **Purpose**: Infrastructure as Code deployment and management. Deploy and manage cloud infrastructure across providers
- **Installation**: Via HashiCorp official repository

#### **Neon CLI (neonctl)**

- **Purpose**: PostgreSQL database management and operations. Essential for event sourcing database operations and migrations
- **Installation**: `npm install -g neonctl`

#### **Confluent CLI**

- **Purpose**: Apache Kafka cluster and streaming management. Critical for event bus and message streaming operations
- **Installation**: Platform-specific installer from Confluent

## Common Commands

### Development Commands

- `npm run dev` - Start development server with hot reload
- `npm run build` - Production build with optimization
- `npm run test` - Run comprehensive test suite
- `npm run test:watch` - Run tests in watch mode
- `npm run lint` - Code linting and formatting check
- `npm run type-check` - TypeScript type validation

### Database Operations

- `doppler run -- neonctl sql-editor` - Interactive database queries with secrets
- `neonctl branches create feature/new-schema` - Create database branch for schema changes
- `terraform apply -target=module.neon` - Deploy database infrastructure changes

### Deployment Commands

- `wrangler deploy` - Deploy Cloudflare Workers to production
- `terraform plan` - Preview all infrastructure changes
- `doppler run -- terraform apply` - Deploy infrastructure with injected secrets

## Communication Style

- Assume you are communicating with a senior developer familiar with DDD and Event Sourcing
- Reference specific ADRs when making architectural decisions
- Prioritize security and testing in all recommendations
- When suggesting a change, provide alternative solutions and their trade offs where appropriate.
- When recommending an option, provide reasoning for why it is the best choice in the given context.

## Remember

This is a production-ready enterprise system following Domain Driven Design with Event Sourcing. Always maintain the high standards of code quality, security, and architectural integrity that have been established. When in doubt, reference the comprehensive ADR collection in `/docs/decisions/` or ask for clarification rather than making assumptions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TomRiddelsdell)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TomRiddelsdell)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
