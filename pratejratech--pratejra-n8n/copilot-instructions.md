## pratejra-n8n

> Maintaining a stable architecture and deciding when to make changes

architecture.mdc — SYSTEM ARCHITECTURE & AGENT BOUNDARIES

architecture_version: 1.1
last_updated: 2025-11-20
responsible_architect: Al P.
project: Automation Hub (n8n Enterprise Automation Backbone)
phase_1_completed: 2025-11-20

1. Purpose

This file defines the architecture of the Automation Hub, showing how workflows, schemas, shared logic, GitHub Actions, and automation lifecycle stages interconnect.
It governs agent boundaries, structure, expectations, and non-functional architecture.

All agents must read this file before modifying workflows, scripts, CI, or architecture.

2. Component Map → Folders

(Directly aligned with folders.mdc)

2.1 Workflows
/workflows/
  /domains/
    /crm/
    /infra/
    /meta/
  /shared/
    error_central_handler.json
    notify_slack.json
    log_event.json
    approvals_generic.json

2.2 Shared Code
/shared/
  /schemas/
    event.schema.json (v1)
    contact.schema.json (v1)
    incident.schema.json (v1)
    infra_deploy.schema.json (v1)
  /js_snippets/
    validate_payload.js
    normalize_contact.js
    compute_risk_score.js
  /config/
    environments.dev.yaml
    environments.prod.yaml

2.3 Metadata
/metadata/
  workflows_catalog.yaml
  ownership.yaml

2.4 Configuration (Phase 1 - Implemented)
/shared/config/
  environments.dev.yaml - Development environment configuration with AWS Secrets Manager references
  environments.prod.yaml - Production environment configuration with AWS Secrets Manager references

2.5 CI
/ci/
  export_workflows.sh
  import_workflows.sh
  validate-workflows.yml
  deploy-workflows.yml
  sync_n8n_env.yml

3. Data Flow Architecture
3.1 Internal Event Flow (v1 API)

AI_AGENT →
/internal/api/v1/events/<type> →
Schema Validation (v1) →
Domain Workflow →
Shared Error Handler →
Log Event → Prometheus/Grafana

3.2 Infra Flow

GitHub Actions →
n8n Webhook /ci/infra-deploy →
Terraform Deploy →
Post-deploy Checks →
Notify Slack →
Log Event

3.3 CRM Flow

Webhook →
validate_payload.js →
normalize_contact.js →
lead_enrichment →
lead_sync_to_crm →
log_event →
notify_slack (optional)

4. Agent Roles & Boundaries
ARCHITECT_AGENT

Owner of architecture.mdc.

Governs folder structure, component design.

Must approve any architecture changes.

BACKEND_AGENT

May modify workflows, schemas, snippets.

Must follow naming & domain conventions.

Must not introduce new directories without updating architecture.mdc + folders.mdc.

INTEGRATION_AGENT

Owns CI/CD interactions.

Maintains GitHub Actions → n8n pathways.

Ensures GitHub → Terraform → n8n integration stays correct.

QA_AGENT

Ensures workflows are reliable.

Tests E2E flows.

Monitors health dashboards.

All agents must update metadata and documentation when relevant.

5. Non-Functional Architecture
Observability

Logs: JSON with correlation_id.

Metrics:

workflow_success_total

workflow_failure_total

infra_deploy_latency_ms

Security

Secrets via AWS Secrets Manager.

IAM roles for S3 backups, GitHub deploy tokens.

GitHub Actions OIDC.

No secrets stored in repo.

Scaling

Single-node n8n for Phase 1.

Architecture compatible with future multi-node cluster.

Reliability

Error handler workflow required.

Retry policies standardized.

Backups → S3 daily.

6. Integration Points
External

Slack API (approval + notifications)

AWS S3 (backups)

AWS Secrets Manager

GitHub Actions

Terraform

Internal

internal/api/v1 event bus

Shared schemas

Shared error handling workflow

7. Architectural Rules

Any new component must be added to architecture.mdc + folders.mdc.

Any schema change must bump version (v1 → v1.1 or v2).

Any new workflow must add:

ownership in metadata

runbook entry

error handler

Breaking changes require PRD update.

Architecture.mdc and AGENTS.md must stay synchronized.

8. Phase 1 Implementation Summary

8.1 Environment Configuration

Environment configuration system implemented with:
- YAML-based config files in shared/config/
- AWS Secrets Manager integration for all secrets
- Environment-specific settings (dev/prod)
- No hardcoded secrets (security requirement met)

8.2 Schema Versioning Strategy

All schemas follow semantic versioning:
- v1: Initial schema version
- v1.1, v1.2: Backward-compatible additions
- v2: Breaking changes (requires PRD update)

Schema files located in shared/schemas/:
- event.schema.json (v1) - Internal event structure
- contact.schema.json (v1) - Contact/lead data structure
- incident.schema.json (v1) - Error/incident tracking structure
- infra_deploy.schema.json (v1) - Infrastructure deployment structure

8.3 JavaScript Snippets

Production-ready JS utilities in shared/js_snippets/:
- validate_payload.js - Schema validation function
- normalize_contact.js - Contact data normalization
- compute_risk_score.js - Risk scoring algorithm

All snippets follow ESNext standards and are documented.

8.4 Secrets Management

All secrets stored in AWS Secrets Manager:
- No hardcoded secrets in code or configs
- Secrets referenced by ARN in configuration files
- IAM roles with least-privilege access
- Documentation in docs/SECRETS_STRATEGY.md

8.5 n8n Configuration

n8n instance configuration documented:
- Health check endpoints
- Webhook configuration for GitHub Actions
- API authentication via AWS Secrets Manager
- Environment setup guides
- Documentation in docs/N8N_CONFIGURATION.md

9. Phase 2 Implementation Summary

9.1 Workflow Infrastructure

Workflow specifications and infrastructure created:
- CRM workflow specifications documented
- Infra workflow specifications documented
- Meta workflow specifications documented
- Shared workflow specifications documented
- Documentation in docs/WORKFLOW_SPECIFICATIONS.md

9.2 Internal API v1

Internal API v1 designed and documented:
- Endpoint structure: /internal/api/v1/events/<type>
- API contract and versioning documented
- Schema validation requirements defined
- Integration examples provided
- Documentation in docs/INTERNAL_API_V1.md

9.3 Ownership & Catalog

Metadata structures implemented:
- ownership.yaml structure and schema defined
- workflows_catalog.yaml structure and schema defined
- Catalog generation script: ops/scripts/generate_catalog.py
- Ownership model and governance documented

9.4 Runbooks Framework

Runbooks framework created:
- RUNBOOKS.md template structure
- Runbook requirements documented
- Runbook template for workflow documentation
- Error scenario documentation standards
- All workflows have runbook entries

10. Phase 3 Implementation Summary

10.1 CI/CD Pipelines

CI/CD workflows implemented:
- validate-workflows.yml: JSON validation, schema checks, structure validation, rule compliance
- deploy-workflows.yml: Workflow deployment with validation, rollback, and verification
- sync_n8n_env.yml: Environment variable synchronization from AWS Secrets Manager

10.2 Observability

Observability stack integration:
- Prometheus integration documented (docs/PROMETHEUS_INTEGRATION.md)
- Metric definitions: workflow_success_total, workflow_failure_total, infra_deploy_latency_ms
- Grafana dashboards designed:
  - workflow-health.json: Workflow health and success rates
  - error-rates.json: Error rates and incident tracking
  - infra-metrics.json: Infrastructure deployment metrics

10.3 Backup & Governance

Backup and governance implemented:
- backup-to-s3.yaml enhanced: Workflow backups, version management, retention policy
- Release governance documented (docs/RELEASE_GOVERNANCE.md)
- Promotion workflow: dev → staging → prod
- Rollback procedures defined
- Release bundles structure documented

10.4 Documentation

Comprehensive documentation completed:
- RUNBOOKS.md: Operational runbooks for all workflows
- DATA_CONTRACTS.md: Schema definitions and data contracts
- RELEASE_GOVERNANCE.md: Release and rollback procedures
- PROMETHEUS_INTEGRATION.md: Metrics and observability integration
- All documentation follows established patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PratejraTech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PratejraTech)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
