## reply-express-api

> This document outlines the standard workflow process for developing features in the Reply Express API codebase, which


# Reply Express API Workflow Rules (Revised)

This document outlines the standard workflow process for developing features in the Reply Express API codebase, which
follows a CQRS/Event Sourcing architecture pattern.

## Development Lifecycle

### 1. Planning Phase

- **Specification Creation**: All features require a detailed specification document in the `/specs` directory.
    - These specifications should outline implementation details, API endpoints, commands, events, etc.
    - No code should be written until the specification is approved.
    - The specification should follow the existing pattern in other spec files.
    - Include a detailed task list with clear dependencies between tasks.

- **Task Dependency Documentation**:
    - Clearly document which tasks depend on others.
    - For complex features, include a visual dependency graph or numbered sequence.
    - Define task states: Not Started, In Progress, Implemented, Verified.

- **Approval Process**: The plan must be explicitly approved before moving to implementation.
    - Clarify any questions or concerns about the plan before implementation.
    - Update the plan if any issues are discovered during review.

### 2. Implementation Phase (TDD Workflow)

#### Systematic File-by-File Approach

1. **Test-First Development**:
    - Write tests before implementation code for each component.
    - Tests should be written one file at a time.
    - Ensure tests reflect the behavior described in the specification.
    - Run tests to verify they fail appropriately before implementation.
    - Document the expected failure to confirm the test is valid.

2. **Explicit Task Checkpoints**:
    - ✅ Write test for feature
    - ✅ Run test to verify it fails (expected failure)
    - ✅ Implement feature
    - ✅ Run test to verify it passes
    - ✅ Perform code quality checks
    - ✅ Get explicit verification before proceeding

3. **One-File-at-a-Time Implementation**:
    - Complete one file entirely before moving to the next.
    - After implementing a file, verify it with the relevant test(s).
    - Fix any issues before proceeding to the next file.
    - Get explicit verification before moving to the next file.
    - Never mark a task as complete until it passes all verification steps.

4. **CQRS Implementation Order**:
    - Follow this sequence when implementing CQRS components:
        1. Commands in `lib/reply_express/accounts/commands/`
        2. Events in `lib/reply_express/accounts/events/`
        3. Aggregates in `lib/reply_express/accounts/aggregates/` (business logic)
        4. Projectors in `lib/reply_express/accounts/projectors/` (update read models)
        5. Projections in `lib/reply_express/accounts/projections/` (schema definitions)
        6. Context functions in respective context modules (public API)
        7. Controllers in `lib/reply_express_web/controllers/api/`
        8. JSON views for API responses

#### Task State Definitions

- **Not Started**: Task has not been attempted
- **In Progress**: Tests written but implementation incomplete
- **Implemented**: Implementation complete but not verified
- **Verified**: Implementation verified with tests and review

#### Structured Verification Steps

Before marking a task complete, use this verification checklist:

1. **Test Verification**:
    - All tests for this file pass
    - Tests run without warnings
    - Tests are comprehensive (success and failure cases)
    - Test coverage is adequate

2. **Code Quality Checks**:
    - Run `mix credo --strict` with no issues
    - Run `mix format` to ensure proper formatting
    - Address all warnings - tests run with `--warnings-as-errors`
    - Code follows the Elixir Style Guide

3. **Documentation**:
    - Add appropriate documentation for modules and functions
    - Include @moduledoc and @doc attributes where appropriate
    - Add @type and @spec definitions for modules and functions that represent public APIs and reusable data structures
      or that define behaviours
    - Ensure comments are clear and helpful

4. **Functionality**:
    - All specified functionality is implemented
    - No regressions in existing functionality
    - Feature works as described in specification

## Communication Protocol

- **Implementation Progress**:
    - Verify with stakeholders after completing each file
    - Clearly communicate the current task state (Not Started, In Progress, Implemented, Verified)
    - Document any challenges or questions that arise during implementation

- **Deviations from Plan**:
    - If implementation needs to differ from the specification, discuss before proceeding
    - Document reasons for deviation and get approval for changes

- **Completion Checklist**: Before finalizing a feature, ensure:
    - All tests pass with no warnings
    - Code follows style guide
    - Documentation is complete
    - All specified functionality is implemented
    - All tasks have reached the "Verified" state

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bponghneng) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
