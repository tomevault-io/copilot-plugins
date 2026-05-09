## llamaindex

> This document explains the core concepts of the `@llamaindex/workflow-core` library for TypeScript, which is used by `typescript-compiler.ts` to generate executable agent scripts. The library uses a functional, event-driven paradigm.

# Understanding @llamaindex/workflow-core (TypeScript)

This document explains the core concepts of the `@llamaindex/workflow-core` library for TypeScript, which is used by `typescript-compiler.ts` to generate executable agent scripts. The library uses a functional, event-driven paradigm.

## Core Concepts

The workflow is built by defining events and then creating handlers that listen for those events. The execution flow is determined by which events are emitted from handlers.

1.  **Event-Driven Flow**: The entire workflow is orchestrated by events. A handler function executes when it receives an event of a type it's registered to handle. After executing its logic, the handler can emit a new event, which in turn triggers another handler.

2.  **Functional Approach**: Instead of class-based inheritance (like in Python), the TypeScript version uses factory functions to build the workflow.
    -   `workflowEvent<T>()`: Creates a new type of event.
    -   `createWorkflow()`: Creates a new workflow instance.
    -   `workflow.handle([...events], handlerFn)`: Registers a function `handlerFn` to be called when any of the specified `events` are detected.

3.  **Asynchronous by Design**: All handlers can be `async` and are expected to return `Promise`s, making them suitable for I/O-bound tasks like LLM calls and tool interactions.

## Key Components

### 1. `workflowEvent`
This function is used to define the different types of events that will flow through your system. They are the data carriers.

```typescript
import { workflowEvent } from "@llamaindex/workflow-core";

// Define an event that carries a string payload
const startEvent = workflowEvent<string>();

// Define an event that carries a structured object
interface ToolCall {
  id: string;
  name: string;
  args: any;
}
const toolCallEvent = workflowEvent<ToolCall>();
```

### 2. `createWorkflow` and `.handle`
`createWorkflow()` creates the main workflow object. The `.handle()` method is then used to register listeners for specific events. The return value of a handler function becomes a new event that is dispatched into the workflow.

```typescript
import { createWorkflow, workflowEvent } from "@llamaindex/workflow-core";

const workflow = createWorkflow();
const startEvent = workflowEvent<string>();
const stopEvent = workflowEvent<string>();

// Register a handler for 'startEvent'
workflow.handle([startEvent], async (event) => {
  console.log(`Received start event with: ${event.data}`);
  // The return value is automatically dispatched as a new event
  return stopEvent("Workflow finished!");
});

// Register a handler for 'stopEvent'
workflow.handle([stopEvent], (event) => {
  console.log(event.data); // "Workflow finished!"
});
```

### 3. `Context`
For more complex workflows, you can access the workflow's context within a handler using `getContext()`. This provides more advanced capabilities.

```typescript
import { getContext } from "@llamaindex/workflow-core";

workflow.handle([someEvent], (event) => {
    const { sendEvent, stream, signal } = getContext();

    // ...
});
```

-   **`sendEvent(event)`**: Manually dispatches an event. This is the key to creating "fan-out" patterns where one handler can trigger multiple downstream handlers in parallel.
-   **`stream`**: An async iterable representing the stream of all events in the workflow. It has powerful utility methods like `.filter()`, `.until()`, and `.toArray()` to manage complex flows, like waiting for the results of parallel tasks.
-   **`signal`**: An `AbortSignal` that can be used to manage cancellation of long-running async tasks if the workflow is aborted.

## Control Flow Patterns

The event-driven architecture enables flexible control flow.

-   **Branching**: A handler can implement conditional logic and return different event types. The workflow will then proceed down the path corresponding to the event that was returned.
-   **Parallel Execution (Fan-out)**: A handler can call `sendEvent()` multiple times to emit several events simultaneously, triggering multiple downstream handlers to run in parallel.
-   **Joining/Aggregation**: The `stream` object in the context is used to collect results from parallel tasks. You can use `stream.filter(...).until(...).toArray()` to wait for a specific number of events to complete before proceeding.
-   **Middleware**: The library supports middleware to add functionality like state management (`createStatefulMiddleware`) or validation (`withValidation`) to your workflows.

This functional and event-driven approach provides a powerful and flexible way to define complex agentic logic that can be compiled into a standalone, executable script. For more details, you can refer to the official documentation at [https://ts.llamaindex.ai/docs/workflows](mdc:https:/ts.llamaindex.ai/docs/workflows).

---
> Source: [run-llama/flow-maker](https://github.com/run-llama/flow-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
