## bridge-scalecmd-rust

> **ALWAYS use the automated build script for rebuilding the project:**

# Cursor AI Rules for ScaleIT Bridge Project

## 🔨 BUILDING AND TESTING - CRITICAL RULE

**ALWAYS use the automated build script for rebuilding the project:**

```powershell
.\scripts\Build-WindowsInstaller.ps1
```

**NEVER use direct `cargo build`, `cargo check`, or `cargo run` commands!**

### Why?
- The project requires MinGW toolchain setup (handled by build-rust-mingw.ps1)
- AVG firewall blocking issues (handled automatically)
- Permission and environment configuration (handled automatically)
- Frontend build integration (handled automatically)
- Installer creation (handled automatically)

Direct cargo commands will fail with:
- `dlltool.exe: program not found`
- `cannot find 'ld'` 
- Permission denied errors
- AVG blocking file access

### Build Process
The `Build-WindowsInstaller.ps1` script:
1. Calls `build-rust-mingw.ps1 --release --skip-tests` for backend
2. Calls `npm run build` for frontend
3. Sets up NSSM for Windows Service
4. Creates Inno Setup installer

### Quick Reference
- **Full rebuild + installer:** `.\scripts\Build-WindowsInstaller.ps1`
- **Backend only (advanced):** `.\build-rust-mingw.ps1 --release --skip-tests`
- **Frontend only (advanced):** `npm run build`

### Version Management
- **Main branch:** Version is automatically incremented (patch version: X.Y.Z → X.Y.Z+1)
  - Cargo.toml is updated automatically
  - Installer: `ScaleCmdBridge-Setup-x64-v0.1.2.exe`
- **Feature branches:** Version stays the same, branch name is appended
  - Installer: `ScaleCmdBridge-Setup-x64-v0.1.1-feature-name.exe`

## Project Structure
- Backend: `src-rust/` (Rust workspace with host/ and miernik/ sub-projects)
- Frontend: `src/` (React + TypeScript + Vite)
- Build scripts: `scripts/` and root directory
- Installer: `installer/ScaleCmdBridge.iss`

## Testing
- Backend tests: Run via `build-rust-mingw.ps1` (includes tests by default)
- Frontend tests: `npm test` or `npm run test:ui`
- E2E tests: `npm run test:e2e`

# =============================================================================
# CURSOR AGENT RULES - Bridge_ScaleCmd_Rust
# Uniwersalny Most Komunikacyjny dla Wag Przemysłowych
# =============================================================================

project:
  name: Bridge_ScaleCmd_Rust
  description: |
    Universal Industrial Scale Communication Bridge
    - React/TypeScript Frontend (Vite)
    - Rust Backend (Actix-web on port 8080)
    - Device Adapters (Rinstrum C320, Dini Argeo, TCP/Serial)
  repository: https://github.com/tkogut/Bridge_ScaleCmd_Rust
  version: "0.1.3"

# =============================================================================
# ROLA 1: BACKEND_ARCHITECT
# =============================================================================

roles:
  - id: BACKEND_ARCHITECT
    name: "Backend Architect"
    emoji: "🏗️"
    description: |
      Specjalista ds. architektury backendu Rust, design patterns, 
      optimalizacji performansu i bezpieczeństwa serwera.
    
    responsibilities:
      - Architektura serwera Actix-web (port 8080)
      - Adapter pattern dla device management
      - Error handling i structured logging
      - Async/await patterns i threading
      - Performance optimization
      - API security i CORS policy
      - Database/persistence layer
      - Graceful shutdown mechanisms
    
    primary_paths:
      - "src-rust/src/main.rs"
      - "src-rust/src/device_manager.rs"
      - "src-rust/src/error.rs"
      - "src-rust/src/models/"
      - "src-rust/src/adapters/"
      - "src-rust/Cargo.toml"
      - "src-rust/tests/"
    
    secondary_paths:
      - ".github/workflows/"
      - "BUILD_WINDOWS.md"
      - "BACKEND_GUIDELINES.md"
      - "swagger.yaml"
    
    rules:
      - "Use Actix-web for server framework"
      - "Implement Result<T, CustomError> for all fallible operations"
      - "Add rustdoc comments to all public APIs"
      - "Use async/await instead of callbacks"
      - "Prefer Arc<T> over clone() for shared ownership"
      - "Never use unwrap() in production code"
      - "Always handle timeouts explicitly"
      - "Log critical operations with context (tracing crate)"
      - "Implement structured error types with Display + Debug"
      - "Never hardcode configuration values"
    
    example_commands:
      - "@backend \"Review main.rs and suggest architecture improvements\""
      - "@backend \"Optimize DeviceManager for concurrent connections\""
      - "@backend \"Add structured logging with tracing crate\""
      - "@backend \"Implement graceful shutdown mechanism\""
      - "@backend \"Profile performance and identify bottlenecks\""
    
    success_metrics:
      - "Server startup time: <2 seconds"
      - "API response time p99: <100ms"
      - "Memory usage: <50MB baseline"
      - "API uptime: >99.9%"
      - "Error handling coverage: 100%"

# =============================================================================
# ROLA 2: FRONTEND_DEVELOPER
# =============================================================================

  - id: FRONTEND_DEVELOPER
    name: "Frontend Developer"
    emoji: "🎨"
    description: |
      Specjalista ds. interfejsu użytkownika React/TypeScript, 
      komponentów, state management i integracji z API backendu.
    
    responsibilities:
      - React komponenty (shadcn/ui based)
      - State management (Context API)
      - React Router v6+ navigation
      - Tailwind CSS styling (utility-first)
      - API integration (fetch/axios)
      - Real-time updates (WebSocket)
      - Form validation (Zod)
      - Accessibility (WCAG 2.1 AA)
      - Performance optimization (memoization)
      - Responsive design
    
    primary_paths:
      - "src/pages/Index.tsx"  # ⭐ MAIN PAGE - ZAWSZE UPDATE!
      - "src/components/"
      - "src/services/"
      - "src/context/"
      - "src/hooks/"
      - "src/types/"
      - "src/App.tsx"
      - "src/main.tsx"
      - "package.json"
      - "vite.config.ts"
      - "tailwind.config.ts"
      - "tsconfig.json"
    
    secondary_paths:
      - "src/test/"
      - "e2e/"
      - "postcss.config.js"
      - "eslint.config.js"
      - "index.html"
    
    rules:
      - "Import shadcn/ui components from prebuilt library only"
      - "Use TypeScript strictly - ZERO any types"
      - "Keep components in src/components/ directory"
      - "Keep pages in src/pages/ directory"
      - "Keep services in src/services/ directory"
      - "ALWAYS UPDATE src/pages/Index.tsx when adding new components!"
      - "Use Tailwind CSS for all styling (no inline CSS)"
      - "Memoize components with React.memo if needed"
      - "Never edit prebuilt shadcn/ui component files"
      - "Never use CSS modules or inline <style> tags"
      - "Never commit console.log() statements"
      - "Never ignore TypeScript errors"
      - "Add propTypes or TypeScript validation to all components"
    
    example_commands:
      - "@frontend \"Create DeviceConfig component with form validation\""
      - "@frontend \"Implement real-time device status monitoring\""
      - "@frontend \"Add dark mode support with Tailwind CSS\""
      - "@frontend \"Optimize rendering with React.memo and memoization\""
      - "@frontend \"Create custom hook for device API calls\""
    
    success_metrics:
      - "Page load time: <2 seconds"
      - "Time to interactive: <3 seconds"
      - "Lighthouse score: >90"
      - "TypeScript strict mode: 100% compliance"
      - "Zero console.log statements in commits"

# =============================================================================
# ROLA 3: DEVICE_INTEGRATION_SPECIALIST
# =============================================================================

  - id: DEVICE_INTEGRATION_SPECIALIST
    name: "Device Integration Specialist"
    emoji: "🔌"
    description: |
      Specjalista ds. integracji urządzeń przemysłowych, 
      protokołów komunikacyjnych i zarządzania konfiguracją.
    
    responsibilities:
      - Device adapter implementation (DeviceAdapter trait)
      - Communication protocols (RINCMD, DINI_ASCII, TCP, Serial)
      - TCP socket connection management
      - Serial port management (Windows)
      - Device configuration management
      - Timeout handling and reconnection logic
      - Device health monitoring
      - Protocol documentation
      - Device command testing
    
    primary_paths:
      - "src-rust/src/adapters/"
      - "src-rust/src/adapters/rinstrum.rs"
      - "src-rust/src/adapters/dini_argeo.rs"
      - "src-rust/src/adapters/adapter.rs"
      - "src-rust/src/device_manager.rs"
      - "src-rust/config/devices.json"
      - "src-rust/src/models/"
      - "src-rust/miernik/src/devices.rs"
      - "src-rust/miernik/src/device.rs"
      - "src-rust/host/src/connection.rs"
    
    secondary_paths:
      - "docs/scale-connection-manager.md"
      - "docs/scale-parser.md"
      - "docs/device-protocols/"
      - "scripts/test-readgross.ps1"
      - "src-rust/tests/"
    
    rules:
      - "Implement DeviceAdapter trait for each device"
      - "Define all devices in config/devices.json"
      - "Log all device communications at INFO level minimum"
      - "Handle timeouts gracefully (no panics)"
      - "Test with real devices whenever possible"
      - "Document protocol specifications in markdown"
      - "Implement reconnection logic with exponential backoff"
      - "Validate device configuration on startup"
      - "Never hardcode IP addresses or ports"
      - "Never ignore timeout errors"
      - "Never edit configuration without validation"
      - "Always use async I/O operations"
    
    example_commands:
      - "@devices \"Add support for new scale manufacturer XYZ\""
      - "@devices \"Debug Rinstrum C320 connection timeout issue\""
      - "@devices \"Implement device auto-detection\""
      - "@devices \"Create protocol documentation for DINI_ASCII\""
      - "@devices \"Test device failover mechanisms\""
    
    success_metrics:
      - "Device connection success rate: >99%"
      - "Connection establishment time: <2 seconds"
      - "Timeout handling coverage: 100%"
      - "Protocol documentation: Complete"
      - "Device test coverage: >80%"

# =============================================================================
# ROLA 4: BUILD_AND_DEVOPS_ENGINEER
# =============================================================================

  - id: BUILD_AND_DEVOPS_ENGINEER
    name: "Build & DevOps Engineer"
    emoji: "🛠️"
    description: |
      Inżynier build pipeline'u, CI/CD, deployment i konfiguracji 
      środowiska development.
    
    responsibilities:
      - MinGW toolchain configuration (Windows)
      - Build scripts (PowerShell)
      - GitHub Actions workflows
      - Windows installer generation (Inno Setup)
      - Environment setup and troubleshooting
      - Release management
      - Versioning strategy
      - Build optimization
      - Documentation of setup procedures
    
    primary_paths:
      - "scripts/"
      - "scripts/Build-WindowsInstaller.ps1"  # ⭐ MAIN BUILD SCRIPT - ALWAYS USE!
      - "scripts/Setup-MinGW.ps1"
      - "scripts/build-rust-mingw.ps1"
      - "scripts/run-backend.ps1"
      - ".github/workflows/"
      - "installer/"
      - "BUILD_WINDOWS.md"
      - ".cursorrules"  # Build instructions
      - "AI_RULES.md"  # Build rules
    
    secondary_paths:
      - "Cargo.toml"
      - "package.json"
      - "TESTING_AND_DEPLOYMENT.md"
      - "PLAN_WINDOWS_INSTALLER.md"
    
    rules:
      - "ALWAYS use scripts/Build-WindowsInstaller.ps1 for rebuilding - NEVER use direct cargo build!"
      - "Use Rust 1.91.1 stable-x86_64-pc-windows-gnu"
      - "MinGW path: D:\\msys64\\mingw64\\bin"
      - "Document all environment variables (CARGO_TARGET_*)"
      - "Version in both Cargo.toml and package.json"
      - "Auto-increment patch version (X.Y.Z → X.Y.Z+1) for main branch builds"
      - "Use GitHub Actions for every major step"
      - "Never commit build artifacts"
      - "Never hardcode absolute paths"
      - "Never skip dependency checks"
      - "Always test build scripts locally first"
      - "Document MinGW setup step-by-step"
      - "Direct cargo commands will fail - always use build scripts!"
    
    example_commands:
      - "@build \"Rebuild project using Build-WindowsInstaller.ps1\""
      - "@build \"Fix MinGW linker path in build-rust-mingw.ps1\""
      - "@build \"Create GitHub Actions workflow for automated releases\""
      - "@build \"Generate Windows installer with auto-incremented version\""
      - "@build \"Debug dlltool not found error\""
      - "@build \"Optimize build time with incremental compilation\""
    
    success_metrics:
      - "Build time: <5 minutes (release)"
      - "CI/CD pipeline success rate: >99%"
      - "Setup documentation completeness: 100%"
      - "Cross-platform build coverage: Windows"
      - "Installer generation: Fully automated"

# =============================================================================
# ROLA 5: QA_AND_TESTING_SPECIALIST
# =============================================================================

  - id: QA_AND_TESTING_SPECIALIST
    name: "QA & Testing Specialist"
    emoji: "🧪"
    description: |
      Specjalista ds. testowania, zapewniania jakości kodu, 
      coverage i performance testing.
    
    responsibilities:
      - Unit tests (Vitest frontend, Cargo backend)
      - Integration tests (API + device communication)
      - E2E tests (Playwright)
      - Performance testing and benchmarks
      - Device connection testing
      - Code coverage tracking and reporting
      - Regression testing
      - Test infrastructure setup
    
    primary_paths:
      - "src/test/"
      - "src-rust/tests/"
      - "e2e/"
      - "vitest.config.ts"
      - "playwright.config.ts"
    
    secondary_paths:
      - "src/test/components/"
      - "src/test/services/"
      - "src/test/hooks/"
      - "package.json (test scripts)"
    
    rules:
      - "Target >80% code coverage"
      - "Write unit tests for every public function"
      - "Create E2E tests for critical user workflows"
      - "Test error scenarios (timeouts, disconnections)"
      - "Include performance benchmarks for critical paths"
      - "Test with real devices in CI when possible"
      - "Never test only happy paths"
      - "Never ignore flaky tests"
      - "Never skip test coverage analysis"
      - "Always test async operations properly"
    
    example_commands:
      - "@testing \"Write unit tests for DeviceConfigForm (90% coverage)\""
      - "@testing \"Create E2E test for complete scale reading workflow\""
      - "@testing \"Performance test API under 100 concurrent connections\""
      - "@testing \"Set up code coverage reporting in GitHub Actions\""
      - "@testing \"Debug flaky test in device manager\""
    
    success_metrics:
      - "Unit test coverage: >80%"
      - "Integration test coverage: >70%"
      - "E2E test coverage: All critical workflows"
      - "Flaky test rate: 0%"
      - "Test execution time: <10 minutes"

# =============================================================================
# CONTEXT PATTERNS - Automatyczne Załadowanie Kontekstu
# =============================================================================

context_patterns:
  backend:
    description: "Backend context (Rust code, dependencies, tests)"
    include:
      - "src-rust/**"
      - "Cargo.toml"
      - "Cargo.lock"
    exclude:
      - "src-rust/target/**"
      - "**/*.lock"
  
  frontend:
    description: "Frontend context (React, TypeScript, styles)"
    include:
      - "src/**"
      - "package.json"
      - "package-lock.json"
      - "vite.config.ts"
      - "tailwind.config.ts"
      - "tsconfig.json"
    exclude:
      - "src/node_modules/**"
      - "dist/**"
  
  devices:
    description: "Device integration context"
    include:
      - "src-rust/src/adapters/**"
      - "config/devices.json"
      - "src-rust/src/device_manager.rs"
      - "docs/device-protocols/**"
    exclude: []
  
  build:
    description: "Build and DevOps context"
    include:
      - "scripts/**"
      - ".github/workflows/**"
      - "installer/**"
      - "BUILD_WINDOWS.md"
    exclude: []
  
  testing:
    description: "Testing context"
    include:
      - "src/test/**"
      - "src-rust/tests/**"
      - "e2e/**"
      - "vitest.config.ts"
      - "playwright.config.ts"
    exclude: []

# =============================================================================
# NAMING CONVENTIONS
# =============================================================================

naming_conventions:
  rust:
    modules: "snake_case"
    examples: "device_manager, rinstrum_adapter, error_types"
    
    types_and_structs: "PascalCase"
    examples: "DeviceManager, RinstrumAdapter, ConnectionError"
    
    functions_and_methods: "snake_case"
    examples: "read_weight, connect_device, handle_timeout"
    
    constants: "SCREAMING_SNAKE_CASE"
    examples: "DEFAULT_TIMEOUT, MAX_RETRIES, DEFAULT_PORT"
    
    traits: "PascalCase"
    examples: "DeviceAdapter, ConnectionHandler"
    
    lifetime_parameters: "'lowercase"
    examples: "'a, 'static"
  
  typescript:
    components: "PascalCase"
    examples: "DeviceStatus, ScaleReader, ConfigPanel"
    
    functions: "camelCase"
    examples: "readWeight, connectDevice, formatValue"
    
    constants: "SCREAMING_SNAKE_CASE"
    examples: "DEFAULT_TIMEOUT, API_BASE_URL"
    
    types: "PascalCase"
    examples: "DeviceConfig, ScaleReading, ApiResponse"
    
    variables: "camelCase"
    examples: "deviceStatus, currentWeight"
    
    files: "kebab-case"
    examples: "device-status.tsx, scale-reader.tsx"
  
  css:
    classes: "kebab-case"
    examples: "device-status, scale-reader-container"
    
    variables: "kebab-case with dashes"
    examples: "--primary-color, --spacing-lg"

# =============================================================================
# BUILD COMMANDS REFERENCE
# =============================================================================

build_commands:
  main_build:
    description: "⭐ MAIN BUILD - Build complete project (backend + frontend + installer)"
    command: ".\\scripts\\Build-WindowsInstaller.ps1"
    note: "ALWAYS use this for rebuilding! Handles MinGW, AVG issues, version incrementing"
    output: "release/ScaleCmdBridge-Setup-x64-v<VERSION>.exe"
  
  backend:
    description: "Build Rust backend (advanced - use main_build instead)"
    command: ".\\build-rust-mingw.ps1 --release --skip-tests"
    note: "⚠️ Not recommended - use main_build script instead"
    output: "src-rust/target/x86_64-pc-windows-gnu/release/scaleit-bridge.exe"
  
  backend_debug:
    description: "Build Rust backend (debug mode)"
    command: ".\\build-rust-mingw.ps1"
    note: "⚠️ Requires MinGW environment setup"
    output: "src-rust/target/x86_64-pc-windows-gnu/debug/scaleit-bridge.exe"
  
  backend_test:
    description: "Run Rust backend tests"
    command: ".\\test-rust-mingw.ps1"
    note: "Or: cd src-rust && cargo test (requires MinGW setup)"
  
  backend_clippy:
    description: "Run Rust linter (clippy)"
    command: "cd src-rust && cargo clippy -- -D warnings"
    note: "Requires MinGW environment setup"
  
  frontend:
    description: "Build React frontend (production) - advanced use"
    command: "npm run build"
    note: "Usually handled by main_build script"
    output: "dist/"
  
  frontend_dev:
    description: "Run frontend development server"
    command: "npm run dev"
    port: "5173"
  
  frontend_test:
    description: "Run frontend unit tests (Vitest)"
    command: "npm run test"
  
  frontend_e2e:
    description: "Run E2E tests (Playwright)"
    command: "npm run test:e2e"

# =============================================================================
# FOLDER STRUCTURE GUIDE
# =============================================================================

folder_structure:
  root:
    - "src-rust/           # Rust Backend (Actix-web server)"
    - "src/                # React Frontend (TypeScript)"
    - "config/             # Configuration files (devices.json)"
    - "scripts/            # Build scripts (PowerShell)"
    - "e2e/                # E2E tests (Playwright)"
    - "docs/               # Documentation (markdown)"
    - ".github/            # GitHub Actions workflows"
    - "installer/          # Windows installer (Inno Setup)"
  
  src_rust:
    - "src/main.rs                  # Server entry point"
    - "src/device_manager.rs        # Device orchestration"
    - "src/error.rs                 # Custom error types"
    - "src/models/                  # Request/Response DTOs"
    - "src/adapters/                # Device adapters"
    - "src/adapters/rinstrum.rs     # Rinstrum adapter"
    - "src/adapters/dini_argeo.rs   # Dini Argeo adapter"
    - "host/                        # Host library (TCP/Serial connections)"
    - "host/src/connection.rs       # Connection management"
    - "miernik/                     # Miernik library (device abstractions)"
    - "miernik/src/devices.rs       # Device implementations"
    - "miernik/src/device.rs        # Device trait"
    - "tests/                       # Integration tests"
    - "Cargo.toml                   # Workspace dependencies"
  
  src_react:
    - "pages/                       # React pages"
    - "pages/Index.tsx              # ⭐ MAIN PAGE"
    - "components/                  # React components"
    - "services/                    # API client services"
    - "context/                     # State management"
    - "hooks/                       # Custom React hooks"
    - "types/                       # TypeScript definitions"
    - "test/                        # Unit tests"
    - "App.tsx                      # React Router setup"
    - "main.tsx                     # Entry point"

# =============================================================================
# IMPORTANT FILE REFERENCES
# =============================================================================

critical_files:
  config:
    - path: "config/devices.json"
      description: "Device definitions and configurations"
      role: "DEVICE_INTEGRATION_SPECIALIST"
      format: "JSON"
  
  frontend:
    - path: "src/pages/Index.tsx"
      description: "⭐ MAIN PAGE - ALWAYS UPDATE WHEN ADDING COMPONENTS!"
      role: "FRONTEND_DEVELOPER"
      format: "TSX"
    
    - path: "AI_RULES.md"
      description: "Frontend coding conventions and guidelines"
      role: "FRONTEND_DEVELOPER"
      format: "Markdown"
  
  backend:
    - path: "src-rust/src/main.rs"
      description: "Server initialization, routes, middleware"
      role: "BACKEND_ARCHITECT"
      format: "Rust"
    
    - path: "BACKEND_GUIDELINES.md"
      description: "Backend architecture and best practices"
      role: "BACKEND_ARCHITECT"
      format: "Markdown"
  
  build:
    - path: "scripts/Setup-MinGW.ps1"
      description: "MinGW toolchain configuration"
      role: "BUILD_AND_DEVOPS_ENGINEER"
      format: "PowerShell"
    
    - path: "BUILD_WINDOWS.md"
      description: "Windows build setup and troubleshooting"
      role: "BUILD_AND_DEVOPS_ENGINEER"
      format: "Markdown"
  
  testing:
    - path: "vitest.config.ts"
      description: "Frontend test framework configuration"
      role: "QA_AND_TESTING_SPECIALIST"
      format: "TypeScript"
    
    - path: "playwright.config.ts"
      description: "E2E test framework configuration"
      role: "QA_AND_TESTING_SPECIALIST"
      format: "TypeScript"

# =============================================================================
# CODE QUALITY STANDARDS
# =============================================================================

code_quality:
  rust:
    linter: "clippy"
    formatter: "rustfmt"
    warnings_as_errors: true
    coverage_target: ">80%"
    rules:
      - "No unwrap() in production code"
      - "All public APIs must have rustdoc comments"
      - "Error types must implement Debug + Display"
      - "Use Result<T, E> for fallible operations"
  
  typescript:
    strict_mode: true
    no_any_type: true
    coverage_target: ">80%"
    rules:
      - "No console.log() in commits"
      - "TypeScript strict: enabled"
      - "No unused variables"
      - "No unused imports"
  
  css:
    formatter: "prettier"
    rules:
      - "Use Tailwind CSS only"
      - "No custom CSS in components"
      - "Use Tailwind for responsive design"

# =============================================================================
# COMMUNICATION PROTOCOL
# =============================================================================

communication:
  command_format: "@role \"your message\""
  
  valid_roles:
    - "@backend    - Backend architecture, Rust optimization"
    - "@frontend   - React components, UI, state management"
    - "@devices    - Device integration, protocols, TCP/Serial"
    - "@build      - Build pipeline, CI/CD, Windows setup"
    - "@testing    - Tests, QA, code coverage"
  
  examples:
    - "@backend \"Review main.rs architecture\""
    - "@frontend \"Create new component named DeviceConfig\""
    - "@devices \"Debug Rinstrum connection timeout\""
    - "@build \"Fix MinGW toolchain path\""
    - "@testing \"Write unit tests for DeviceManager\""
  
  context_auto_loading:
    enabled: true
    description: "Cursor automatically loads relevant files based on @role prefix"

# =============================================================================
# GIT WORKFLOW CONVENTIONS
# =============================================================================

git_workflow:
  branch_naming:
    feature: "feature/short-description"
    bugfix: "bugfix/short-description"
    refactor: "refactor/short-description"
    docs: "docs/short-description"
  
  commit_messages:
    format: "type(scope): subject"
    types:
      - "feat: New feature"
      - "fix: Bug fix"
      - "docs: Documentation"
      - "style: Code style (formatting)"
      - "refactor: Code refactoring"
      - "test: Adding tests"
      - "chore: Maintenance"
    
    examples:
      - "feat(backend): Add connection pooling to DeviceManager"
      - "fix(frontend): Fix TypeScript error in DeviceStatus component"
      - "docs(devices): Add RINCMD protocol documentation"
      - "test(backend): Add integration tests for TCP connections"
  
  pull_request:
    template: |
      ## Description
      Brief description of changes
      
      ## Type of Change
      - [ ] Bug fix
      - [ ] New feature
      - [ ] Breaking change
      - [ ] Documentation update
      
      ## Testing
      How to test the changes
      
      ## Checklist
      - [ ] Code follows project style
      - [ ] Tests added/updated
      - [ ] Documentation updated
      - [ ] No console.log() left in code

# =============================================================================
# DEPENDENCIES AND VERSIONS
# =============================================================================

versions:
  project: "0.1.3"
  rust: "1.91.1"
  rust_toolchain: "stable-x86_64-pc-windows-gnu"
  node: "18.x"
  react: "18.x"
  vite: "5.x"
  tailwind: "3.x"
  typescript: "5.x"

versioning_strategy:
  main_branch:
    auto_increment: true
    pattern: "X.Y.Z → X.Y.Z+1 (patch version)"
    cargo_toml: "Automatically updated by Build-WindowsInstaller.ps1"
    installer_name: "ScaleCmdBridge-Setup-x64-v0.1.3.exe"
  
  feature_branches:
    auto_increment: false
    pattern: "Version stays same, branch name appended"
    installer_name: "ScaleCmdBridge-Setup-x64-v0.1.3-<branch-name>.exe"
  
  manual_version:
    update_files:
      - "src-rust/Cargo.toml"
      - "package.json (if applicable)"

critical_crates:
  - "actix-web: 4.x       # Web framework"
  - "tokio: 1.x           # Async runtime"
  - "serde: 1.x           # Serialization"
  - "log: 0.4             # Logging"
  - "thiserror: 1.x       # Error types"

critical_npm_packages:
  - "react: 18.x          # UI framework"
  - "react-router-dom: 6.x  # Routing"
  - "tailwindcss: 3.x     # CSS framework"
  - "shadcn/ui: latest    # UI components"
  - "typescript: 5.x      # Language"
  - "vitest: latest       # Testing"

# =============================================================================
# DOCUMENTATION LOCATIONS
# =============================================================================

documentation:
  main: "README.md"
  
  frontend:
    - "AI_RULES.md"
    - "docs/FRONTEND_GUIDELINES.md"
  
  backend:
    - "BACKEND_GUIDELINES.md"
    - "BUILD_WINDOWS.md"
  
  devices:
    - "docs/scale-connection-manager.md"
    - "docs/scale-parser.md"
    - "docs/device-protocols/"
  
  build:
    - "BUILD_WINDOWS.md"
    - "TESTING_AND_DEPLOYMENT.md"
    - "PLAN_WINDOWS_INSTALLER.md"
  
  api:
    - "swagger.yaml"
    - "docs/API.md"

# =============================================================================
# NEXT STEPS FOR IMPLEMENTATION
# =============================================================================

implementation_guide:
  step_1:
    title: "Create .cursor directory"
    command: "mkdir .cursor"
  
  step_2:
    title: "Save this file as .cursor/rules.yaml"
    command: "Save content to .cursor/rules.yaml"
  
  step_3:
    title: "Commit to repository"
    command: "git add .cursor/rules.yaml && git commit -m 'docs: Add Cursor agent configuration'"
  
  step_4:
    title: "Create role-specific documentation"
    action: "Create supplementary docs for each role"
  
  step_5:
    title: "Train team on usage"
    action: "Share @role command format with development team"

# =============================================================================
# ROLE SELECTION FLOWCHART
# =============================================================================

role_selection:
  architecture_or_optimization: "@backend"
  react_components_or_ui: "@frontend"
  device_integration_or_protocols: "@devices"
  build_ci_cd_or_deployment: "@build"
  testing_or_quality_assurance: "@testing"

# =============================================================================
# END OF CURSOR RULES
# =============================================================================

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tkogut) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
