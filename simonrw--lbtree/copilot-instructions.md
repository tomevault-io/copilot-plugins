## lbtree

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`lbtree` is a Rust CLI tool that displays a tree representation of AWS resources.

**Supported Resources:**
- **Application Load Balancer (ELBv2)**: Shows hierarchical structure of load balancers, listeners, rules, actions, target groups, and targets
- **API Gateway REST APIs**: Shows hierarchical structure of REST APIs, resources, methods, and integrations

More resource types are planned for the future.

## Development Commands

**Build:**
```bash
cargo build
```

**Run:**
```bash
# Interactive mode - select resource type, then specific resource
cargo run

# Application Load Balancer with ARN specified
cargo run -- elbv2 --load-balancer-arn <ARN>

# Application Load Balancer interactive selection
cargo run -- elbv2

# API Gateway with REST API ID specified
cargo run -- apigateway --api-id <API_ID>

# API Gateway interactive selection
cargo run -- apigateway 
```

**Check/Lint:**
```bash
cargo check
cargo clippy
```

**Format:**
```bash
cargo fmt
```

**Test:**
```bash
# Requires LocalStack running on localhost:4566
cargo test

# Run specific integration test
cargo test apigateway_integration

# Review snapshot test changes (after running tests)
cargo insta review
```

## Architecture

### Module Structure

The codebase is organized into focused modules:
- **src/main.rs**: CLI entry point, subcommand routing, resource type selection
- **src/alb.rs**: Application Load Balancer display logic
- **src/apigateway.rs**: API Gateway REST API display logic
- **src/present.rs**: `Present` trait and output abstraction (`OutputWriter`)
- **src/lib.rs**: Library interface exposing modules for testing

### Core Design Pattern: Present Trait

The application uses a trait-based presentation pattern (`Present` trait in src/present.rs) that all AWS resource types implement. Each type provides:
- `content()`: String representation of the resource
- `indent()`: Hierarchical indentation level (0 for root, 2 for resources, 4 for methods, 6 for integrations/actions)
- `present()`: Renders the resource using an `OutputWriter` with proper indentation

The `OutputWriter` trait abstracts output, enabling:
- `StdoutWriter`: Production use (prints to stdout)
- `BufferWriter`: Testing use (captures output for snapshot tests)

### Data Flow

1. **Authentication**: Uses AWS SDK default credential chain via `aws_config::load_from_env()`
2. **Resource Selection**: Either CLI arg or interactive skim selection
3. **Parallel Fetching**: Concurrent tasks fetch resource hierarchies (ALB uses tokio::join!)
4. **Display**: Resources implement `Present` and render via `OutputWriter`

### AWS Resource Hierarchies

**Application Load Balancer:**
```
LoadBalancer (indent: 0)
├── Listener (indent: 2)
│   └── Rule (indent: 4)
│       └── Action (indent: 6)
└── TargetGroup (indent: 2)
    └── TargetHealthDescription (indent: 4)
```

**API Gateway REST API:**
```
RestApi (indent: 0)
└── Resource (indent: 2)
    └── Method (indent: 4)
        └── Integration (indent: 6)
```

### Parallelization Strategy

**ALB:** Spawns two tokio tasks to fetch data concurrently:
- **listeners_fut**: Fetches listeners → rules → actions for each listener
- **target_groups_fut**: Fetches target groups → target health for each group

**API Gateway:** Sequential fetching (resources → methods → integrations)

## Testing

### Integration Tests

The project uses LocalStack for integration testing of AWS services:
- **tests/apigateway_integration.rs**: API Gateway snapshot tests
- **tests/common/mod.rs**: Shared test utilities (LocalStack config, health checks)

**Requirements:**
- LocalStack must be running on `localhost:4566`
- Tests create real AWS resources in LocalStack and clean them up afterward

### Snapshot Testing

Uses `insta` for snapshot testing of tree output:
- Snapshots stored in `tests/snapshots/`
- Fixtures use filters to replace dynamic values (IDs, names) with placeholders
- Run `cargo insta review` to review and accept snapshot changes

### Test Fixtures

`ApiGatewayTestFixture` provides:
- Automatic setup/teardown of test resources
- Snapshot filtering for consistent test output
- Guaranteed cleanup even on test failure

## External Dependencies

**Testing only** (LocalStack for integration tests)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/simonrw)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/simonrw)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
