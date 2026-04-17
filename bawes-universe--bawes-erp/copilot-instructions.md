## bawes-erp

> Keep in mind that this project is a Nestjs backend, for frontends in other projects. No frontend will be implemented in this project.

Keep in mind that this project is a Nestjs backend, for frontends in other projects. No frontend will be implemented in this project.

# Required process
- Follow test driven development process. Before writing any code, write the test first.
- Confirm the test passes before delivering your work as completed. Keep improving the tests until they pass.
- Do not delete existing tests and code just to make things pass. Deletion should only happen when it is absolutely necessary or when it is a part of a refactoring.

# Development Guidelines
- Project docs are under /docs
- We use Github actions for publishing swagger as SDK for frontend usage
- SDK publishing is to a regular github repo, no registry, not github packages or npm registry
- Keep changes small and focused and incremental
- Keep code clean and readable, simple and easy to understand
- Keep code scalable and easy to scale
- Test your work and confirm it works before presenting it

# SDK Development Workflow
1. Feature Branch Development:
   - Create feature branch: `git checkout -b feature/my-feature`
   - SDK automatically generated on push
   - SDK branch matches feature branch name
   - Version format: `0.0.0-feature-name-timestamp`

2. SDK Testing:
   - Frontend can use branch-specific SDK
   - SDK updates on every push to feature branch
   - No interference with production SDK
   - Manual SDK generation via workflow dispatch

3. Branch Strategy:
   - feature/* -> Creates matching SDK branch
   - dev -> Updates dev SDK branch
   - main -> Updates production SDK with versioning
   - Each branch gets its own SDK branch

4. Version Management:
   - Feature branches: `0.0.0-branch-timestamp`
   - Dev branch: `0.0.0-dev-timestamp`
   - Main branch: Semantic versioning
   - Breaking changes trigger major version bump

# Testing Guidelines
1. Test File Organization:
   - Co-locate test files with source files
   - Use `.spec.ts` for unit tests, `.e2e-spec.ts` for E2E
   - Group related tests in describe blocks
   - Clear, descriptive test names
   - Test DTOs through controller/service integration tests, not separately

2. Test Structure:
   - Follow Arrange-Act-Assert pattern
   - One behavior per test
   - Use beforeEach for setup
   - Clean up in afterEach when needed
   - Mock external dependencies
   - Only create separate DTO tests for custom validation or transformation logic

3. Coverage Requirements:
   - 80% minimum for critical paths
   - 60% minimum overall coverage
   - 100% for permission code
   - Test success and error cases
   - Include edge cases
   - Ensure DTOs are covered through controller/service tests

4. Database Testing:
   - Use test database
   - Clean before each test
   - Use transactions
   - Mock for unit tests
   - Real DB for E2E

5. Authentication Testing:
   - Test auth states
   - Verify tokens
   - Check permissions
   - Test refresh flows
   - Test expiration

6. Error Handling:
   - Test all errors
   - Verify messages
   - Check status codes
   - Test validation
   - Verify logging

7. Code Quality:
   - No test duplication
   - Use test helpers
   - Keep maintainable
   - Document complexity
   - Follow style guide

# RBAC Implementation Guidelines
- Technical Implementation:
  - Permissions use globally unique power-of-2 bitfields (1, 2, 4, 8, 16, etc.)
  - Permission bitfields are automatically assigned on discovery
  - JWT contains combined permission bitfield as string
  - Permission checks use Decimal.js for precise bitwise operations
  - Each permission gets next power of 2 as its bitfield
  - Supports over 100k unique permissions via Decimal(40,0)

- Permission Format: `category.action` (e.g., users.create, roles.update)
  - Category: lowercase plural noun (users, roles, permissions)
  - Action: lowercase verb (create, read, update, delete)
  - Examples: users.create, roles.update, permissions.manage

- Categories (users.*, roles.*, permissions.*) are for organization only
- Categories in code must match database categories
- Super admin role automatically gets all permissions
- System roles cannot be modified through API
- Cache permission checks for 5 minutes
- Clear permission cache when roles change

# Permission Categories
- system.* - System-level operations
- users.* - User management
- roles.* - Role management
- permissions.* - Permission management
- audit.* - Audit log operations

# Common Actions
- create - Create new resources
- read - View/list resources
- update - Modify existing resources
- delete - Remove resources
- manage - Full control over resources
- assign - Assign resources to others
- approve - Approve/reject operations

# Financial Guidelines
- All financial calculations must use decimal.js library
- Never use native JavaScript floating-point arithmetic for monetary values
- Always import Decimal from 'decimal.js' when handling currency
- Enforce strict type checking for all monetary values
- Monetary values should always be of type Decimal
- Validate decimal places (max 2 for standard currency)
- Cache Decimal instances outside loops when possible
- Use decimal.js methods instead of native JavaScript methods for precision
- Implement proper error handling and logging for financial calculations
- Regularly review and update decimal precision requirements

# Test-Driven Development
- Follow test driven development process. Before writing any code, write the test first.
- Confirm the test passes before delivering your work as completed. Keep improving the tests until they pass.
- Do not delete existing tests and code just to make things pass. Deletion should only happen when it is absolutely necessary or when it is a part of a refactoring.

# Test Verification Requirements
- ALWAYS run `npm run test` after making any code changes
- ALWAYS run `npm run test:cov` after completing changes to verify coverage
- NEVER ignore failing tests - fix them immediately
- Maintain minimum 80% coverage for critical paths
- Maintain minimum 60% overall coverage
- Document any coverage exceptions with clear reasons

# Test Failure Protocol
1. When a test fails:
   - Stop and analyze the failure immediately
   - Fix the failing test before proceeding
   - Verify the fix with both `npm run test` and `npm run test:cov`
   - Document the fix if it reveals a systemic issue

2. Database test failures:
   - Ensure proper cleanup between tests
   - Verify foreign key constraints are handled
   - Check transaction isolation
   - Confirm test database is being used

3. Coverage failures:
   - Identify areas below threshold
   - Add missing test cases
   - Document complex or untestable code
   - Never skip coverage verification

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BAWES-Universe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
