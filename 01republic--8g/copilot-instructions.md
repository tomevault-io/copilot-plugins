## 8g

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 📚 Documentation Index

For comprehensive project information, refer to the following documents:

- **[Documentation Principles](.claude/DOCUMENTATION_PRINCIPLES.md)** - 📐 문서화 작성 원칙 및 가이드라인 (필독!)
- **[Product Requirements Document (PRD)](.claude/PRD.md)** - 전체 프로젝트 기획, 요구사항, 아키텍처 개요
- **[Feature Specifications](.claude/product-specs/README.md)** - 기능별 상세 명세 (사전식 인덱스)
- **[Technical Architecture](#architecture-overview)** - 기술 아키텍처 (이 문서 하단)
- **[Workflow Execution Architecture](WORKFLOW_EXECUTION_ARCHITECTURE.md)** - 워크플로우 실행 상세 (있는 경우)

**문서화 원칙 (중요!)**:

- ✅ 모든 문서는 한글로 작성 (CLAUDE.md 제외)
- ✅ 모든 문서는 200줄 이하로 작성
- ✅ 개발 완료 직후 문서 최신화 필수

**빠른 참조:**

- 새 기능 개발 시: PRD에서 요구사항 확인 → Feature Specs에서 상세 확인 → 이 문서에서 구현 방법 확인
- 버그 수정 시: Feature Specs에서 기대 동작 확인 → 이 문서에서 관련 파일 찾기
- 코드 리뷰 시: Feature Specs에서 명세 준수 확인 → PRD에서 비기능 요구사항 확인
- 문서 작성 시: [Documentation Principles](.claude/DOCUMENTATION_PRINCIPLES.md) 필독!

---

## Project Overview

8G is a visual workflow automation platform that enables users to design complex web scraping and data extraction workflows through a drag-and-drop interface. The platform executes workflows via the 8G Extension (browser extension SDK). The project was recently refactored to focus purely on workflow builder functionality (branch: `only-builder`).

**상세 정보**: [PRD - Section 1: Product Overview](.claude/PRD.md#1-product-overview)

## Development Commands

### Essential Commands

```bash
# Development server (port 3000)
pnpm run dev

# Production build and start
pnpm run start

# Build only
pnpm run build

# Type checking
pnpm run typecheck

# Code formatting
pnpm run format
pnpm run format:check

# Generate database entities from schema
pnpm run db:generate-entities
```

### Environment Setup

Create a `.env` file with database configuration:

```env
DB_HOST=localhost
DB_PORT=3306
DB_USER=myuser
DB_PASSWORD=mypassword
DB_NAME=payplo_staging
NEXT_PUBLIC_CARD_SIGN_KEY=spurstodo
```

### Database Setup

Create the workflow metadata table manually (no migration tool currently):

```sql
CREATE TABLE IF NOT EXISTS integration_app_workflow_metadata (
  id INT NOT NULL AUTO_INCREMENT,
  description VARCHAR(255),
  meta JSON NOT NULL,
  created_at DATETIME(6) DEFAULT CURRENT_TIMESTAMP(6),
  updated_at DATETIME(6) DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6),
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Architecture Overview

**상세 아키텍처 정보**: [PRD - Section 2: System Architecture](.claude/PRD.md#2-system-architecture)

### Tech Stack

- **Frontend**: React 19, React Router 7 (config-based routing)
- **Server**: Express 5, React Router server-side rendering
- **Graph Visualization**: @xyflow/react (React Flow v12), dagre layout
- **Database**: MySQL 3306, TypeORM 0.3.27
- **UI Components**: shadcn/ui (Radix UI primitives), TailwindCSS 4
- **Schema Validation**: Zod 3.25.76
- **Workflow Execution SDK**: scordi-extension v1.16.0

### Key Architectural Patterns

#### 1. Workflow Data Flow

**Loading a workflow:**

1. React Router `loader()` in `app/routes/workflow-builder.tsx` calls `findWorkflowMetadata(id)`
2. TypeORM fetches workflow with `meta: FormWorkflow` JSON structure from database
3. `convertWorkflowToNodesAndEdges()` in `app/client/admin/workflowBuilder/utils/workflowConverter.ts` transforms workflow JSON → React Flow nodes/edges
4. Dagre layout algorithm auto-arranges nodes hierarchically
5. React Flow renders the interactive graph

**Saving a workflow:**

1. `buildWorkflowJson()` in `app/models/workflow/WorkflowBuilder.ts` converts React Flow graph → Workflow JSON
2. Form submission to route `action()` with FormData
3. `upsertWorkflowMetadata()` in `app/.server/services/workflow/` creates or updates database record
4. Redirects to workflows list page

**Executing a workflow:**

1. Workflow with `vars` field sent to Extension via `EightGClient.collectWorkflow()`
2. Extension handles variable substitution using `${vars.varName}` template syntax
3. Results displayed in `ResultPanel.tsx` component

#### 2. Block System Architecture

All workflow blocks are defined by **Zod v3 schemas** from the `scordi-extension` package (`AllBlockSchemas`). The `GenericBlockNode` component dynamically renders any block type by:

1. Parsing the Zod schema via `schema-parser.ts` to extract field definitions
2. Rendering appropriate field input components (StringFieldBlock, NumberFieldBlock, EnumFieldBlock, etc.)
3. Validating block configuration at runtime with Zod

This allows adding new block types in the SDK without modifying the UI code.

#### 3. Workflow JSON Structure

The core workflow format (`FormWorkflow` type):

```typescript
{
  version: "1.0",
  start: "step_id",  // Starting step
  steps: [
    {
      id: "step_id",
      block: { name: "block-name", /* block config */ },
      repeat?: { forEach: "$.path", count: 5 },  // Optional loops
      switch?: [                                  // Conditional branching
        { when: { equals: {...} }, next: "other_step" }
      ],
      next: "next_step",
      delayAfterMs: 500,
      retry?: { attempts: 3 },
      timeoutMs: 5000
    }
  ],
  targetUrl: "https://example.com",
  parser?: { expression: "$.steps.final.result" },  // Output transformation
  vars?: { key: "value" }  // Variables for ${vars.key} substitution
}
```

#### 4. Conditional Edge System

Edges between workflow nodes can have conditional logic stored in `edge.data.when`. The `EdgeConfigDialog` component provides a UI for building complex conditions:

- **Comparison operators**: equals, regex, contains
- **Logical operators**: AND, OR combinations
- **Existence checks**: exists
- **JSONPath expressions** for accessing previous step results

Conditions are evaluated at runtime by the workflow engine.

#### 5. Server-Side Architecture

The Express server (`server.js`) integrates React Router's server handler:

- **Development**: Vite middleware for HMR
- **Production**: Serves pre-built static files
- **WebSocket server** runs on the same port for real-time features
- **TypeORM DataSource** initialized on first request (lazy initialization)
- Development mode enables TypeORM query logging

## Important Concepts

### React Router 7 Config-Based Routing

This project uses **config-based routing** (not file-based). Routes are defined in `app/routes.ts` with explicit layout nesting:

```typescript
route("/workflow-builder/:workflowId?", "./routes/workflow-builder.tsx");
```

Route files in `app/routes/*.tsx` export:

- `loader()` - Server-side data fetching
- `action()` - Form submission handlers
- `default` - React component

### Workflow Execution via Extension SDK

All workflow execution happens through the **8G browser extension**. The client-side code:

1. Calls `EightGClient.collectWorkflow({ targetUrl, workflow })`
2. Extension receives workflow, opens browser tab to targetUrl
3. Extension executes each workflow step sequentially
4. Results returned as JSON with step outputs

**Important**: The extension must be installed and connected for workflow execution to work.

### Block Types (20+ supported)

**Data extraction**: get-text, attribute-value, get-element-data, get-value-form
**Interaction**: event-click, keypress, scroll
**Control flow**: wait, element-exists
**Integration**: fetch-api, ai-parse-data, transform-data
**Loops**: forEach (iterate arrays), count (fixed repetitions)
**Branching**: switch conditions with JSON expressions

**상세 블록 명세**: [Feature Specs - F-002: Block System](.claude/product-specs/F-002-block-system.md)
See `WORKFLOW_EXECUTION_ARCHITECTURE.md` for complete block documentation (if exists).

### Variable Substitution

Variables use `${vars.varName}` template syntax. The Extension handles variable resolution:

- Static variables from `workflow.vars` field
- Dynamic variables from previous step results (using JSONPath expressions)
- Context variables in loops (`$.forEach.item`, `$.loop.index`)

## Code Organization Patterns

### Server-Only Code (.server directory)

Code in `app/.server/` is **never bundled to the client**. This includes:

- Database entities and TypeORM configuration
- Service layer functions (workflow CRUD operations)
- Sensitive business logic

### Models vs Components

- **`app/models/`**: Business logic, data transformation, execution logic (WorkflowBuilder, WorkflowRunner)
- **`app/components/`**: Reusable UI components
- **`app/client/`**: Page-level components organized by section (admin, private, public)

### Type-Safe Workflow Building

The workflow builder maintains type safety through:

1. Zod schemas define block structure at runtime
2. TypeScript types inferred from Zod schemas
3. React Flow typed nodes and edges
4. Form validation before database persistence

## Development Workflow

### Adding a New Block Type

1. Add block schema to `scordi-extension` package (external SDK)
2. No UI changes needed - `GenericBlockNode` auto-generates UI from schema
3. Update block labels in `app/client/admin/workflowBuilder/nodes/index.ts` if needed

### Modifying Workflow Execution Logic

1. Core execution happens in the extension (external)
2. UI-side: Update `WorkflowBuilder.ts` for JSON conversion
3. Runner-side: Update `WorkflowRunner.ts` for client-side orchestration
4. Variables are passed via `workflow.vars` field to Extension

### Testing Workflows

1. Build workflow in UI (`/workflow-builder`)
2. Click "Run Workflow" to execute via extension
3. Results appear in `ResultPanel`
4. Check browser console for extension communication logs
5. Inspect workflow JSON in browser DevTools Network tab

### Testing JSONata Expressions (transform-data block)

1. Add a `transform-data` block to your workflow
2. In the block configuration modal:
   - Select **sourceData**: Choose a previous step's output (e.g., `${steps.node_xxx.result.data}`)
   - Write **expression**: JSONata expression to transform the data
3. Use the built-in test panel:
   - Execution results auto-load into test input
   - Click "표현식 실행" to test the expression
   - See results or errors immediately
4. Common JSONata patterns:
   - Filter: `*.data.data.results[membershipType = "owner"]`
   - Extract fields: `*.data.data.results[].{ "id": id, "name": name }`
   - Get unique values: `$distinct(data.users.*.user_root.*.value.space_view_pointers[].spaceId)`
   - Nested navigation: `data.users.*.user_root.*.value.space_view_pointers[]`

## Important Files

### Core Workflow Files

- `app/client/admin/workflowBuilder/WorkflowBuilderPage.tsx` - Main builder component
- `app/models/workflow/WorkflowBuilder.ts` - Converts React Flow graph → Workflow JSON
- `app/models/workflow/WorkflowRunner.ts` - Executes workflows via SDK
- `app/client/admin/workflowBuilder/utils/workflowConverter.ts` - Bidirectional workflow ↔ nodes/edges conversion
- `app/client/admin/workflowBuilder/utils/autoLayout.ts` - Dagre hierarchical layout algorithm

### Database Layer

- `app/.server/db/config.ts` - TypeORM DataSource configuration
- `app/.server/db/entities/IntegrationAppWorkflowMetadata.ts` - Workflow entity
- `app/.server/services/workflow/` - Workflow CRUD services

### Routing

- `app/routes.ts` - React Router configuration
- `app/routes/workflow-builder.tsx` - Builder route with loader/action
- `app/routes/workflows.tsx` - Workflows list route

### Edge Conditions

- `app/client/admin/workflowBuilder/edges/EdgeConfigDialog/` - Condition builder UI
- `app/client/admin/workflowBuilder/utils/conditionUtils.ts` - Condition rendering helpers

### Field Blocks (Block Configuration UI)

- `app/client/admin/workflowBuilder/nodes/fieldBlock/ExpressionFieldBlock.tsx` - JSONata expression testing UI for transform-data blocks
  - Real-time JSONata expression evaluation
  - Auto-loads execution results from previous steps
  - Supports both `context.steps` and `result.steps` data structures
  - Uses EightGClient helper functions for data access
- `app/client/admin/workflowBuilder/nodes/BlockActionHandlerModal.tsx` - Block configuration modal with dynamic field rendering

## Common Tasks

### Run a Single Test

(No test framework currently configured)

### Build for Production

```bash
pnpm run build
NODE_ENV=production node server.js
```

### Check Types

```bash
pnpm run typecheck
```

### Debug Workflow Execution

1. Open browser DevTools console
2. Look for messages prefixed with `[8G Extension]`
3. Check Network tab for extension communication
4. Inspect `result` object returned from `collectWorkflow()`

### Access Database

Connection is established via TypeORM using `.env` credentials. Use TypeORM query builder or raw SQL via DataSource.

## Notes

- Package manager: **pnpm** (not npm or yarn)
- Database migrations: Not implemented - use manual SQL
- Multi-tenancy: Not currently implemented in workflow builder
- WebSocket: Server setup exists but not actively used
- The project focuses exclusively on workflow builder functionality after recent refactoring

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/01Republic)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/01Republic)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
