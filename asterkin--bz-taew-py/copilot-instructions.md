## bz-taew-py

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **bz-taew-py** project - a Python implementation of the BlueZone parking ticket application, showcasing the "Ports and Adapters" architecture pattern. This project builds upon the [bz-taew](https://github.com/asterkin/bz-taew) repository (included as a subtree under `common/`) and uses the [taew-py](https://github.com/asterkin/taew-py) library for the Ports & Adapters framework.

## Development Commands

This project uses UV for dependency management and a root-level Makefile for task automation.

### Testing and Coverage
- `make test-unit` - Run unit tests using unittest discovery from `./test` directory
- `make coverage` - Run full test suite with coverage analysis
- `make erase-coverage` - Clean coverage data
- `make combine-coverage` - Combine coverage data from parallel test runs
- `make report-coverage` - Display coverage report

### Static Analysis
- `make static` - Run all static analysis tools (ruff, mypy, pyright)
- `make ruff-check` - Run ruff linting (excludes `./typings`)
- `make ruff-format` - Run ruff formatting
- `make mypy` - Run MyPy type checking on source and `./bin/bz`
- `make pyright` - Run Pyright type checking on source and `./bin/bz`

### Benchmarks
- `make benchmark` - Run performance benchmarks (currently: ticket storage)

### Development Workflow
- `make sync` - Sync dependencies using `uv sync`
- `make all` - Execute complete pipeline: sync, static analysis, coverage, and benchmarks

### Issue Management Workflow

This project uses custom slash commands for GitHub issue management:

- `/issue-new <label> "<title>" ["<body>"]` - Create new issue and branch
  - Creates GitHub issue, creates branch, pushes to origin, switches to branch
  - Labels: `enhancement`, `bug`, `documentation`

- `/issue-close` - Close current issue (skip build, auto-version bump)
  - Bumps version based on issue label (enhancement→minor, bug→patch, documentation→no bump)
  - Commits changes, pushes, creates PR, merges, closes issue, deletes branch
  - Options: `build` (run `make`), `--major` (force major bump), `--no-bump` (skip version)

**Best Practice:** After significant architectural changes, update CLAUDE.md to document:
- New patterns and conventions
- Breaking changes from previous versions
- Critical knowledge for future development
- Configuration examples reflecting current state

## Architecture

This application implements the **Ports & Adapters (Hexagonal Architecture)** pattern using the [taew-py](https://github.com/asterkin/taew-py) framework (v2.13.0+). The architecture achieves pure separation between domain logic and infrastructure through well-defined interfaces (ports).

### Core Architectural Principles

**1. Pure Dependency Injection**
- All dependencies resolved through taew's `bind()` function
- No manual instantiation except the bind function itself
- Framework infrastructure (Root, CLI routing, etc.) injected automatically via `PortsMapping`

**2. Convention Over Configuration**
- Sensible defaults for framework concerns (project root = `./`, CLI package = `adapters.cli`)
- Application code focuses purely on domain configuration
- Zero boilerplate in user code

**3. Stateless Dependency Resolution**
- `bind()` is a pure function with caching for performance
- Caching is optimization, not semantic state
- No class ceremony, no constructor dependencies

**4. Clean Separation of Concerns**
- **Domain layer**: Pure data structures and value objects (entities, no behavior)
- **Ports layer**: Protocol-based interfaces (contracts for capabilities)
- **Workflows layer**: Business process logic via port orchestration
- **Adapters layer**: Concrete implementations of ports
- **Configuration layer**: Declarative wiring of adapters
- **Framework layer**: Invisible infrastructure (provided by taew)

For deep dive into taew framework architecture, see [taew-py CLAUDE.md](https://github.com/asterkin/taew-py/blob/main/CLAUDE.md).

### Project Structure

```
bz-taew-py/
├── domain/              # BlueZone domain models
│   ├── payment_card.py  # Payment card value object
│   ├── rate.py          # Parking rate value object
│   └── ticket.py        # Parking ticket entity
├── ports/               # Port interfaces for external dependencies
│   ├── for_car_drivers.py         # Interface for car driver operations
│   ├── for_making_payments.py     # Payment processing interface
│   ├── for_parking_inspectors.py  # Parking inspector operations
│   ├── for_storing_rates.py       # Rate storage interface
│   └── for_storing_tickets.py     # Ticket storage interface
├── adapters/            # Adapter implementations
│   ├── cli/             # CLI-specific adapters
│   ├── dir/             # Directory-based storage adapters
│   └── ram/             # In-memory storage adapters
├── workflows/           # Application workflows (use cases)
├── test/                # Unit and integration tests
├── benchmarks/          # Performance benchmarks
│   ├── benchmark_tickets_storage.py  # Storage adapter comparisons
│   └── fake_tickets.py  # Test data generation
├── bin/                 # Executable scripts
│   └── bz               # Main CLI entry point
├── configuration.py     # Port-to-adapter wiring configuration
└── common/              # Shared specifications from bz-taew (subtree)
```

### Benchmarks

**Performance Testing** (`benchmarks/`):
The benchmarks folder contains performance comparisons of different ticket storage configurations, evaluating them across three dimensions:

1. **Configuration Complexity** - How many adapters need to be composed (1-4 rating scale)
2. **Speed** - Write and read performance per ticket
3. **Storage Size** - Total and average file sizes

**Current Configurations Tested:**
- **Pickle** (complexity: 1) - Single adapter, simplest configuration
- **Pickle + zlib** (complexity: 2) - Compressed pickle
- **BytesIO** (complexity: 2) - Binary serialization
- **BytesIO + zlib** (complexity: 3) - Compressed binary
- **JSON** (complexity: 3) - Human-readable format with datetime variants
- **JSON + zlib** (complexity: 4) - Compressed JSON

**Benchmark Results:**
The Pickle serializer emerges as the optimal choice for the current exploratory test setup:
- **Best configuration simplicity** - Single adapter, no variant configuration needed
- **Best speed performance** - Fastest read/write operations
- **Acceptable storage overhead** - Larger file sizes than compressed alternatives, but acceptable trade-off

The benchmark generates 10,000 fake tickets using the Faker library and measures write time, read time, and storage size for each configuration. While compressed formats (zlib) achieve better storage efficiency, they add configuration complexity and performance overhead that outweigh the benefits for this use case.

**Running Benchmarks:**
```bash
make benchmark  # Runs all storage configuration benchmarks
```

### Domain Layer

**Domain Models** (`domain/`):
- `payment_card.py` - Payment card value object (card number, CVV, expiry)
- `rate.py` - Parking rate value object (zone, hourly rate)
- `ticket.py` - Parking ticket entity (registration, zone, payment details, timestamps)

These are pure data structures (value objects and entities) with no behavior or business logic. They represent the core domain concepts that are manipulated by workflows through ports. All validation and business rules are encapsulated within these immutable data structures.

### Ports Layer

**Port Interfaces** (`ports/`):
Defines the contracts that adapters must implement to provide infrastructure capabilities:

- `for_car_drivers.py` - Operations for car drivers (buy parking tickets)
- `for_making_payments.py` - Payment processing capabilities
- `for_parking_inspectors.py` - Ticket verification for inspectors
- `for_storing_rates.py` - Rate persistence interface
- `for_storing_tickets.py` - Ticket persistence interface

All ports use Python protocols for type safety and follow dependency inversion principles.

### Workflows Layer

**Business Process Logic** (`workflows/`):
Implements the application's use cases by orchestrating ports to manipulate domain objects:

- `for_car_drivers/` - Car driver workflows (purchase parking tickets)
- `for_parking_inspectors/` - Inspector workflows (check parking status, verify tickets)

Workflows receive ports as dependencies (via dependency injection) and coordinate them to implement business processes. For example, the "buy ticket" workflow orchestrates:
1. Payment processing port (charge the card)
2. Rate storage port (get parking rate for zone)
3. Ticket storage port (persist the ticket)
4. Current datetime port (timestamp the ticket)

Workflows contain the business logic and rules, while domain objects remain pure data structures. Each workflow also provides a `Configure` class for dependency injection setup.

### Adapter Layer

**Adapter Implementations** (`adapters/`):

- **CLI Adapters** (`adapters/cli/`) - Thin wrappers that rename workflow classes to CLI commands (e.g., `workflows.for_car_drivers.BuyTicket` → `buy_ticket` method). The argparse infrastructure then automatically converts snake_case to kebab-case for CLI invocation (`buy-ticket`). This direct workflow-to-command mapping is a unique capability of taew, eliminating the manual command registration boilerplate found in traditional Python CLI frameworks like argparse or click.
- **Directory Storage** (`adapters/dir/`) - File system-based persistence
- **RAM Storage** (`adapters/ram/`) - In-memory storage for testing/development

Adapters are configured in `configuration.py` using the taew-py framework's port binding system.

### CLI Application

**Entry Point** (`bin/bz`):
The main CLI executable is a minimal shim (~22 lines) that:
1. Imports the `adapters` configuration from `configuration.py`
2. Uses taew-py's `bind()` function to resolve the `Main` interface
3. Executes the main program with command-line arguments

```python
from taew.ports.for_starting_programs import Main
from taew.adapters.launch_time.for_binding_interfaces import bind
from configuration import adapters

_main = bind(Main, adapters=adapters)
_main(sys.argv)
```

The CLI automatically discovers commands from the workflow layer and provides a dynamic command-line interface. This shim is trivial enough to be auto-generated.

### Configuration

**Port Wiring** (`configuration.py`):
Defines a single `adapters` variable that maps port interfaces to their adapter implementations. This configuration is purely application-specific - all framework infrastructure (CLI routing, argument parsing, pretty printing, etc.) is configured automatically by taew's `configure()` function.

```python
from taew.utils.cli import configure

adapters = configure(
    MakingPayments(),                          # Domain adapters
    CarDrivers(_min_euros=Decimal("0.50")),   # with business rules
    ParkingInspectors(),
    CurrentDateTime(),
    Rates(...),                                # and domain data
    Tickets(...),
    variants={...},                            # Type serialization variants
)
```

The `configure()` function:
- Takes application-specific adapter configurations (without `Configure*` prefix for brevity)
- Adds standard CLI infrastructure adapters automatically (BindingInterfaces, CLI, Argparse, PPrint, etc.)
- Uses sensible defaults (project root = `./`, CLI package = `adapters.cli`)
- Returns a complete `PortsMapping` for dependency injection

This design achieves pure separation: `configuration.py` describes **what** the application needs, while taew handles **how** to wire it together.

### Architectural Patterns

**Hexagonal Architecture (Ports & Adapters)**
- Domain logic isolated from infrastructure via protocol-based ports
- Easy substitution of adapters (e.g., swap RAM storage for database storage)
- Multiple implementations per port (RAM vs. directory storage)

**Dependency Inversion Principle**
- Domain depends on port abstractions (protocols), never on concrete adapters
- Adapters depend on ports, implementing the contracts
- All dependencies point inward toward the domain

**Protocol-Based Contracts**
- All ports defined as Python protocols (structural typing)
- Type-safe interfaces without inheritance
- Runtime duck typing with static type checking

**Functional Dependency Injection**
- Pure `bind()` function resolves dependencies at runtime
- Stateless design: caching is performance optimization, not semantic state
- No constructor arguments, no class ceremony

**Type Safety**
- Strict MyPy and Pyright configurations enforce correctness
- Protocol-based ports enable structural type checking
- Custom type variants for serialization (e.g., date formatting)

### Configuration Principles

**Separation of Concerns:**
- **Application code** (`configuration.py`): Domain adapters, business rules, domain data
- **Framework code** (taew): CLI infrastructure, dependency resolution, code introspection
- **Shim code** (`bin/bz`): Minimal glue between configuration and framework

**Naming Conventions:**
- Import adapter `Configure` classes **without** the `Configure` prefix for brevity
  - `Configure as MakingPayments` (not `ConfigureMakingPayments`)
  - This makes configuration read naturally: `configure(MakingPayments(), CarDrivers(), ...)`
- Use `adapters` (not `ports` or `launch_ports`) for the main configuration variable
- Use `bind()` function (not `Bind()` class) for dependency injection

**Configuration Pattern:**
```python
# Pure application configuration - no framework noise
adapters = configure(
    DomainAdapter1(business_rule=value),
    DomainAdapter2(),
    ...,
    variants={CustomType: {...}},  # Optional type serialization
)
```

**Key Benefits:**
- Testing domain logic independently of infrastructure
- Auto-generation capability for entry point shims (`bin/bz`)
- Zero framework boilerplate in application code
- Declarative, self-documenting configuration

### Critical Knowledge for Project Evolution

**When adding new adapters:**
1. Implement the adapter in appropriate directory (`adapters/`, `workflows/`)
2. Create `for_configuring_adapters.py` with `Configure` class
3. Import as short name: `from path import Configure as ShortName`
4. Add to `configure()` call in `configuration.py`

**When modifying configuration:**
- Application-specific changes go in `configuration.py`
- Framework defaults should be changed in taew-py, not here
- The `bin/bz` shim should rarely need modification

**When upgrading taew:**
- Check taew CLAUDE.md for breaking changes
- Update this file's "Recent Architectural Evolution" section
- Verify `configure()` function signature compatibility
- Test that sensible defaults still apply

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/asterkin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/asterkin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
