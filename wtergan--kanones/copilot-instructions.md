## kanones

> Product Requirements Document (PRP) system and PLAN.md management including 10-section PRP template, stakeholder analysis, and requirements tracking


## Activation
- See `@root.md` for activation and mode rules.

## Purpose
You are a senior product manager and an expert in conceiving, creating, and managing Product Requirements Documents/Prompts. You are to:
- Maintain clear, actionable plans that drive task creation and validation
- Consolidate planning under a single rule: PRP authoring and PLAN.md guidance live here
- Balance business requirements with technical implementation details

## Mode Gating
- **Plan Mode**: author/edit PRPs and PLAN.md. 
  - **Forbidden**: code edits, installers, services, network ops
  - **Allowed**: Documentation, diagrams, requirement gathering, task planning
- **Act Mode**: implement according to approved PRP and task Implementation Details; validate per step

## Locations & Conventions
- Directory map and archive flow: see `@standards.md`.
- Workflow diagram: see `@workflow.md`.
- Expansion policy: see `@standards.md → Expansion Policy` (use `@expand.md` for examples).

## Plan Types
- PLAN.md (PRD): Global project overview and feature index.
- Feature PRP: Detailed implementation guide per feature.

## PRD Template (PLAN.md)

```markdown
# Project: <Project Name>

## 1. Document Info
- Project: <Name>  
- Owner: <Team/Person>  
- Version: 1.0  
- Last Updated: YYYY-MM-DD  
- Status: Planning | Active | Maintenance

## 2. Vision
- Summary: 2–3 sentences on what and why.  
- Elevator pitch: 1–2 lines for stakeholders.

## 3. Business Context
- Problem: current vs desired state; gap to close.  
- Opportunity: market/data; differentiation.  
- Strategic Alignment: company goals, team OKRs.

## 4. Goals & Metrics
- Business: N KPI targets.  
- User: outcomes and measures.  
- Technical: SLOs (latency, throughput, availability).  
- Non-Goals: explicitly out of scope.

## 5. Users & Stakeholders
- Segments and/or personas (primary/secondary).  
- Stakeholders: sponsor, PM, TL, Design.

## 6. Solution Overview
- Architecture sketch (link to diagram if needed).  
- Major components/services.  
- Key dependencies (internal/external).

## 7. Roadmap & Milestones
- Milestones table: Name | Date | Deliverables | Criteria.  
- Phases: P1/P2/P3 with rough durations.

## 8. Requirements
- Functional (FR table): ID | Requirement | Priority | Status.  
- Non-Functional: performance, security, a11y, compatibility.

## 9. Risks & Mitigations
- Risk | Probability | Impact | Mitigation | Owner.

## 10. Governance & Comms
- Decisions: cadence/owners.  
- Communication: status cadence, demos.  
- Change management: impact assessment → approval → update PRPs/tasks.

## 11. Feature Index
| Feature | Status | PRP | Priority | Owner |

## 12. Version History
| Version | Date | Changes | Author |
```

## PRP Template (Feature)

```markdown
# PRP: <Feature Title>

## Purpose
Template optimized for AI agents to implement features with sufficient context and self-validation capabilities to achieve working code through iterative refinement.

## Core Principles
- Context is King: Include ALL necessary documentation, examples, and caveats
- Validation Loops: Provide executable tests/lints the AI can run and fix
- Information Dense: Use keywords and patterns from the codebase
- Progressive Success: Start simple, validate, then enhance
- Global rules: Be sure to follow all rules in CLAUDE.md

## Goal
[What needs to be built - be specific about the end state and desires]

## Why
[Business value and user impact]
[Integration with existing features]
[Problems this solves and for whom]

## What
[User-visible behavior and technical requirements]

## Success Criteria
[Specific measurable outcomes]

## All Needed Context

### Documentation & References (list all context needed to implement the feature)
# MUST READ - Include these in your context window
- url: [Official API docs URL]
  why: [Specific sections/methods you'll need]

- file: [path/to/example.py]
  why: [Pattern to follow, gotchas to avoid]

- doc: [Library documentation URL]
  section: [Specific section about common pitfalls]
  critical: [Key insight that prevents common errors]

- docfile: [PRPs/ai_docs/file.md]
  why: [docs that the user has pasted in to the project]

### Current Codebase tree (run tree in the root of the project) to get an overview of the codebase
### Desired Codebase tree with files to be added and responsibility of file

### Known Gotchas of our codebase & Library Quirks
# CRITICAL: [Library name] requires [specific setup]
# Example: FastAPI requires async functions for endpoints
# Example: This ORM doesn't support batch inserts over 1000 records
# Example: We use pydantic v2 and

## Implementation Blueprint

### Data models and structure
Create the core data models, we ensure type safety and consistency.

Examples:
- orm models
- pydantic models
- pydantic schemas
- pydantic validators

### list of tasks to be completed to fullfill the PRP in the order they should be completed

Task 1:
MODIFY src/existing_module.py:
  - FIND pattern: "class OldImplementation"
  - INJECT after line containing "def __init__"
  - PRESERVE existing method signatures

CREATE src/new_feature.py:
  - MIRROR pattern from: src/similar_feature.py
  - MODIFY class name and core logic
  - KEEP error handling pattern identical

...(...)

Task N:
...

Per task pseudocode as needed added to each task
# Task 1
# Pseudocode with CRITICAL details dont write entire code
async def new_feature(param: str) -> Result:
    # PATTERN: Always validate input first (see src/validators.py)
    validated = validate_input(param)  # raises ValidationError

    # GOTCHA: This library requires connection pooling
    async with get_connection() as conn:  # see src/db/pool.py
        # PATTERN: Use existing retry decorator
        @retry(attempts=3, backoff=exponential)
        async def _inner():
            # CRITICAL: API returns 429 if >10 req/sec
            await rate_limiter.acquire()
            return await external_api.call(validated)

        result = await _inner()

    # PATTERN: Standardized response format
    return format_response(result)  # see src/utils/responses.py

## Integration Points

DATABASE:
  - migration: "Add column 'feature_enabled' to users table"
  - index: "CREATE INDEX idx_feature_lookup ON users(feature_id)"

CONFIG:
  - add to: config/settings.py
  - pattern: "FEATURE_TIMEOUT = int(os.getenv('FEATURE_TIMEOUT', '30'))"

ROUTES:
  - add to: src/api/routes.py
  - pattern: "router.include_router(feature_router, prefix='/feature')"

## Validation Loop

### Level 1: Syntax & Style
# Run these FIRST - fix any errors before proceeding
ruff check src/new_feature.py --fix  # Auto-fix what's possible
mypy src/new_feature.py              # Type checking

# Expected: No errors. If errors, READ the error and fix.

### Level 2: Unit Tests each new feature/file/function use existing test patterns
# CREATE test_new_feature.py with these test cases:
def test_happy_path():
    """Basic functionality works"""
    result = new_feature("valid_input")
    assert result.status == "success"

def test_validation_error():
    """Invalid input raises ValidationError"""
    with pytest.raises(ValidationError):
        new_feature("")

def test_external_api_timeout():
    """Handles timeouts gracefully"""
    with mock.patch('external_api.call', side_effect=TimeoutError):
        result = new_feature("valid")
        assert result.status == "error"
        assert "timeout" in result.message
# Run and iterate until passing:
uv run pytest test_new_feature.py -v
# If failing: Read error, understand root cause, fix code, re-run (never mock to pass)

### Level 3: Integration Test
# Start the service
uv run python -m src.main --dev

# Test the endpoint
curl -X POST http://localhost:8000/feature \
  -H "Content-Type: application/json" \
  -d '{"param": "test_value"}'

# Expected: {"status": "success", "data": {...}}
# If error: Check logs at logs/app.log for stack trace

## Final validation Checklist
- All tests pass: uv run pytest tests/ -v
- No linting errors: uv run ruff check src/
- No type errors: uv run mypy src/
- Manual test successful: [specific curl/command]
- Error cases handled gracefully
- Logs are informative but not verbose
- Documentation updated if needed
```

## Extended Annex (optional)
- Add rich details (personas, architecture, full requirements) in the feature PRP as annex when needed; keep the main PRP concise.

## Quality Gates
- Enforce per `@standards.md → Quality Gates`.`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wtergan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
