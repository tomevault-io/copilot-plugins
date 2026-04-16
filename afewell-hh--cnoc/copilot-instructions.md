## cnoc

> **Mission**: Enterprise-grade cloud networking operations infrastructure with bootable ISO deployment

# Cloud NetOps Command (CNOC) Project Context

**Mission**: Enterprise-grade cloud networking operations infrastructure with bootable ISO deployment
**Methodology**: FORGE (Formal Operations with Rigorous Guaranteed Engineering) + ruv-swarm coordination
**Status**: Production Development - Go-based CLI with Kubernetes architecture and distributed coordination
**Branch**: modernization/k8s-foundation

## REQUIRED:
- COMMIT FILE CHANGES FREQUENTLY PER GIT BEST PRACTICES
- UPDATE RELATED GITHUB ISSUES WITH EVERY COMMIT
- ALL AGENTS HAVE ACCESS TO GH CLI UTILITY TO SIMPLIFY INTERACTIONS WITH GITHUB SERVICES INCLUDING ISSUE TICKETS
- TESTS THAT VALIDATE FULL FUNCTIONALITY MUST BE WRITTEN BEFORE ANY CODE CHANGES
- PASSING VALIDATION TESTS ARE MANDATORY FOR ALL CODE CHANGES
- E2E BROWSER TESTING WITH PLAYWRIGHT IS REQUIRED FOR ALL GUI CHANGES (see /test/e2e/playwright/README.md)

## System Architecture Overview

**CNOC (Cloud NetOps Command)**: The new production system based on Go CLI, Kubernetes infrastructure, and domain-driven design patterns. This system replaces the original HNP prototype.

**HNP (Hedgehog NetBox Plugin)**: Original Python/Django prototype system preserved for reference on experimental/main branch.

## CNOC Technical Stack (Production Architecture + ruv-swarm Enhancement)
- Core: Go 1.24 backend with Gorilla Mux router and FORGE validation coverage
- Frontend: Bootstrap 5.3 with vanilla JavaScript and server-rendered Go HTML templates
- Infrastructure: Kubernetes (K3s), GitOps, PostgreSQL 15, Redis 7
- Architecture: Domain-driven design with monolithic deployment pattern and FORGE evidence validation
- Testing Strategy: FORGE Symphony movements - red-green-refactor, mutation testing, comprehensive validation
- E2E Testing: Playwright browser automation for GUI validation (test/e2e/playwright/)
- Deployment: HOSS bootable ISO with pre-configured components or direct K3s deployment
- Components: ArgoCD, Prometheus, Grafana, cert-manager with FORGE monitoring suites
- **Coordination**: ruv-swarm distributed agent coordination (25+ MCP tools, 4 topologies)
- **Performance**: 2.8-4.4x speed improvements through parallel coordination while maintaining FORGE quality

## Technology Architecture Decisions

### Current Production Stack (CNOC)
**Rationale**: Proven Go+Bootstrap stack provides excellent performance (<200ms API responses), maintainability, and operational simplicity. This pragmatic approach enables rapid delivery of HNP feature parity while maintaining enterprise-grade reliability.

## ruv-swarm Integration Architecture

### FORGE + Swarm Coordination
**Integration Strategy**: Enhancement-over-replacement approach preserving proven FORGE methodology while adding distributed coordination capabilities.

### Swarm-Enhanced Agent Capabilities
```yaml
Enhanced_Agents:
  coordination-orchestrator: 
    - ruv-swarm topology selection (mesh/hierarchical/star/ring)
    - Intelligent agent spawning and workload distribution
    - Performance monitoring and bottleneck resolution
    
  testing-validation-engineer:
    - Parallel test generation with 86.1% neural accuracy
    - Distributed test validation across multiple domains
    - Evidence aggregation from coordinated testing agents
    
  implementation-specialist:
    - Coordinated parallel implementation with dependency management
    - Test integrity protection across distributed agents
    - Centralized evidence collection and validation
```

### MCP Tool Integration
**Primary Tools**: `mcp__ruv-swarm__swarm_init`, `mcp__ruv-swarm__agent_spawn`, `mcp__ruv-swarm__task_orchestrate`
**Topology Selection**: Dynamic selection based on task complexity and resource availability
**Quality Gates**: FORGE evidence requirements strictly maintained across all swarm operations

## HNP Prototype Stack (Reference Only)
- Backend: Django 4.2, NetBox 4.3.3 plugin architecture  
- Frontend: Bootstrap 5 with progressive disclosure UI
- Integration: Kubernetes Python client, ArgoCD GitOps
- Database: PostgreSQL (shared with NetBox core)

## Development Environment (FORGE Symphony Orchestration)

**CLI Tool**: cnocfab (replacing hossfab) with FORGE validation suite (production Go application)
- **Infrastructure**: hossfab/, infrastructure/, hoss/ directories with FORGE patterns

### FORGE Symphony Task Orchestration Rules
```yaml
FORGE_Task_Routing:
  # Implementation tasks (ANY code writing) - MANDATORY FORGE test-first
  Go_Development: 
    Pattern: "*.go|*/go.mod|*/go.sum"
    FORGE_Movement: testing-validation-engineer → implementation-specialist
    Hook: forge_test_enforcement.sh (PreToolUse)
    
  Infrastructure_Code:
    Pattern: "*terraform/*|*/kubernetes/*|*/manifests/*"
    Process: infrastructure-deployment-specialist + testing-validation-engineer
    Validation: end-to-end deployment testing required
    
  CLI_Development:
    Pattern: "*cnocfab*|*/cmd/*|*/cli/*"
    Process: testing-validation-engineer → implementation-specialist
    Requirements: command testing, integration testing, error path testing
    
  Domain_Modeling:
    Pattern: "*models*|*domain*|*types*"
    Process: model-driven-architect + testing-validation-engineer
    Validation: bounded context definition + test scenarios
    
  GUI_Development:
    Pattern: "*.html|*/templates/*|*/web/*"
    Process: testing-validation-engineer (E2E Playwright tests) → implementation-specialist
    Requirements: Browser automation tests in test/e2e/playwright/
    Validation: npm run test:e2e must pass before commit
```

### E2E Testing with Playwright
```yaml
E2E_Testing_Requirements:
  Location: test/e2e/playwright/
  Documentation: test/e2e/playwright/README.md
  
  Running_Tests:
    All_Tests: "cd test/e2e/playwright && npm run test:e2e"
    Specific_Suite: "npm run test:e2e:badges"
    Debug_Mode: "npm run test:e2e:debug"
    With_Browser: "npm run test:e2e:headed"
  
  Test_Categories:
    Navigation: badges.spec.ts - Badge counts and updates
    Dashboard: metrics.spec.ts - Dashboard state validation  
    Repositories: management.spec.ts - CRUD operations
  
  Writing_New_Tests:
    Pattern: FORGE TDD - RED → GREEN → REFACTOR
    File_Naming: feature-name.spec.ts
    Selectors: Use data-testid and data-badge attributes
    Organization: Group by feature in tests/ subdirectories
  
  Required_For:
    - Any GUI template changes
    - JavaScript functionality changes
    - New interactive features
    - Bug fixes affecting user interface
```

### Agent Selection Logic for Project Types (Enhanced with ruv-swarm)
```yaml
CNOC_Tasks_Enhanced: 
  - Go backend development → coordination-orchestrator → parallel testing-validation-engineer + implementation-specialist
  - Infrastructure deployment → coordination-orchestrator → infrastructure-deployment-specialist + kubernetes-gitops-specialist
  - CLI development → coordination-orchestrator → parallel testing-validation-engineer + implementation-specialist  
  - Domain modeling → coordination-orchestrator → model-driven-architect + testing-validation-engineer
  - Complex multi-component development → coordination-orchestrator → swarm coordination with multiple specialized agents

Swarm_Coordination_Patterns:
  - Simple tasks: star topology with central coordination
  - Complex development: hierarchical topology with layered coordination
  - Parallel implementation: mesh topology for peer coordination
  - Sequential validation: ring topology for ordered quality gates
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/afewell-hh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
