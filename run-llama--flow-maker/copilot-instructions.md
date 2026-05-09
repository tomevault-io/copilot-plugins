## architecture

> This document outlines the core architecture of the Llama Agent Creator application, focusing on how the visual workflow graph is translated into executable logic.

---
description: 
globs: 
alwaysApply: true
---
# Llama Agent Creator Architecture

This document outlines the core architecture of the Llama Agent Creator application, focusing on how the visual workflow graph is translated into executable logic.

## Core Components and Data Flow

The application has a clear separation between the visual graph representation, an intermediate JSON format, and two distinct execution engines.

`Visual Graph (React Flow)` --> `workflow-compiler.ts` --> `WorkflowJson (Intermediate State)` --> `[RunView.tsx OR typescript-compiler.ts]`

1.  **Visual Graph Editor (`src/components/AgentFlow.tsx`)**:
    *   This is where the user builds the agent workflow using a drag-and-drop interface powered by `@xyflow/react`.
    *   The state of the graph (nodes and edges) is saved to `localStorage`.

2.  **Workflow Compiler (`src/lib/workflow-compiler.ts`)**:
    *   This is a critical module that acts as a bridge between the UI and the execution logic.
    *   It takes the raw nodes and edges from the React Flow state.
    *   It processes this graph structure and compiles it into a standardized intermediate representation, `WorkflowJson`. This JSON object describes the nodes, their connections (via an event-based system), and their configurations.

3.  **Execution Engines**:
    The `WorkflowJson` can be consumed by two different parts of the application to execute the workflow:

    *   **Interactive Runner (`src/components/RunView.tsx`)**:
        *   Provides an in-browser, step-by-step execution of the workflow.
        *   It loads the graph from `localStorage`, uses `workflow-compiler.ts` to get the `WorkflowJson`, and then walks through the nodes.
        *   It implements the logic for each node type (e.g., 'promptLLM', 'userInput', 'decision') directly, making API calls to the backend (`/api/...`) as needed.
        *   It visually highlights the currently active node on the graph and provides a chat interface for user interaction.
        *   This provides a way to debug and test the workflow interactively.

    *   **TypeScript Code Generator (`src/lib/typescript-compiler.ts`)**:
        *   Takes the same `WorkflowJson` and generates a standalone, executable TypeScript file.
        *   This generated script uses the `@llamaindex/workflow-core` library to define the workflow programmatically.
        *   The logic within this compiler *mirrors* the logic in `RunView.tsx`, but instead of executing nodes directly, it generates code that will execute them in a Node.js environment. For example, it generates LLM calls, tool definitions, and the event handling logic that connects the nodes.
        *   This allows the user to export their visually-designed agent as a portable script.

## Key Takeaway

When modifying or debugging, it's essential to understand this dual-execution model. Changes to the logic of a node type (e.g., how a 'decision' node works) often need to be implemented in **both** `RunView.tsx` (for interactive execution) and `typescript-compiler.ts` (for code generation) to ensure consistent behavior across both modes. The `workflow-compiler.ts` is central to ensuring both execution engines receive the same structured information from the UI. 

---
> Source: [run-llama/flow-maker](https://github.com/run-llama/flow-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
