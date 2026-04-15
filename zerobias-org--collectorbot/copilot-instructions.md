## collectorbot

> This file provides guidance to Claude Code (claude.ai/code) when working with collector bot content in this repository.

# CLAUDE.md - ZeroBias Community Collector Bot Repository

This file provides guidance to Claude Code (claude.ai/code) when working with collector bot content in this repository.

## IMPORTANT: Development Rules

**Before creating or modifying any collector bot, Claude MUST read and follow:**
- **[.claude/COLLECTORBOT_RULES.md](.claude/COLLECTORBOT_RULES.md)** - Comprehensive rules for all file types, naming conventions, and patterns
- **[.claude/DEVELOPMENT_WORKFLOW.md](.claude/DEVELOPMENT_WORKFLOW.md)** - Step-by-step workflow for creating and modifying collectors
- **[.claude/ADVANCED_MAPPING_GUIDE.md](.claude/ADVANCED_MAPPING_GUIDE.md)** - Advanced mapping, validation, and relationship handling
- **[.claude/VENDOR_PATTERNS.md](.claude/VENDOR_PATTERNS.md)** - Vendor-specific architectural patterns and best practices

**Creation & Validation System:**
- **[/create-collector](.claude/skills/create-collector.md)** - Create new collector with automatic validation
- **[/review-collector](.claude/skills/review-collector.md)** - Comprehensive validation using 8 parallel agents
- **[.claude/agents/*.md](.claude/agents/)** - 10 specialized agents (2 for creation, 8 for validation)

**Rules document defines:**
- Package structure and naming conventions (`package/<vendor>/<suite-if-exists>/<product>/`)
- NPM scope rules (`@zerobias-org/collectorbot-*` for this repository)
- Schema package usage (base vs `-ts` packages)
- Required vs optional source files
- Batch processing and groupId rules (CRITICAL for data integrity)
- Configuration file templates and validation rules

**Workflow document defines:**
- 10-phase development process from planning to commit
- Modification workflows for existing collectors
- Dependency verification (module and schema must exist)
- Configuration file creation and code generation
- Implementation patterns and best practices
- Testing and documentation requirements
- Schema class selection (collect everything possible)
- Parameter design (ask user what they need)
- Concurrency limits for nested requests (≤30 total)

**Advanced mapping guide defines:**
- Schema validation using base + TypeScript packages
- Date vs DateTime handling (Object.assign workaround)
- Semantic field mapping strategies
- Enum mapping patterns
- ID generation from unique fields
- Object relationship discovery and implementation
- Complex type conversions and edge cases
- TODO pattern for uncertain mappings
- Performance optimization (memory management, concurrency, pagination)
- Incremental collection patterns (timestamp, state, cursor-based)
- Retry logic and resilience (transient failures, rate limiting)

**Vendor patterns document defines:**
- AWS: Account + region patterns, ARN-based IDs, metadata structure
- Azure: Subscription + resource group patterns
- GCP: Project + zone patterns
- GitHub: Organization patterns, rate limit considerations
- Microsoft 365/Entra: Tenant-based patterns
- Generic SaaS: Multi-tenant patterns
- Shared collectors: Global data patterns

## Project Overview

This is the **ZeroBias Community Collector Bot Repository** containing open-source ETL (Extract/Transform/Load) logic for data collection. Collector bots read from source systems using Hub Modules and write to AuditgraphDB (objects described by Schema classes).

**Repository Role:** Community-contributed, open-source content for automated data collection from external systems

## Repository Structure

```
collectorbot/
├── .claude/                    # Claude Code configuration
│   └── COLLECTORBOT_RULES.md  # ⭐ MUST READ - Development rules
├── package/                    # Collector bot packages
│   └── <vendor>/              # Vendor directory
│       └── <suite?>/          # Optional suite directory
│           └── <product>/     # Product collector
│               ├── package.json
│               ├── collector.yml
│               ├── hub.yml
│               ├── parameters.yml
│               ├── tsconfig.json
│               ├── .eslintrc
│               ├── .mocharc.json
│               ├── generated/     # Auto-generated (DO NOT EDIT)
│               ├── src/           # Source code
│               └── test/          # Tests
├── collector/                  # ⚠️ DEPRECATED - Do not use
└── lerna.json                 # Monorepo configuration
```

## Quick Reference

### Creating a New Collector (Automated)

**Use the creation skill:**
```bash
/create-collector <vendor> <suite?> <product>

# Examples:
/create-collector avigilon alta access
/create-collector github github
/create-collector auditmation generic tls
```

This will:
1. Verify module and schema dependencies exist
2. Create directory structure and configuration files
3. Generate code and spawn implementation + mapping agents in parallel
4. Create documentation (README, USERGUIDE)
5. Build and lint
6. **Automatically validate with /review-collector**
7. Report compliance score and any issues

**Manual creation** (if needed):
1. Create directory: `package/<vendor>/<suite?>/<product>/`
2. Copy structure from existing collector
3. Follow `.claude/DEVELOPMENT_WORKFLOW.md` step-by-step

### Key Commands

```bash
# Install dependencies
npm install

# Generate code from hub.yml and parameters.yml
npm run generate

# Build the collector
npm run build

# Run linting
npm run lint

# Run integration tests
npm run test:integration
```

### Execution
Collector bots are invoked via:
- **Caller Pipelines** - Scheduled execution with parameters
- **Manual Triggers** - On-demand execution via UI/API
- **Event-Driven** - Triggered by platform events

## Integration with Platform

### Dataloader Integration
**Handler Location:** `../../auditmation/platform/dataloader/src/processors/collectorbot/`
**Database Table:** `catalog.collector_bot`

### Runtime Integration
- **Hub Modules:** Collector bots invoke Hub module operations
- **AuditgraphDB:** Write collected data as objects with versions
- **Schemas:** Objects conform to schema class definitions
- **Pipelines:** Scheduled via caller pipeline configuration

## Collector Bot Architecture

### Typical Flow
```
Collector Bot
  → Hub Module (via Hub Client)
    → External API/System
  → Transform Data
  → Create/Update Objects in AuditgraphDB
  → Publish Events
```

### Key Components
- **Module Invocation:** Call Hub module operations with connection profiles
- **Data Transformation:** Map external data to schema objects
- **Object Creation:** Create/update objects and versions in PostgreSQL
- **Link Establishment:** Create relationships between objects
- **Event Publishing:** Notify platform of new/changed data

## ZeroBias-Org Specific Configuration

### NPM Package Scopes

**This repository uses the following scopes:**

| Package Type | Scope | Example |
|-------------|-------|---------|
| **Collector bots** (this repo) | `@zerobias-org/collectorbot-*` | `@zerobias-org/collectorbot-avigilon-alta-access` |
| **Hub modules** | `@zerobias-org/module-*` (fallback: `@auditlogic/module-*`) | `@zerobias-org/module-avigilon-alta-access` |
| **Product packages** | `@zerobias-org/product-*` | `@zerobias-org/product-avigilon-alta-access` |
| **Schema packages** | `@auditlogic/schema-*` | `@auditlogic/schema-avigilon-alta-access` |
| **Schema TypeScript** | `@auditlogic/schema-*-ts` | `@auditlogic/schema-avigilon-alta-access-ts` |
| **Hub infrastructure** | `@auditmation/hub-*` | `@auditmation/hub-client` |
| **Utility packages** | `@auditmation/util-*` | `@auditmation/util-collector-utils` |

### Package Naming Convention

**Collector Bot Package Name:**
```
@zerobias-org/collectorbot-<vendor>-<suite?>-<product>
```

**Examples:**
- Directory: `package/avigilon/alta/access/` → `@zerobias-org/collectorbot-avigilon-alta-access`
- Directory: `package/github/github/` → `@zerobias-org/collectorbot-github-github`
- Directory: `package/auditmation/generic/tls/` → `@zerobias-org/collectorbot-auditmation-generic-tls`

**Product Package Name:**
```
@zerobias-org/product-<vendor>-<suite?>-<product>
```

**Note:** Suite is included with hyphen when present!

**Examples:**
- `@zerobias-org/product-avigilon-alta-access`
- `@zerobias-org/product-github-github`
- `@zerobias-org/product-amazon-aws-iam` (for AWS IAM)

### Discovering Product Packages

To find available product packages, check the product bundle:

```bash
npm view @zerobias-org/product-bundle --json | jq '.dependencies'
```

This will show all available products in the ZeroBias ecosystem. Match by the pattern `@zerobias-org/product-<vendor>-<suite?>-<product>`.

### Dependency Priority Rules

When resolving dependencies:

1. **Modules:** Prefer `@zerobias-org/module-*`, fallback to `@auditlogic/module-*`
2. **Products:** Always use `@zerobias-org/product-*` (`@auditlogic/product-*` is deprecated)
3. **Schemas:** Always use `@auditlogic/schema-*` (no alternative)
4. **Infrastructure:** Always use `@auditmation/*` (hub, utils, types)

## Related Documentation

- **[Root README.md](../../README.md)** - Meta-repo overview
- **[ContentArtifacts.md](../../ContentArtifacts.md)** - Content catalog system (Collector Bots section)
- **[platform/dataloader/CLAUDE.md](../../auditmation/platform/dataloader/CLAUDE.md)** - Dataloader system
- **[hub/CLAUDE.md](../../auditmation/hub/CLAUDE.md)** - Hub platform integration
- **[module/CLAUDE.md](../module/CLAUDE.md)** - Hub module development
- **[schema/CLAUDE.md](../schema/CLAUDE.md)** - Schema definitions

## Important Notes

- **Monorepo Structure:** Each collector bot is independently versioned
- **Module Dependencies:** Must reference valid Hub modules
- **Schema Dependencies:** Must reference valid schema packages
- **Idempotency:** Collection should be idempotent where possible
- **Error Handling:** Robust error handling for external API failures
- **Batching:** Process data in batches for large collections
- **Incremental Collection:** Support incremental updates where possible

## Development Guidelines

For comprehensive development rules and patterns, see:
- **[.claude/COLLECTORBOT_RULES.md](.claude/COLLECTORBOT_RULES.md)** - Complete file structure rules, templates, and validation checklists

### Best Practices

1. **Always use Mappers.ts** - Separate data transformation from collection logic
2. **Understand groupId** - Critical for data integrity (see rules document)
3. **Use schema -ts packages** - Import TypeScript interfaces for type safety
4. **Symlink config files** - Never copy `.npmrc` and `.nvmrc`
5. **Keep it simple** - Avoid optional complexity files unless necessary
6. **Validate before commit** - Run `/review-collector` to check all rules

### Quality Assurance

**Automated validation:**
- `/create-collector` automatically runs `/review-collector` at the end
- Validates all aspects before completion
- Reports issues with fix suggestions

**Manual validation:**
```bash
/review-collector [path]
```

Spawns 8 specialized validation agents in parallel:
- Configuration, Schema, Mapping, Pagination, GroupId, Security, Performance, Dependencies
- Reports violations with file:line references and fix suggestions
- Provides compliance score (aim for >90%)

### Reference Implementations

Good examples to study in this repository:
- `package/avigilon/alta/access/` - Multi-resource collector with relationships
- `package/auditmation/generic/tls/` - Simple, clean collector pattern

---

**Last Updated:** 2024-12-23
**Repository:** zerobias-org/collectorbot
**Maintainers:** ZeroBias Community

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zerobias-org) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
