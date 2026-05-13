## schedcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

schedcp is a Linux scheduler optimization project that uses the sched-ext (scx) framework to implement and test various BPF-based kernel schedulers. The project includes multiple scheduler implementations in both C and Rust, along with benchmarking tools, workloads, and an MCP (Model Context Protocol) server for AI-assisted scheduler management.

## Essential Build Commands

### Building the MCP Server and CLI
```bash
# Build the schedcp MCP server and CLI tool
cd mcp
cargo build --release

# Binaries will be available at:
# - mcp/target/release/schedcp (MCP server)
# - mcp/target/release/schedcp-cli (CLI tool)
```

### Building Schedulers
```bash
# Install dependencies first (one-time setup)
cd scheduler
make deps

# Build all schedulers (C and Rust) - outputs to sche_bin/
make

# Build specific types
make build-c        # Build only C schedulers
make build-rust     # Build only Rust schedulers
make build-tools    # Build tools (scx_loader, scxctl, scxtop) - outputs to tools/

# Generate scheduler documentation
make doc

# Clean build artifacts
make clean

# Update scx submodule
make update

# Install schedulers to ~/.schedcp/scxbin/
make install
```

### Running Tests
```bash
# Unit tests for sched-ext framework
cd scheduler/scx
meson setup build
meson test -v -C build

# Run specific benchmark workloads
python workloads/basic/scheduler_test/schbench_bench_start.py
python workloads/llama.cpp/llamacpp_bench_start.py
python workloads/cxl-micro/cxl_micro_bench_start.py
```

## High-Level Architecture

### Core Components Integration

**MCP Server Architecture**: The heart of the project is the MCP server (`mcp/src/`) which provides AI-assisted scheduler management. Key integration points:

- **scheduler_manager.rs**: Manages the lifecycle of kernel schedulers, interfaces with embedded scheduler binaries, handles both built-in and custom schedulers
- **scheduler_generator.rs**: Creates, compiles, and verifies custom BPF schedulers from source code (private module accessed through manager)
- **system_monitor.rs**: Collects real-time CPU, memory, and scheduler metrics from `/proc` for performance analysis
- **workload_profile.rs**: Creates natural language workload descriptions and tracks performance history across different schedulers
- **storage.rs**: Persists workload data in `schedcp_workloads.json`, enabling learning from past optimizations

**Scheduler Framework Integration**: The project embeds all sched-ext schedulers as resources, enabling standalone operation without requiring local builds. The embedded approach allows the MCP server to manage any scheduler configuration programmatically.

**Workload-Scheduler Optimization Loop**: The architecture enables a closed-loop optimization system where:
1. Workloads are described in natural language and classified
2. Multiple schedulers are tested automatically with optimal configurations
3. Performance history is tracked and used for future recommendations
4. AI assistants can make data-driven scheduler selections

### Directory Structure and Data Flow

```
schedcp/
├── mcp/                           # MCP server for AI-assisted optimization
│   ├── src/
│   │   ├── scheduler_manager.rs   # Scheduler lifecycle management
│   │   ├── scheduler_generator.rs # Custom scheduler creation/compilation
│   │   ├── system_monitor.rs      # Real-time metrics collection
│   │   ├── workload_profile.rs    # Workload classification and history
│   │   ├── storage.rs             # Persistent performance data
│   │   ├── main.rs                # MCP server entry point
│   │   ├── lib.rs                 # Core MCP implementation
│   │   └── cli.rs                 # CLI tool implementation
│   ├── new_sched/                 # Custom scheduler working directory
│   │   ├── loader                 # BPF loader for custom schedulers
│   │   └── *.bpf.{c,o}           # Custom scheduler sources and objects
│   └── schedcp_workloads.json     # Performance history database
├── scheduler/                     # Scheduler build system and metadata
│   ├── scx/                       # sched-ext framework (submodule)
│   ├── sche_bin/                  # Compiled scheduler binaries
│   ├── scheduler_runner.py        # Python scheduler interface
│   └── schedulers.json            # Scheduler metadata and capabilities
└── workloads/                     # Benchmark workloads for testing
    ├── basic/schbench/            # Scheduler latency benchmark
    ├── llama.cpp/                 # LLM inference workload
    └── cxl-micro/                 # Memory subsystem benchmark
```

**Critical Integration Points**:
- `schedulers.json` contains metadata about each scheduler's algorithm, tuning parameters, and use cases
- `scheduler_runner.py` provides a Python interface that the MCP server uses to control schedulers
- The MCP server embeds all scheduler binaries, making it self-contained for deployment

### Scheduler Selection and Optimization Logic

The project implements an intelligent scheduler selection system:

**Workload Analysis**: Natural language descriptions are analyzed to identify workload characteristics (latency-sensitive, throughput-focused, interactive, etc.)

**Scheduler Matching**: Each scheduler in `schedulers.json` has detailed metadata including:
- Algorithm type (vruntime-based, EDF, multi-domain, etc.)
- Performance profile (low latency, balanced, throughput-optimized)
- Optimal use cases and tuning parameters
- Production readiness status

**Automated Testing**: The system can automatically test multiple schedulers with workload-specific configurations and compare results across metrics like wakeup latency, request latency, and throughput.

## MCP Server Integration

### AI-Assisted Workflow
The MCP server enables Claude to perform complete scheduler optimization workflows:

1. **Workload Profiling**: Create detailed workload profiles from natural language descriptions
2. **Scheduler Analysis**: Automatically identify optimal scheduler candidates based on workload characteristics
3. **Custom Scheduler Creation**: Create, compile, and verify custom BPF schedulers from source code
4. **Automated Testing**: Run systematic comparisons across multiple schedulers with optimal configurations
5. **Performance Monitoring**: Collect real-time CPU, memory, and scheduler metrics during execution
6. **Performance Tracking**: Maintain persistent history of scheduler performance for different workloads
7. **Data-Driven Recommendations**: Make scheduler recommendations based on historical performance data

### Key MCP Commands
- `list_schedulers`: Get comprehensive scheduler information including algorithms and tuning parameters
- `run_scheduler`: Start schedulers with specific configurations (works for both built-in and custom schedulers)
- `stop_scheduler`: Stop running schedulers by execution ID
- `get_execution_status`: Monitor running scheduler performance and output
- `create_and_verify_scheduler`: Create, compile, and verify custom BPF schedulers
- `system_monitor`: Collect CPU, memory, and scheduler metrics (start/stop commands)
- `workload`: Unified command for workload profile management (create/list/get_history/add_history)

### Key CLI Commands
- `schedcp-cli list [--name NAME] [--production]`: List available schedulers with detailed information
- `schedcp-cli run SCHEDULER [ARGS...]`: Run a scheduler with optional arguments
- `schedcp-cli create-and-run SOURCE.bpf.c`: Create and run a custom scheduler from source
- `schedcp-cli monitor [--duration SECS]`: Monitor system metrics for specified duration (default: 10s)

## Development Patterns

### Adding Built-in Schedulers
1. Implement scheduler in `scheduler/scx/scheds/c/` or `scheduler/scx/scheds/rust/`
2. Update `scheduler/schedulers.json` with metadata, algorithm type, and tuning parameters
3. Add performance characteristics and optimal use cases
4. Run `make install` to copy to `~/.schedcp/scxbin/`
5. Test with existing workloads and update MCP server scheduler database

### Creating Custom Schedulers via MCP
1. Use `create_and_verify_scheduler` tool with BPF C source code
2. Scheduler is automatically compiled using clang with appropriate flags
3. Verification loads the scheduler in kernel for 10 seconds to test
4. Compiled scheduler stored in `mcp/new_sched/` directory
5. Custom schedulers can be run via `run_scheduler` like built-in schedulers

### Custom Scheduler Architecture
- **SchedulerGenerator** (private): Handles compilation and verification
  - Uses clang with BPF target and scx includes from `scheduler/scx/scheds/include`
  - Compiles `.bpf.c` to `.bpf.o` in `mcp/new_sched/` directory
  - Uses loader binary for kernel verification
- **SchedulerManager** (public): Provides unified API for both built-in and custom schedulers
  - `create_execution()` automatically detects scheduler type (built-in vs custom)
  - Custom schedulers tracked in execution map with `"custom:"` prefix
  - Stop command uses `pkill` for custom schedulers, ProcessManager for built-in

### Workload Development
1. Create workload in appropriate `workloads/` subdirectory
2. Implement using the patterns from `schbench` (message/worker thread model for scheduling-sensitive workloads)
3. Ensure output can be captured for metrics analysis
4. Add workload profile creation support in the MCP server
5. Use `system_monitor` tool to collect metrics during execution

### Performance Analysis Integration
The project emphasizes automated performance analysis:
- Use JSON output format for consistent metrics capture
- Implement workload-specific metrics (latency percentiles, throughput, scheduling delay)
- Use `system_monitor` to collect CPU, memory, and scheduler metrics
- Store results in persistent workload history for trend analysis
- Enable Claude to analyze performance patterns and make optimization recommendations

### System Monitoring
The `system_monitor` module collects real-time metrics:
- **CPU metrics** from `/proc/stat`: user, system, idle, iowait times → utilization %
- **Memory metrics** from `/proc/meminfo`: total, free, available, buffers, cached → usage %
- **Scheduler metrics** from `/proc/schedstat`: run time, wait time, timeslices
- Samples collected every 1 second with async task-based collection
- Summary includes averages, maximums, and totals for analysis

## Requirements and Environment

- Linux kernel 6.12+ with sched-ext support
- Clang/LLVM >= 16 (17 recommended)
- Rust toolchain >= 1.82
- Meson >= 1.2.0, libbpf >= 1.2.2
- Root privileges required for scheduler loading (configure sudo access)

## Testing and Verification

### CLI Testing
```bash
# Test listing schedulers
./target/release/schedcp-cli list --production

# Test system monitoring
./target/release/schedcp-cli monitor --duration 5

# Test custom scheduler creation
./target/release/schedcp-cli create-and-run test_scheduler.bpf.c
```

### MCP Integration Testing
```bash
# Build and test
cargo build --release
cargo test

# The MCP server can be tested via Claude Desktop or programmatically
# Custom scheduler workflow is fully integrated
```

### Verification Steps for Custom Schedulers
1. Source validation: Check for required BPF operations (select_cpu, enqueue, dispatch, init, exit)
2. Compilation: Clang compiles with BPF target and scx includes
3. Kernel verification: Loader loads scheduler in kernel for 10 seconds
4. Execution tracking: Both built-in and custom schedulers tracked in execution map
5. Output capture: Stdout/stderr captured for debugging and analysis

## Security and Deployment Notes

Schedulers require root privileges to load. For AI-assisted operation:
- Set `SCHEDCP_SUDO_PASSWORD` environment variable for MCP server
- Consider passwordless sudo configuration for production deployments
- All scheduler operations are logged to `schedcp.log` for audit trails
- Custom schedulers compiled with standard BPF security restrictions
- Verification step ensures custom schedulers can load before persistent use

---
> Source: [eunomia-bpf/schedcp](https://github.com/eunomia-bpf/schedcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
