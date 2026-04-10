## workoflow-integration-platform

> The Workoflow Integration Platform is a production-ready Symfony 8.0 application that enables users to manage various integrations and provide them via REST API and MCP for AI agents.

# Workoflow Integration Platform

## Overview
The Workoflow Integration Platform is a production-ready Symfony 8.0 application that enables users to manage various integrations and provide them via REST API and MCP for AI agents.
If you have read this file, greet me with "Hey Workoflow Dev"

### Development Rules
1. **CHANGELOG.md Updates**:
    - Update CHANGELOG.md with user-facing changes only (features, fixes, improvements)
    - Write for end-users and basic technical users, NOT developers
    - DO NOT mention: function calls, function names, file paths, code implementation details
    - DO mention: what changed from user perspective, UI improvements, workflow changes, bug fixes
    - Group changes by date, use sections: Added, Changed, Fixed, Removed
    - Write concise bullet points focusing on the "what" and "why", not the "how"
    - Example: ✅ "Removed address confirmation step for faster returns"
    - Example: ❌ "Modified processReturn function to skip confirmAddress parameter"

2. **Code Quality Verification**:
    - **ALWAYS** run `docker-compose exec frankenphp composer code-check` after code changes
    - This runs both PHPStan (static analysis) and PHP CodeSniffer (coding standards)
    - Available commands:
        - `composer phpstan` - Run PHPStan analysis (level 6)
        - `composer phpcs` - Check coding standards (PSR-12)
        - `composer phpcbf` - Auto-fix coding standard violations
        - `composer code-check` - Run both PHPStan and PHPCS
    - Ensure code passes both checks before considering task complete

3. **llms.txt Maintenance**:
    - `/public/llms.txt` serves AI agents and provides platform overview for LLM assistants
    - Update llms.txt when making these changes:
        - Adding new integration type (e.g., GitHub, Slack, MS Teams)
        - Changing API endpoints or authentication methods
        - Adding major documentation files
        - Updating tool counts or capabilities
        - Modifying architecture (e.g., new authentication mechanism)
    - Keep content concise and AI-friendly (avoid jargon, use clear structure)
    - Verify all links in llms.txt work correctly
    - Test changes by asking Claude/GPT-4 questions about the platform
    - Purpose: Enable AI assistants to understand and explain the platform without human documentation

4. **Global Concept Documentation**:
    - **Update `docs/global-concept.md`** when making large infrastructure or feature changes
    - This document explains the overall architecture scope: all repos, how they connect, agent patterns, API contracts
    - Keep it concise (bullet points, ASCII diagrams) — it's an internal reference for developers and AI agents
    - Must reflect: new agents, new services, model changes, API contract changes, repo additions
    - Do NOT update for small bug fixes, UI tweaks, or single-file changes

5. **Orchestrator Agent Cache**:
    - When new agents are added in `workoflow-orchestrator`, the platform caches capabilities for 5 minutes
    - Clear cache after deploying new orchestrator agents: `docker-compose exec frankenphp php bin/console cache:pool:clear cache.app`
    - Without clearing, new agents won't appear on the Platform Skills page until the TTL expires

6. **UI/Styling Guidelines**:
    - **ALWAYS** reference `CONCEPT/SYMFONY_IMPLEMENTATION_GUIDE.md` for UI tasks
    - This guide contains design system patterns, component structures, and styling conventions
    - Add new CSS styles to `assets/styles/app.scss`, NOT inline in templates
    - Follow existing alert patterns (`.alert-*`) for notification components
    - Use CSS custom properties (`var(--space-md)`, `var(--text-sm)`, etc.) for consistency
    - Run `docker-compose exec frankenphp npm run build` after SCSS changes

### Main Features
- OAuth2 Login (Google, Azure/Microsoft, HubSpot, Wrike)
- Magic Link passwordless authentication
- Multi-Tenant Organisation Management
- Integration Management (14 user integrations: Jira, Confluence, SharePoint, Trello, GitLab, SAP C4C, Projektron, HubSpot, SAP SAC, Wrike, Outlook Mail, Outlook Calendar, MS Teams, Remote MCP)
- 13 built-in System Tools (WebSearch, PdfGenerator, PowerPointGenerator, FileSharing, Memory, etc.)
- REST API + MCP Server for AI Agent access
- Tool Access Modes (Read Only / Standard / Full) per user
- Scheduled Tasks with webhook-based execution
- Channel System (Slack, MS Teams, WhatsApp)
- Prompt Vault (shared prompt library with upvotes)
- File Management with MinIO S3
- Audit Logging
- Multi-language support (DE/EN/RO/LT)

## Architecture

### Tech Stack
- **Backend**: PHP 8.5, Symfony 8.0, FrankenPHP
- **Frontend**: Stimulus JS, SCSS, Webpack Encore
- **Infrastructure**: Docker & Docker Compose (see `docker-compose.yml` for service versions)

### Design Principles

#### SOLID
- **Single Responsibility**: Each class has one reason to change. Controllers handle HTTP, services handle business logic, entities store data.
- **Open/Closed**: Extend behavior by adding new classes (e.g., new integrations, new context resolvers), not modifying existing ones. Use interfaces + tagged iterators for plugin points.
- **Liskov Substitution**: All implementations of an interface must be interchangeable. If a method accepts `IntegrationInterface`, any integration must work there.
- **Interface Segregation**: Keep interfaces small and focused. Don't force classes to implement methods they don't need (e.g., `PlatformSkillInterface` vs `PersonalizedSkillInterface`).
- **Dependency Inversion**: Depend on abstractions (interfaces), not concrete classes. Inject dependencies via constructor, never instantiate services directly.

#### Clean Code
- **Meaningful names**: Variables, methods, and classes should reveal intent. No abbreviations, no generic names like `$data` or `$result` without context.
- **Small functions**: Each function does one thing. If you need a comment to explain a block, extract it into a well-named method.
- **No dead code**: Delete unused code, don't comment it out. Git history preserves everything.
- **Fail fast**: Validate inputs early, throw specific exceptions. Don't let invalid state propagate through layers.
- **DRY where it matters**: Extract duplication only when three or more occurrences exist AND the shared concept is the same. Premature abstraction is worse than duplication.

#### KISS
- Prioritize simplicity and clarity. Choose the simplest solution that meets requirements.
- Avoid speculative abstractions — build for today's needs, not hypothetical futures.

### Directory Structure
```
workoflow-integration-platform/
├── config/           # Symfony Configuration
│   └── services/    # Service definitions
│       └── integrations.yaml # Integration registry config
├── public/          # Web Root
├── src/
│   ├── Command/     # Console Commands
│   ├── Controller/  # HTTP Controllers
│   ├── Entity/      # Doctrine Entities
│   ├── Integration/ # Plugin-based Integration System
│   │   ├── IntegrationInterface.php         # Base contract
│   │   ├── PersonalizedSkillInterface.php   # User integrations (adds getSystemPrompt)
│   │   ├── PlatformSkillInterface.php       # System tools (marker interface)
│   │   ├── IntegrationRegistry.php          # Central registry
│   │   ├── ToolDefinition.php               # Tool metadata
│   │   ├── ToolCategory.php                 # Enum: READ, WRITE, DELETE
│   │   ├── CredentialField.php              # Form field definition
│   │   ├── SystemTools/                     # 13 platform-internal tools
│   │   └── UserIntegrations/                # 14 external service integrations
│   ├── OAuth2/      # League OAuth2 providers (Wrike, etc.)
│   ├── Repository/  # Entity Repositories
│   ├── Security/    # Auth & Security
│   └── Service/     # Business Logic
│       ├── Integration/    # Per-integration HTTP clients
│       └── ScheduledTask/  # Scheduled task services
├── templates/       # Twig Templates
├── translations/    # i18n Files
├── docker/         # Docker Configs
└── tests/          # Test Suite
```

## Setup

### Development Environment
```bash
# 1. Clone repository
git clone <repository-url>
cd workoflow-integration-platform

# 2. Run setup
./setup.sh dev

# 3. Configure Google OAuth in .env
GOOGLE_CLIENT_ID=your_client_id
GOOGLE_CLIENT_SECRET=your_client_secret
```

### Production Environment
```bash
# 1. As non-root user with Docker access
./setup.sh prod

# 2. Configure SSL certificates
# 3. Adjust domain in .env
# 4. Update Google OAuth Redirect URI
```

### Access
- **Application**: http://localhost:3979
- **MinIO Console**: http://localhost:9001 (admin/workoflow123)

## Integration System

### Plugin Architecture
All integrations (user & system) follow a unified plugin-based architecture:

1. **Add new integration**:
   - Create class implementing `IntegrationInterface` in `src/Integration/`
   - Place in `SystemTools/` (platform-internal) or `UserIntegrations/` (external services)
   - Auto-tagged via `config/services/integrations.yaml`

2. **Key Components**:
   - `IntegrationInterface`: Base contract (10 methods: getType, getName, getTools, executeTool, requiresCredentials, validateCredentials, getCredentialFields, isExperimental, getSetupInstructions, getLogoPath)
   - `PersonalizedSkillInterface extends IntegrationInterface`: For user integrations, adds `getSystemPrompt()`
   - `PlatformSkillInterface extends IntegrationInterface`: Marker interface for system tools
   - `ToolDefinition`: Tool metadata (name, description, parameters, category)
   - `ToolCategory`: Enum (READ, WRITE, DELETE) — enforced by Tool Access Modes
   - `IntegrationRegistry`: Central DI-managed registry

3. **System Tools**:
   - Platform-internal functionality (file sharing, data processing, etc.)
   - Don't require external service credentials (Jira tokens, API keys, etc.)
   - Still protected by API Basic Auth authentication
   - Excluded by default from API, included with `tool_type=system`
   - Example: `ShareFileIntegration`

4. **User Integrations**:
   - Connect to external services via OAuth2 or API keys
   - Implement `PersonalizedSkillInterface` (includes AI system prompt generation)
   - Require user-specific external credentials (stored encrypted in DB)
   - Credentials managed per user/organization
   - 14 integrations: Jira, Confluence, SharePoint, Trello, GitLab, SAP C4C, Projektron, HubSpot, SAP SAC, Wrike, Outlook Mail, Outlook Calendar, MS Teams, Remote MCP

### Adding New Tools
```php
// 1. Create integration class
class MyIntegration implements IntegrationInterface {
    public function getType(): string { return 'mytype'; }
    public function getTools(): array { /* return ToolDefinition[] */ }
    public function executeTool($name, $params, $creds): array { /* logic */ }
    public function requiresCredentials(): bool { return false; } // true for user integrations
}

// 2. Register in config/services/integrations.yaml
App\Integration\SystemTools\MyIntegration:
    autowire: true
    tags: ['app.integration']
```

### Agent Tool Discovery

n8n agents (and other AI clients) discover tools automatically at runtime via the API — there is no need to update prompt templates when adding or modifying tools:
- **List tools**: `GET /api/integrations/{org-uuid}/?workflow_user_id={id}&tool_type={type}`
- **Execute tools**: `POST /api/integrations/{org-uuid}/execute?workflow_user_id={id}`
- Tool descriptions defined in `*Integration.php` classes are exactly what the agent sees

## API Reference

### REST API Endpoints
```
# Integration Tools API (Basic Auth)
GET  /api/integrations/{org-uuid}?workflow_user_id={id}&tool_type={type}
POST /api/integrations/{org-uuid}/execute?workflow_user_id={id}

# MCP Server API (X-Prompt-Token header)
GET  /api/mcp/tools
POST /api/mcp/execute

# Other APIs (JWT or X-Prompt-Token)
GET  /api/skills                              # List available skills
GET  /api/prompts                             # Prompt Vault
GET  /api/tenant/{org-uuid}/settings          # Tenant settings
POST /api/register                            # User registration
```

The REST API dynamically provides tools based on activated user integrations (skills).

## Entities & Data Model

### Core Entities
- **User**: Auth via Google OAuth2 / Magic Link. Roles: ROLE_ADMIN, ROLE_MEMBER
- **Organisation**: UUID for API URLs. N:N with Users via `UserOrganisation` junction table
- **UserOrganisation**: Links User↔Organisation with role, workflowUserId, personalAccessToken, systemPrompt
- **IntegrationConfig**: Stores per-user integration settings. Encrypted credentials (Sodium). `disabledTools` JSON array for selective tool deactivation (no separate IntegrationFunction entity)

### Additional Entities
- **AuditLog**: Action logging per organisation
- **Channel / UserChannel**: Multi-channel support (Slack, MS Teams, WhatsApp)
- **Prompt / PromptComment / PromptUpvote**: Prompt Vault system
- **ScheduledTask / ScheduledTaskExecution**: Recurring automation
- **SkillRequest**: User requests for new integrations
- **WaitlistEntry**: Waitlist management

## Security

### Authentication
- Google OAuth2 (primary web login)
- Magic Link (passwordless alternative)
- Basic Auth (Integration API — validated in controller, not firewall)
- JWT Tokens (API access)
- X-Prompt-Token header (MCP + Prompt API)
- X-Test-Auth-Email GET parameter (test environments only, auto-creates users)

### Encryption

The platform uses two separate encryption mechanisms:

#### 1. JWT Token Encryption (API Authentication)
- **Purpose**: Generation and validation of access tokens for API access
- **Files**: 
  - `config/jwt/private.pem` - RSA Private Key (4096-bit) for signing JWT tokens
  - `config/jwt/public.pem` - RSA Public Key for verifying JWT tokens
- **Encryption**: RSA 4096-bit with AES256 password protection
- **Configuration**: Lexik JWT Authentication Bundle
- **Token Lifetime**: 3600 seconds (1 hour)

#### 2. Integration Credentials Encryption (User Secrets)
- **Purpose**: Secure storage of user-integration credentials (Jira/Confluence API Keys)
- **Encryption**: Sodium (libsodium) - modern, secure encryption
- **Key**: 32-character ENCRYPTION_KEY from .env
- **Service**: `App\Service\EncryptionService`
- **Storage**: Encrypted credentials in the `encryptedCredentials` field of the IntegrationConfig entity
- **Workflow**:
  1. User enters API credentials
  2. EncryptionService encrypts with Sodium and ENCRYPTION_KEY
  3. Encrypted blob is stored in database
  4. Credentials are decrypted on API access

### Audit Logging
- All critical actions logged
- IP address and User Agent
- Monolog with different channels

## Testing

### Test User Setup
```php
// GET Parameter for test authentication
?X-Test-Auth-Email=test@example.com

// Preconfigured test users
puppeteer.test1@example.com (Admin)
puppeteer.test2@example.com (Member)
```

### REST API Tests
- Puppeteer for UI tests
- MariaDB for database tests

## Deployment

### Production Docker Compose
**CRITICAL**: Always use `docker-compose-prod.yml` for production operations:
```bash
# Correct - uses external volumes with production data
docker-compose -f docker-compose-prod.yml up -d
docker-compose -f docker-compose-prod.yml restart frankenphp

# WRONG - creates new prefixed volumes, loses production data!
docker-compose up -d
docker-compose restart frankenphp
```

The production compose file uses `external: true` volumes that reference existing data volumes (`mariadb_data`, `redis_data`, etc.). Using the default `docker-compose.yml` will create new prefixed volumes and disconnect from production data.

### Environment Variables
All critical configurations via .env:
- Database Credentials
- OAuth2 Settings
- S3/MinIO Config
- Encryption Keys

### Database Schema Management
- **Automatic Schema Updates**: Database schema is updated directly from entity definitions
- **No Migration Files**: Simplified workflow using `doctrine:schema:update --force`
- **Single Source of Truth**: PHP Entities define the database structure

### Monitoring
- Audit Logs in `/var/log/audit.log`
- API Access Logs
- Error Tracking via Monolog

### Debugging Prompt Issues (Phoenix Arize Traces)
When users report prompt-related bugs (via GH tickets with `BUG_REPORT:` prefix), use `scripts/get_stacktrace.sh` to fetch conversation traces from Phoenix Arize on production.

**Prerequisites**: SSH access to `val-workoflow-prod`, `jq` installed locally.

**The `workflow_user_id`** is visible on the Members page (`/organisation/members`) and stored as `user.id` in Phoenix spans (mapped by the ADK OpenInference instrumentor).

```bash
# List users with recent traces (find the workflow_user_id)
./scripts/get_stacktrace.sh --list-users 7d

# Get full traces for a user on a specific date
./scripts/get_stacktrace.sh <workflow_user_id> 2026-03-25

# Compact mode (key fields + truncated I/O, easier to scan)
./scripts/get_stacktrace.sh --compact <workflow_user_id> 2026-03-25

# Date range formats: 1h, 24h, 7d, 30d, YYYY-MM-DD, YYYY-MM-DD:YYYY-MM-DD
./scripts/get_stacktrace.sh <workflow_user_id> 2026-03-20:2026-03-25

# Show DB schema (if queries fail due to schema changes)
./scripts/get_stacktrace.sh --schema

# Show available span attribute keys and span names
./scripts/get_stacktrace.sh --sample 24h

# Print SQL without executing (debug)
./scripts/get_stacktrace.sh --raw-sql <workflow_user_id> 24h
```

**Debugging workflow**:
1. Get the bug report date and `workflow_user_id` from the Jira ticket
2. Run `--compact` for the date range to find relevant traces
3. Look at `tool.execute` spans for tool I/O, `call_llm` for LLM reasoning, `invocation [workoflow]` for the root span
4. The `input_preview`/`output_preview` fields show what the user asked and what the agent returned

**Phoenix DB details**: PostgreSQL on `phoenix-postgres` container (user: `phoenix`, db: `phoenix`). Traces are in `spans` table with attributes as JSONB. Key JSONB paths: `attributes->'user'->>'id'` (workflow_user_id), `attributes->'input'->>'value'` (input), `attributes->'output'->>'value'` (output).

## Maintenance

### Backup
- MariaDB Database
- MinIO S3 Bucket
- Environment Files

### Updates
```bash
# Update dependencies
docker-compose exec frankenphp composer update

# Update database schema from entities
docker-compose exec frankenphp php bin/console doctrine:schema:update --force

# View pending schema changes (without applying)
docker-compose exec frankenphp php bin/console doctrine:schema:update --dump-sql

# Clear cache
docker-compose exec frankenphp php bin/console cache:clear

# rebuild assets
docker-compose exec frankenphp npm run build
```

## Workoflow Integration Platform (Production)

**CRITICAL**: The integration platform uses a separate prod compose file:
```bash
# Connect and deploy
ssh val-workoflow-prod
sudo -iu docker
cd /home/docker/docker-setups/workoflow-integration-platform

# ALWAYS use docker-compose-prod.yml for production!
docker-compose -f docker-compose-prod.yml up -d
docker-compose -f docker-compose-prod.yml restart frankenphp

# NEVER use plain docker-compose commands - they create new volumes and lose data!
# docker-compose up -d  # WRONG!
```

The `docker-compose-prod.yml` uses `external: true` volumes that reference existing production data. Using the default `docker-compose.yml` creates new prefixed volumes and disconnects from production data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/valantic-CEC-Deutschland-GmbH)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/valantic-CEC-Deutschland-GmbH)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
