## data-designer

> 🦀 **Data Designer** - Cross-platform Rust application (Desktop + WASM) with unified HTTP client and gRPC microservices architecture for designing, testing, and managing dynamic data transformation rules using a soft DSL system for financial services.

# CLAUDE.md

## Project Overview

🦀 **Data Designer** - Cross-platform Rust application (Desktop + WASM) with unified HTTP client and gRPC microservices architecture for designing, testing, and managing dynamic data transformation rules using a soft DSL system for financial services.

### Architecture

**Platform Layers:**
- **Desktop UI**: Native Rust application with egui + wgpu + Tokio async runtime (`web-ui/src/main.rs`)
- **Web UI**: Pure Rust WASM client with egui (`web-ui/src/lib.rs`)
- **Unified HTTP Client**: Both platforms use identical reqwest-based HTTP layer (`web-ui/src/grpc_client.rs`)

**Backend Services:**
- **gRPC Server**: Financial taxonomy service with dual Protocol Buffers + HTTP endpoints (`grpc-server/`)
  - Port 50051 (gRPC) + Port 8080 (HTTP REST fallback)
- **Onboarding Library**: Metadata-driven workflow compiler and executor (`onboarding/`)
  - YAML-based configuration (CBU templates, product catalog, resource dictionaries)
  - Async execution with sync compilation
  - IR-based plan generation and task orchestration
- **Core Library**: Expression engine with database layer (`data-designer-core/`)

**Data Layer:**
- **Database**: PostgreSQL with pgvector for semantic similarity
  - 21 tables (cleaned up from 25)
  - Database views for CBU/entity mapping
  - 100 entities + 8 CBUs in current dataset

### Quick Start

```bash
# Desktop application (best for development/debugging)
./rundesk.sh

# Web application (WASM + browser)
./runwasm.sh

# Development
cargo build                   # Build entire workspace
cargo test --all             # Run comprehensive test suite
cargo clippy -- -D warnings   # Enforce code quality
```

### Workspace Structure

```
data-designer/
├── web-ui/              # Cross-platform UI (desktop + WASM)
├── grpc-server/         # Backend gRPC + HTTP server
├── data-designer-core/  # Expression engine + database layer
├── onboarding/          # Workflow compiler and executor
├── onboarding-cli/      # CLI demo for onboarding library
├── cbu-dsl-lsp/        # Language server (WIP)
├── template-server/     # Template management (legacy)
├── proto/              # Protocol Buffers definitions
├── database/           # PostgreSQL migrations and scripts
└── scripts/            # Build and deployment automation
```

### Current Focus Areas

**Active Development:**
1. **Onboarding Library** - Workflow compilation and execution system
   - Metadata-driven plan generation from YAML configs
   - IR-based task orchestration with dependency resolution
   - Adapter stubs for gRPC, HTTP, and Kafka integration
2. **Resource DSL Management** - Dual-tab UI for CBU and Entity management
3. **Cross-Platform Stability** - Desktop renderer migrated to wgpu for system stability

**Key Features:**
- **CBU DSL Management** - Client Business Unit DSL editor and execution
- **Staged Loading** - CBUs on startup (8 records), entities on demand (100 records)
- **Cross-Platform** - Identical functionality desktop (native) and web (WASM)
- **Native Debugging** - Full IDE integration with desktop version
- **Advanced Parser** - nom-based parser with 6 extensions

### Key Files

**Frontend (Cross-Platform):**
- `web-ui/src/main.rs` - Desktop application entry point (native + Tokio)
- `web-ui/src/lib.rs` - WASM web application entry point
- `web-ui/src/cbu_dsl_ide.rs` - Shared CBU DSL IDE (~99.5% code shared)
- `web-ui/src/grpc_client.rs` - Unified HTTP client (reqwest)
- `web-ui/src/wasm_utils.rs` - Cross-platform utilities (logging, async)

**Backend:**
- `grpc-server/src/main.rs` - gRPC + HTTP server
- `proto/financial_taxonomy.proto` - API definitions
- `onboarding/src/planner/compile.rs` - Workflow compiler
- `onboarding/src/runtime/scheduler.rs` - Async task executor
- `onboarding/meta/*.yaml` - Configuration metadata

**Database:**
- `database/migrations/` - Sequential PostgreSQL migrations
- `database/schema/` - Current schema exports

**Scripts:**
- `rundesk.sh` - Desktop launcher
- `runwasm.sh` - WASM build and deploy
- `web-ui/build-web.sh` - WASM compilation
- `web-ui/serve-web.sh` - Local HTTP server

### Recent Changes (Last 2 Weeks)

**Onboarding Library Integration:**
- Added metadata-driven workflow compiler with YAML configs
- Implemented IR-based plan generation and task orchestration
- Database schema cleanup (25 → 21 tables)
- Added 5 new tables for grpc-server integration
- Fixed all clippy warnings with serde rename attributes

**UI Improvements:**
- Resource DSL management with dual-tab interface
- Centralized CBU/Entity state management
- Desktop renderer migrated from glow to wgpu

**Cross-Platform Architecture:**
- Unified HTTP client for both desktop and WASM
- ~99.5% code sharing between platforms
- Minimal conditional compilation

### Testing & Quality

**Test Coverage:**
- 20+ comprehensive tests including gRPC integration
- Parser tests: expressions, functions, conditionals, arithmetic
- Database tests: models, attributes, data dictionary integration
- gRPC integration tests: health checks, entity management, CBU operations

**Code Quality:**
- Cargo clippy with `-D warnings` enforced
- Minimal unsafe code
- Comprehensive error handling with anyhow

### Database Schema

**Current State:**
- PostgreSQL database: `data_designer`
- 21 tables (after cleanup migration 006)
- 3 database views: `cbu`, `cbu_investment_mandate_structure`, `cbu_member_investment_roles`
- Views handle entity type mapping automatically

**Key Tables:**
- `cbu_profile` - CBU records and metadata
- `entity`, `entity_attributes` - Entity data with dynamic attributes
- `product_options`, `resource_templates`, `resource_instances` - Onboarding workflow data
- `resource_sheets`, `dsl_execution_logs` - Execution tracking

### Performance Notes

- **Build time**: Sub-second incremental builds
- **Runtime**: Native desktop performance + 60fps WASM
- **Startup**: 93% faster with staged loading (8 CBUs → 100 entities on-demand)
- **Distribution**: Single native binary + WASM bundle
- **Network**: Efficient JSON over HTTP/1.1 with reqwest

### Development Workflow

**Feature Branch Active:**
- `feat/onboarding-library-integration` - Current working branch
- Main branch: `main`
- Recent merges: `feat/desktop-wgpu-renderer`, `pure-rust-egui`

**Environment:**
- Database URL: `postgresql:///data_designer?user=adamtc007`
- gRPC port: 50051
- HTTP port: 8080
- WASM dev server: 8000

**Editor Setup:**
- Zed editor configs in `.zed/`
- Tasks: onboarding compile/run, tests, watch mode
- Keybindings: cmd-alt-o (run demo), cmd-alt-r (rerun)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adamtc007)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adamtc007)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
