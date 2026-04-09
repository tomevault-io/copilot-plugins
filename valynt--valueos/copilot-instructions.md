## valueos

> These rules are applied to ALL agents across the entire SaaS platform, regardless of tenant or function. They are immutable safety and compliance constraints that cannot be overridden by local rules.

# Global Rules - The Platform "Constitution"

These rules are applied to ALL agents across the entire SaaS platform, regardless of tenant or function. They are immutable safety and compliance constraints that cannot be overridden by local rules.

## Rule Categories

- **systemic_safety**: Critical system protection rules
- **data_sovereignty**: Multi-tenant data isolation rules
- **pii_protection**: Personal information protection rules
- **cost_control**: Resource usage and spending limits
- **audit_compliance**: Logging and audit requirements

## Systemic Safety Rules

### GR-001: Block Dangerous System Commands

**Severity:** Critical | **Enforcement:** Block | **Override:** No

Prevents execution of dangerous system-level commands including:

- SQL destructive operations (DROP TABLE, TRUNCATE, DELETE without WHERE)
- Shell commands (rm -rf, sudo, chmod 777, eval)
- Process manipulation (kill -9, shutdown, reboot)
- Credential exposure patterns

### GR-002: Network Allowlist Enforcement

**Severity:** Critical | **Enforcement:** Block | **Override:** No

Blocks outbound traffic to non-allowlisted domains. Allowlisted domains include:

- Internal services (\*.supabase.co, localhost)
- LLM providers (api.together.ai, api.openai.com, api.anthropic.com)
- Monitoring (_.datadoghq.com)
- CDN (_.cloudflare.com, _.fastly.net)

Blocked domains in production:

- _.github.com, _.githubusercontent.com
- _.pastebin.com, _.ngrok.io, \*.serveo.net

### GR-003: Recursion Depth Limit

**Severity:** High | **Enforcement:** Block | **Override:** Yes (dev only)

Limits recursion depth to prevent infinite loops:

- Development: Max 10 levels
- Staging: Max 7 levels
- Production: Max 5 levels

## Data Sovereignty Rules

### GR-010: Tenant Isolation Enforcement

**Severity:** Critical | **Enforcement:** Block | **Override:** No

Ensures all database operations include tenant_id filter matching the current context tenant. Prevents cross-tenant data access.

### GR-011: Cross-Tenant Data Transfer Block

**Severity:** Critical | **Enforcement:** Block | **Override:** No

Blocks any operation that transfers data between different tenants (copy, move, transfer, migrate, export).

## PII Protection Rules

### GR-020: PII Detection and Redaction

**Severity:** Critical | **Enforcement:** Block | **Override:** No

Detects and blocks PII patterns including:

- Social Security Numbers
- Credit card numbers
- Bulk email lists
- Phone numbers
- Passport/bank account numbers
- Driver's licenses
- Dates of birth
- Healthcare IDs

### GR-021: Logging PII Prevention

**Severity:** High | **Enforcement:** Block | **Override:** No

Prevents PII from being written to logs, traces, debug output, or audit entries.

## Cost Control Rules

### GR-030: Reasoning Loop Step Limit

**Severity:** High | **Enforcement:** Block | **Override:** Yes (dev only)

Limits reasoning loop steps and LLM calls:

- Development: 20 steps, 50 LLM calls
- Staging: 15 steps, 30 LLM calls
- Production: 10 steps, 20 LLM calls

### GR-031: Session Cost Limit

**Severity:** High | **Enforcement:** Block | **Override:** Yes (dev only)

Hard spending caps per session:

- Development: $5.00 max
- Staging: $10.00 max
- Production: $25.00 max
  (Warns at 80% threshold)

### GR-032: Execution Time Limit

**Severity:** High | **Enforcement:** Block | **Override:** Yes (dev only)

Maximum execution time per agent action:

- Development: 60 seconds
- Staging: 45 seconds
- Production: 30 seconds

## Audit Compliance Rules

### GR-040: Action Audit Trail Requirement

**Severity:** Medium | **Enforcement:** Audit | **Override:** Yes (dev only)

Requires audit logging for significant actions:

- create, update, delete, export, import
- approve, reject, submit, finalize, publish
- grant, revoke, login, logout, configure

Must include: requestId, userId, timestamp in metadata.

## Enforcement

All global rules are enforced via Policy-as-Code pattern for consistent development and production behavior. Critical rules cannot be overridden in any environment. High-severity rules may be overridden in development environments only.

## Development Workflow Triggers

Trigger: When starting a multi-package change with CI dependencies.
Action: Run pnpm run typecheck:signal:json and verify baselines match current error counts.
Owner: Human developer.
Success signal: No baseline mismatches detected in telemetry output.

Trigger: Upon encountering tool constraints during implementation.
Action: Manually create blocked files using terminal commands and verify with git add and pnpm run test.
Owner: Human developer.
Success signal: Files created and tests pass without tool errors.

Trigger: Before committing changes that affect CI tooling.
Action: Stage files and run pnpm run typecheck:ratchet:precommit locally to validate.
Owner: System (pre-commit hook).
Success signal: Ratchet passes without script modifications needed.

Trigger: During test implementation phase.
Action: Create test files in parallel with feature files, using shared test utilities.
Owner: Agent (automated test generation).
Success signal: Test coverage reaches 100% for new security controls.

Trigger: At project planning start.
Action: Audit tool availability and document constraints in rules with workarounds.
Owner: Human developer.
Success signal: Constraints documented and referenced in subsequent PRs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Valynt)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Valynt)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
