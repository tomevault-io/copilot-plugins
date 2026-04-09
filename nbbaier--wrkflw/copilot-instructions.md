## wrkflw

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**wrkflw** is a TypeScript-first workflow engine for Val Town, inspired by Mastra. It enables building strongly-typed, composable workflows that run on Val Town's serverless platform with SQLite-backed durable state.

The engine features full TypeScript type inference across workflow definitions, runtime validation via Zod schemas, and supports all Val Town trigger types (HTTP, Cron, Email).

## Development Environment

### Running Examples

```bash
# Run a simple workflow example
deno run --allow-all examples/simple-workflow.ts

# Run the API server with canvas visualizer (local development)
deno run --allow-all examples/api-server-example-local.ts

# Then in another terminal, start the frontend
cd frontend
npm install
npm run dev
```

### Testing Workflows

The project doesn't have a formal test suite. Test workflows by:
1. Running example files directly with Deno
2. Using the API server + frontend visualizer for interactive testing
3. Checking SQLite state in the database after execution

### Code Quality

```bash
# Format code with Biome
deno run -A npm:@biomejs/biome format --write .

# Lint code
deno run -A npm:@biomejs/biome lint .
```

## Architecture

### Core Components

The workflow engine consists of these key parts:

1. **Step System** (`backend/step.ts`): Individual workflow steps with typed inputs/outputs
2. **Workflow Builder** (`backend/workflow.ts`): Fluent API for composing steps with type inference
3. **Execution Engine** (`backend/engine.ts`): Sequential step execution with context management
4. **Storage Layer** (`backend/storage.ts`): SQLite persistence for workflow state
5. **Run Management** (`backend/run.ts`): Workflow execution instance lifecycle

### Type Inference Pattern

The workflow builder threads types through the `.then()` chain:
- Each step's `outputSchema` must match the next step's `inputSchema`
- TypeScript enforces this at compile time
- The workflow's final output type is inferred from the last step

Example flow:
```typescript
// Step 1: string -> number
const step1 = createStep({
  inputSchema: z.object({ a: z.string() }),
  outputSchema: z.object({ b: z.number() }),
  execute: async ({ inputData }) => ({ b: 42 })
});

// Step 2: number -> boolean (MUST accept step1's output type)
const step2 = createStep({
  inputSchema: z.object({ b: z.number() }),
  outputSchema: z.object({ c: z.boolean() }),
  execute: async ({ inputData }) => ({ c: true })
});

// Workflow builder enforces type compatibility
const workflow = createWorkflow({
  inputSchema: z.object({ a: z.string() }),
  outputSchema: z.object({ c: z.boolean() })
})
  .then(step1)  // OK: accepts { a: string }
  .then(step2)  // OK: accepts { b: number } from step1
  .commit();    // Final type: { c: boolean }
```

### State Management

Workflows persist state to SQLite after each step execution:
- Each run gets a unique `runId` (UUID)
- Snapshots include: status, execution path, step results, state, errors
- Storage uses UPSERT pattern to handle interruptions
- Migrations in `backend/migrations.ts` handle schema evolution

### Execution Context

Steps receive a rich execution context:
- `inputData`: Validated input for this step
- `getStepResult(step)`: Access outputs from previous steps
- `getInitData()`: Get workflow's initial input
- `state` / `setState`: Workflow-level state (optional)
- `runId`, `workflowId`: Execution identifiers

## Project Structure

```
wrkflw/
├── backend/              # Core workflow engine
│   ├── index.ts         # Main exports
│   ├── types.ts         # TypeScript type definitions
│   ├── step.ts          # Step creation and execution
│   ├── workflow.ts      # Workflow builder with type inference
│   ├── engine.ts        # Sequential execution engine
│   ├── storage.ts       # SQLite persistence layer
│   ├── run.ts           # Run lifecycle management
│   ├── migrations.ts    # Database schema migrations
│   ├── prebuilt-steps.ts # Reusable steps (httpGet, template, etc.)
│   ├── visualize.ts     # Mermaid diagram generation
│   └── api-server.ts    # REST API for canvas visualizer
├── frontend/            # React canvas visualizer
│   ├── src/
│   │   ├── App.tsx                      # Main app container
│   │   ├── components/
│   │   │   ├── WorkflowVisualizer.tsx   # React Flow canvas
│   │   │   └── StepNode.tsx             # Custom step nodes
│   │   └── types.ts                     # Frontend types
│   └── package.json
├── examples/            # Usage examples
│   ├── simple-workflow.ts
│   ├── http-trigger.ts
│   ├── cron-trigger.ts
│   ├── prebuilt-steps-simple.ts
│   ├── workflow-visualization.ts
│   └── api-server-example-local.ts
├── deno.json            # Deno configuration
├── biome.json           # Biome formatter/linter config
└── PLAN.md              # Original implementation plan
```

## Common Development Tasks

### Creating New Steps

Steps should have clear, focused responsibilities:

```typescript
import { createStep } from "./backend/index.ts";
import { z } from "npm:zod@^3.23";

const myStep = createStep({
  id: 'my-step',                    // Unique identifier
  description: 'What this step does', // Optional description
  inputSchema: z.object({ /* ... */ }),   // Input validation
  outputSchema: z.object({ /* ... */ }),  // Output validation
  execute: async ({ inputData, getStepResult }) => {
    // Step logic here
    return { /* matches outputSchema */ };
  }
});
```

### Adding Prebuilt Steps

When adding reusable steps to `backend/prebuilt-steps.ts`:
1. Follow the existing pattern (httpGet, template, etc.)
2. Use generic Zod schemas for flexibility
3. Add clear JSDoc comments with examples
4. Export from `backend/index.ts`

### Database Migrations

When modifying workflow state schema:
1. Add migration to `backend/migrations.ts`
2. Update `WorkflowStorage.init()` to create new columns/tables
3. Test migration with existing data
4. Consider backward compatibility

### Adding API Endpoints

The API server (`backend/api-server.ts`) follows REST conventions:
- `GET /api/workflows` - List workflows
- `GET /api/workflows/:id` - Get workflow details
- `POST /api/workflows/:id/runs` - Start new run
- `GET /api/runs/:runId` - Get run status

When adding endpoints:
1. Add route matching in `handleRequest()`
2. Include CORS headers
3. Serialize data appropriately
4. Handle errors gracefully

## Val Town Specifics

### Import Patterns

The codebase uses dynamic imports for Val Town compatibility:
```typescript
// Storage imports sqlite dynamically
const { sqlite } = await import("https://esm.town/v/std/sqlite");
```

This allows the code to work both:
- In Val Town (where sqlite is available)
- Locally with Deno (where it may not be)

### Trigger Types

Workflows are designed to be called from any Val Town trigger:

**HTTP Trigger** (`*.http.ts`):
```typescript
export default async function(req: Request) {
  const run = await workflow.createRun();
  const result = await run.start({ inputData: { /* ... */ } });
  return Response.json(result);
}
```

**Cron Trigger** (`*.cron.ts`):
```typescript
export default async function(interval: Interval) {
  const run = await workflow.createRun();
  await run.start({ inputData: { /* ... */ } });
}
```

**Email Trigger** (`*.email.ts`):
```typescript
export default async function(email: Email) {
  const run = await workflow.createRun();
  await run.start({
    inputData: {
      from: email.from,
      subject: email.subject
    }
  });
}
```

### Storage Constraints

- SQLite is the only available database in Val Town
- No blob storage in Phase 1 (use JSON serialization)
- Tables use `wrkflw_` prefix to avoid naming conflicts
- Indexes on `workflow_id` and `status` for performance

## Visualization

### Mermaid Diagrams

Generate workflow structure diagrams:
```typescript
const diagram = workflow.visualize("mermaid");
console.log(diagram); // Paste into GitHub, mermaid.live, etc.
```

Generate execution state diagrams:
```typescript
const run = await workflow.createRun();
await run.start({ inputData: { /* ... */ } });
const executionDiagram = await run.visualize("mermaid");
```

### Canvas Visualizer

The frontend provides an interactive React Flow canvas:
1. Start API server: `deno run --allow-all examples/api-server-example-local.ts`
2. Start frontend: `cd frontend && npm run dev`
3. Open http://localhost:3000
4. Select workflows, create runs, watch execution in real-time

## Important Patterns

### Error Handling

Errors bubble up from steps to the engine:
- Step execution wrapped in try/catch
- Errors stored in `StepResult` with `status: 'failed'`
- Workflow snapshot saved with error details
- Error propagates to caller (no automatic retries in Phase 1)

### Initialization

Workflows auto-initialize on first run:
```typescript
const run = await workflow.createRun(); // Calls init() internally
```

This creates database tables if they don't exist. For explicit control:
```typescript
await workflow.init();
```

### Accessing Step Results

Within a step's execute function:
```typescript
execute: async ({ getStepResult, getInitData }) => {
  // Get output from a specific previous step
  const userData = getStepResult(fetchUser);

  // Get the workflow's initial input
  const initialInput = getInitData();

  return { /* ... */ };
}
```

### Workflow State

Define workflow-level state that persists across steps:
```typescript
const workflow = createWorkflow({
  id: 'stateful-workflow',
  inputSchema: z.object({ /* ... */ }),
  outputSchema: z.object({ /* ... */ }),
  stateSchema: z.object({ counter: z.number() }) // Workflow state
});

const step = createStep({
  execute: async ({ state, setState }) => {
    const count = (state?.counter || 0) + 1;
    setState({ counter: count });
    return { /* ... */ };
  }
});
```

## Future Phases

The roadmap is defined in PLAN.md:
- **Phase 2**: Parallel execution, branching, data transformations
- **Phase 3**: Durability (sleep, suspend/resume)
- **Phase 4**: Advanced features (foreach, nested workflows, retries)
- **Phase 5**: Observability (streaming, debugging UI, time-travel)

When implementing new features, follow the type-safe pattern established in Phase 1.

## Key Files to Understand

To quickly understand the codebase architecture:

1. **types.ts** - Core type definitions and interfaces
2. **workflow.ts** - Type inference through builder pattern
3. **engine.ts** - Sequential execution logic
4. **examples/simple-workflow.ts** - Complete end-to-end example

Reading these four files provides 80% of the architectural understanding needed to work effectively in this codebase.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nbbaier)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nbbaier)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
