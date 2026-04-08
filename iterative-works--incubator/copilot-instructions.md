## incubator

> We are working according to a plan, always.

# YNAB Importer Project Guidelines

# Disciplined Development Approach

We are working according to a plan, always.

We follow a disciplined development process. This is important. We complete only one step of the plan at a time. After each step is done, we stop to review our work and document our progress.

If you encounter a problem that proves difficult to resolve after an initial attempt, do not continue struggling alone. Reach out for assistance. Remember that you have experienced developers as colleagues who can provide guidance. Always ask for help when facing unclear or complicated solutions rather than attempting to solve challenging problems by yourself.

We are working according to principles in [principles.md](./ai-context/principles/principles.md).

You can find the description of our architecture in [architecture.md](./ai-context/architecture/architecture.md).

Be sure to read both of these files before starting to work on coding tasks.

## Build & Test Commands

We usually use metals for development. It is useful to run sbt server separately in a console, and than access the server using `sbtn`.

- **Compile**: `sbtn compile`
- **Run application**: `sbtn reStart`
- **Run tests**: `sbtn test`
- **Run specific test**: `sbtn "transactions/testOnly works.iterative.incubator.transactions.infrastructure.PostgreSQLTransactionRepositorySpec"`
- **Run with specific test**: `sbtn "ynab-it/testOnly *PostgreSQLTransactionRepositorySpec -- -t 'should save and retrieve a transaction'"`
- **Format code**: `sbtn scalafmtAll`

### Integration Testing

Integration tests use TestContainers to spin up a PostgreSQL database in a Docker container. The database schema is managed using Flyway migrations, ensuring consistency between tests and production environments. Before each test, Flyway cleans the database and applies all migrations to create a fresh schema.

## Application URLs

The application provides multiple UI modules accessible at these URLs:

- **Transactions**: `/transactions` - View and manage transaction imports and processing
- **Source Accounts**: `/source-accounts` - Manage bank account connections
  - List accounts: `/source-accounts`
  - Add account: `/source-accounts/new`
  - View account details: `/source-accounts/{id}`
  - Edit account: `/source-accounts/{id}/edit`

## Development Workflow

Our development process follows a disciplined Behavior-Driven Development (BDD) approach with Gherkin scenarios as the central driving force throughout the entire development lifecycle.

## Core Workflow Phases

1. **Change Request**: Formal documentation of proposed changes (CR-XXXX)
2. **Feature Specification & Scenario Definition**: Detailed requirements captured as Gherkin scenarios
3. **Business Value Decomposition**: Prioritization of scenarios by business value
4. **Feature Planning**: Technical implementation planning based on scenarios
5. **Scenario-Driven Implementation**: Development guided by scenarios using a Functional Core approach
6. **Scenario-Based Testing**: Multi-level verification of scenario implementation
7. **Deployment**: Phased release with scenario-based feature flags

## Implementation Principles

- **BDD-Driven Functional Core**: Design complete domain model as pure functional core first
- **Mock-First Development**: Create comprehensive mocks before real implementations
- **UI Validation**: Build and validate UI components with mock implementations before infrastructure
- **Strict Separation**: Maintain clear boundaries between pure domain logic and side-effecting code
- **Scenario Traceability**: All code changes must trace back to specific scenarios

## Working with AI Agents

- Always provide Gherkin scenarios as primary context
- Request scenario-specific implementations and tests
- Have AI map responses to specific scenario steps
- Request multi-level testing (domain, UI, E2E)

## Key Templates and Documentation

The workflow is supported by 11 standardized templates covering each phase of development, from Change Requests to Scenario Coverage Reports.

> For complete details, refer to the full [Development Workflow](ai-context/workflows/development_workflow.md) document including implementation guides and templates.

### Pre-commit/Pre-PR Checklist

We are using Github, repository iterative-works/incubator. Each set of changes should be tested locally before pushing to the remote repository and creating a pull request. A standard review process is followed to ensure code quality and consistency.

Before committing changes or creating a pull request, always run the following commands to ensure your code is clean and working properly:

1. `sbtn scalafmtAll` - Format code according to our style guide
2. `sbtn clean` - Clean all compiled artifacts to ensure a fresh build
3. `sbtn compile` - Compile the code and verify there are no compiler warnings
4. `sbtn test` - Run all tests to ensure everything is working correctly
5. `sbtn fio-it/test` - Run integration tests separately

This cycle helps catch issues early and ensures that our codebase remains clean and maintainable.

## Code Style Guidelines
- **Architecture**: Follow Functional Core/Imperative Shell pattern (see principles.md and ynab-importer/doc/architecture.md)
- **Formatting**:
  - 4-space indentation, 100 column limit
  - Use new Scala 3 syntax without braces
  - End markers for methods with 5+ lines
- **Imports**: Group imports by package, sort alphabetically
- **Naming**:
  - Domain concepts in CamelCase reflecting ubiquitous language
  - Methods express intent (e.g., `findBySourceAccount` vs `getByAccount`)
- **Types**: Full type annotations for public APIs, immutable by default
- **Error Handling**: Use ZIO effects for error tracking, never throw exceptions
- **Performance Optimizations**:
  - Use caching for repeated database lookups (prefer ZIO Ref for thread-safety)
  - Clear caches at appropriate boundaries to avoid stale data
  - Document caching behavior in comments and test thoroughly
- **Testing**: Write tests first, focus on domain invariants
- **Comments**: Explain "why", not "what" - code should be self-documenting

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iterative-works)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/iterative-works)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
