## ros-plan

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ROS-Plan is a comprehensive ROS 2 toolchain that provides:
1. **Declarative plan format** - YAML-based node configuration with socket/link abstraction
2. **Plan compiler** - Transforms plans into executable ROS 2 launch files
3. **Runtime system** - Process management with graceful shutdown, restart policies, and dynamic parameter updates
4. **Metadata extraction** - launch2dump tool for analyzing existing launch files

The project consists of both Rust (core compiler and runtime) and Python (launch file integration) components.

## Project Status

**Current Phase**: Phase 12 (Launch-to-Plan Conversion) - In Progress
- ✅ Phase 8 Complete: Metadata Tracking (13 tests)
- ✅ 440 tests passing (330 Rust + 25 launch2dump + 9 ros2-introspect + 85 launch2plan)
- ✅ Zero warnings (clippy + compiler)
- ✅ Dependency checking with explicit error messages
- ✅ 54 features implemented across 11 phases (core system)

**Phase 12 Implementation Progress**:
- ✅ Phase 1-5: Foundation, introspection, socket inference, argument handling, plan generation
- ✅ Phase 6: Conditional branch exploration with `when` clauses
- ✅ Phase 7: Include handling & plan hierarchy (8 tests)
- ✅ Phase 8: Metadata tracking with TODO completion detection (13 tests)
- ⏳ Phase 9-12: Validation, end-to-end testing, QoS, recursive conversion (not yet started)
- See `book/src/launch2plan.md` for complete design and progress tracking

## Core Architecture

### Rust Crates (Workspace Structure)

1. **ros-plan-format** - Data structures and parsing for ROS plan files (YAML format)
   - Handles node definitions, links, sockets, parameters, QoS settings
   - Key types: Plan, Node, Link, NodeSocket, PlanSocket
   - Custom YAML tags: !pub, !sub, !pubsub, !i64, !f64, !bool, etc.

2. **ros-plan-compiler** - Core compilation logic
   - Main entry point: `Compiler::compile()`
   - Evaluation, linking, program generation
   - Lua integration for dynamic expressions
   - Link resolution and validation

3. **ros-plan-runtime** - Process management and lifecycle
   - Node spawning with ROS 2 arguments
   - Graceful shutdown (SIGTERM → SIGKILL)
   - Restart policies: Never, Always, OnFailure
   - Dynamic parameter updates with program diffing
   - Launch include tracking and reloading
   - Status reporting (table and JSON formats)
   - Event logging with circular buffer
   - Metrics collection (starts, stops, crashes, restarts)

4. **ros2plan** - CLI tool
   - Commands: `compile`, `run` (with runtime)
   - Usage: `ros2plan compile <plan-file> [args]`
   - Argument passing: `key=value` or `key:type=value`

5. **ros-utils** - ROS 2 integration utilities
   - Package and executable resolution via `ros2` CLI
   - Node command-line generation with remapping
   - ROS 2 conventions compliance

### Python Components

- **launch2dump** - Metadata extraction from ROS 2 launch files
  - Visits launch entities without spawning processes
  - Extracts nodes, parameters, remappings, containers
  - JSON/YAML output formats
  - Uses UV for dependency management (migrated from Rye)

- **ros2-introspect** - ROS 2 node interface introspection
  - Discovers publishers, subscriptions, services, clients without middleware
  - Uses rmw_introspect_cpp RMW implementation
  - Dependency checking with clear error messages
  - Requires workspace to be built and sourced

- **launch2plan** - Launch-to-Plan conversion tool (Phase 12, in progress)
  - Converts ROS 2 launch files to ROS-Plan YAML format
  - Socket inference from introspection and remappings
  - Conditional branch exploration with `when` clauses
  - Automatic link generation from topic connections
  - Argument type inference from default values
  - Include handling with cycle detection and argument forwarding
  - Metadata tracking with TODO completion detection (JSONPath addressing)
  - CLI commands: `convert` (generates plan + metadata), `status` (shows TODO progress)

### External Dependencies (Git Submodules)

- **rmw_introspect** - Located at `external/rmw_introspect/` (git submodule)
  - Separately maintained at https://github.com/jerry73204/rmw_introspect
  - Contains two packages:
    - `rmw_introspect_cpp/` - C++ RMW implementation for lightweight introspection
    - `ros2_introspect/` - Python wrapper for the RMW implementation
  - **Integration with build system**:
    - `ros2/rmw_introspect_cpp` is a symlink to `external/rmw_introspect/rmw_introspect_cpp`
    - Built automatically via `make build` using colcon
    - `ros2-introspect` (Python) added to UV workspace via `python/pyproject.toml`
  - **Dependency chain**:
    - launch2plan (Python) → ros2-introspect (Python) → rmw_introspect_cpp (ROS 2 C++)
    - launch2plan tests require workspace sourced (for rmw_introspect_cpp)
  - Required by launch2plan for node interface introspection without middleware

## Key File Formats

### Plan Files (*.yaml)
Plans define ROS 2 nodes and their connections. Example structure:
```yaml
node:
  node_name:
    pkg: package_name
    exec: executable_name
    socket:
      socket_name: !pub/!sub
link:
  link_name: !pubsub
    type: msg/type/Name
    src: ["node/socket"]
    dst: ["node/socket"]
```

## Development Commands

This project uses a Makefile for common development tasks. Use `make help` to see all available targets.

### Building
```bash
# Build all targets (uses release-with-debug profile)
make build

# Build specific crate
cargo build -p ros-plan-compiler --profile release-with-debug

# Build external dependencies (rmw_introspect)
# Note: External modules are built independently
cd external/rmw_introspect && source /opt/ros/humble/setup.bash && make build
```

### Testing
```bash
# Run all tests (uses cargo-nextest with release-with-debug profile)
make test

# Run tests for specific crate
cargo nextest run -p ros-plan-compiler --cargo-profile release-with-debug

# Run tests with standard cargo test
cargo test -p ros-plan-compiler --profile release-with-debug
```

### Code Quality
```bash
# Format code
make format

# Check formatting and run clippy (with -D warnings)
make lint

# Format code manually
cargo +nightly fmt

# Run clippy manually
cargo clippy --all-targets --all-features -- -D warnings
```

### Cleaning
```bash
# Clean build artifacts
make clean
```

### Running the Compiler
```bash
# Compile a plan file
cargo run --bin ros2plan -- compile examples/introduction.yaml

# With arguments
cargo run --bin ros2plan -- compile examples/eval.yaml --args key=value

# With output file
cargo run --bin ros2plan -- compile examples/introduction.yaml -o output.launch
```

### Python Development
```bash
# Install Python dependencies (uses UV workspace)
cd python && uv sync

# Run launch2dump
cd python/launch2dump && uv run launch2dump <launch-file>

# Run ros2-introspect (requires workspace sourced)
source install/setup.bash
cd python/ros2-introspect && uv run ros2-introspect <package> <executable>

# Run launch2plan (requires workspace sourced)
source install/setup.bash
cd python/launch2plan && uv run launch2plan convert <launch-file>

# Run Python tests (requires workspace sourced)
# Note: Makefile handles environment sourcing automatically
make test

# Run individual Python test suites
source install/setup.bash
cd python/launch2dump && uv run pytest
cd python/ros2-introspect && uv run pytest
cd python/launch2plan && uv run pytest
```

## Important Implementation Details

### Compiler & Format
- Uses mlua (Lua) for dynamic evaluation of expressions in plan files
- Plans support parameter substitution and conditional compilation using the `when` clause
- Custom YAML tags: !pub, !sub, !pubsub, !i64, !f64, !bool, !path, !expr
- Socket/link abstraction decouples node definitions from topic names
- Transparent includes allow multi-level socket forwarding
- Node identifiers (not names) are used for references

### Runtime System
- Spawns ROS 2 nodes as child processes using `Command`
- Graceful shutdown: SIGTERM → wait → SIGKILL
- Restart policies with configurable backoff
- Program diffing detects parameter changes for selective restarts
- Launch include tracking maps nodes to their source files
- Event logging uses a circular buffer (configurable size)
- Status reporting supports both human-readable tables and JSON

### ROS 2 Integration
- Resolves packages and executables using `ros2 pkg` and `ros2 run` commands
- Generates node command-line arguments following ROS 2 conventions
- Supports remapping, parameters, namespaces, ROS arguments
- Compatible with standard ROS 2 launch files via launch2dump

### Testing
- 440 total tests (330 Rust + 110 Python)
  - Rust: 330 tests (format, compiler, runtime, CLI, ros-utils)
  - Python: 25 launch2dump + 9 ros2-introspect + 85 launch2plan (4 skipped - require rmw_introspect_cpp)
- Uses cargo-nextest for parallel Rust test execution
- Python tests require ROS 2 and workspace sourced (Makefile handles this)
- All Rust tests run with `release-with-debug` profile for speed
- Zero tolerance for clippy warnings (`-D warnings`)

### Dependency Management (Python)
- All Python packages use UV for dependency management
- ros2-introspect requires rmw_introspect_cpp to be built and sourced
- Dependency checking provides explicit error messages:
  - Missing ROS 2: Instructs to source `/opt/ros/humble/setup.bash`
  - Missing rmw_introspect_cpp: Instructs to build and source workspace
- Makefile automatically sources workspace before running Python tests
- Function `check_rmw_introspect_available()` validates dependencies before use

## Documentation

Comprehensive documentation in `book/src/`:
- `roadmap.md` - Implementation status and feature tracking
- `plan-format.md` - Plan YAML specification
- `runtime_user_guide.md` - Runtime system usage
- `runtime_architecture.md` - Internal design
- `runtime_config.md` - Configuration reference
- `launch2plan.md` - Launch-to-plan conversion design (Phase 12, phases 1-8 complete)

## Development Workflow

1. **Before making changes**: Read relevant documentation in `book/src/`
2. **During development**: Run `make lint` frequently to catch issues early
3. **After changes**: Always run `make test` to verify all tests pass
4. **Before committing**: Ensure `make lint && make test` passes completely
5. **Phase 12 implementation**: Currently implementing launch2plan incrementally
   - Phases 1-8 complete (foundation through metadata tracking)
   - See `book/src/launch2plan.md` for design and progress tracking
   - Follow incremental phase-by-phase approach as documented

## Important Notes

### Makefile Shell Execution
- **Critical**: Each line in a Makefile target runs in a separate shell
- Environment sourcing must be on the same line as commands that need it
- **Correct**: `. install/setup.sh && cd python/launch2plan && uv run pytest`
- **Incorrect**: Splitting across lines with backslash continuation (sourcing won't persist)
- This affects all Python test commands that require ROS 2 environment

### Phase 12 (launch2plan) Recent Completions
- **Phase 8**: Metadata tracking complete (13 tests)
  - Created `metadata.py` with data structures (TodoItem, ConversionMetadata, ConversionStats, TodoContext, etc.)
  - Modified `builder.py` to collect TODOs during plan generation with rich context
  - Implemented metadata persistence with save/load to JSON (`.plan.meta.json` files)
  - Created `PlanParser` for YAML parsing and TODO discovery with JSONPath addressing
  - Implemented `TodoStatusUpdater` for detecting user-completed TODOs
  - Added `statistics.py` for conversion statistics calculation
  - Integrated metadata into `convert` CLI command with SHA256 staleness detection
  - Added `status` subcommand to display TODO completion progress
  - See `metadata.py`, `statistics.py`, and `__main__.py:handle_status()` for implementation
- **Phase 7**: Include handling complete (8 tests)
  - Recursive include processing with cycle detection
  - Argument forwarding with type inference
  - Conditional includes with `when` clauses
- **Phase 6**: Conditional branch exploration complete (18 tests)
  - Handles IfCondition and UnlessCondition with proper subclass checking
  - Converts conditions to Lua expressions for `when` clauses
  - Supports nested conditions with "and" operator

## Common Patterns

### Value Types in Format
```rust
// ros-plan-format uses an enum for values
pub enum Value {
    I64(i64),
    F64(f64),
    Bool(bool),
    String(String),
    Path(PathBuf),
    Expr(String),  // Lua expression
}
```

### Socket References
```yaml
# Socket reference format: node_id/socket_name
link:
  camera_image: !pubsub
    src: ["camera/image"]      # Node ID "camera", socket "image"
    dst: ["detector/image"]     # Node ID "detector", socket "image"
```

### When Clauses
```yaml
node:
  optional_node:
    pkg: my_pkg
    exec: my_node
    when: "$(enable_feature)"  # Lua expression in $()
```

### Restart Policies
```rust
pub enum RestartPolicy {
    Never,                              // No restart
    Always { backoff: Duration },       // Always restart with delay
    OnFailure {                         // Restart on non-zero exit
        max_retries: u32,
        backoff: Duration,
    },
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jerry73204) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
