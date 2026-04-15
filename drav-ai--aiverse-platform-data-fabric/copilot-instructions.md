## aiverse-platform-data-fabric

> - ALL execution units MUST be stateless

# AIVerse Data Fabric Domain - Cursor Rules

## Domain Constraints

### Stateless Execution (CRITICAL)
- ALL execution units MUST be stateless
- NO internal caching, sessions, or accumulated state
- Units can be terminated at ANY point with NO side effects
- Use Protocol-based dependency injection for all external dependencies

### No Orchestration (CRITICAL)
- Execution units MUST NOT call other execution units
- NO loops, retries, or conditionals within units
- MCOP handles all scheduling and sequencing
- NO workflow logic in this domain

### Domain Independence (CRITICAL)
- This domain can be FULLY REMOVED without breaking the Control Plane
- NO cross-domain imports (model, agent, business, etc.)
- ALL interactions via published Control Plane interfaces only
- NO direct Kubernetes or cluster references

## Code Style

### Python
- Use Python 3.11+ features (type hints, dataclasses, etc.)
- All dataclasses should be frozen=True for immutability
- Use Protocol classes for dependency injection
- Document failure modes in docstrings

### File Organization
- Execution units: `execution_units/eu_XX_<name>.py`
- Registry cards: `registry_cards/eu_XX_<name>_card.json`
- Feedback signals: `feedback_signals/<type>_<name>.json`
- Contracts: `schemas/contracts.py`

## Testing

- Mock ALL Protocol dependencies in unit tests
- Verify failure modes produce correct error codes
- DO NOT test orchestration (there is none)
- Integration tests verify registry card registration

## Forbidden Patterns

1. `import kubernetes` or `from kubernetes import` - Use MCOP adapter
2. `@retry` or retry loops - MCOP handles retries
3. `asyncio.gather` for parallel EU calls - No orchestration
4. Direct SQL/database access - Use Registry/Ledger protocols
5. Environment-specific code (cluster IDs, hardware specs)
6. Cross-domain imports (aiverse_platform_training, etc.)

## ADR Reference

Always consult `docs/adr/PHASE3-DATA-FABRIC-PLUGIN-SPEC.md` for authoritative definitions.

## Git Workflow

- Feature branches from develop
- Commit frequently with descriptive messages
- Push immediately after commits
- PR to merge feature -> develop -> main

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/drav-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
