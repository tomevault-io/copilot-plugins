## base

> This document supplements the main `architecture.mdc` file by providing specific details about the technologies and implementation of the Llama Agent Creator.

# Llama Agent Creator: Technical Details Supplement

This document supplements the main `architecture.mdc` file by providing specific details about the technologies and implementation of the Llama Agent Creator.

## Application Stack

-   **Framework**: [Next.js](mdc:https:/nextjs.org)
-   **Language**: [TypeScript](mdc:https:/www.typescriptlang.org)
-   **UI Library**: [React](mdc:https:/react.dev) with [shadcn/ui](mdc:https:/ui.shadcn.com) components.
-   **Graph Visualization**: [`@xyflow/react`](mdc:https:/reactflow.dev) is used to build the interactive workflow editor in `AgentFlow.tsx`.
-   **State & Storage**:
    -   The visual graph's state (nodes and edges) is persisted to `localStorage`.
    -   Global chat state is managed via React Context in `src/components/ChatProvider.tsx`.
-   **Core Workflow Engine**: [`@llamaindex/workflow-core`](mdc:https:/www.npmjs.com/package/@llamaindex/workflow-core) provides the foundational classes and logic for creating and running workflows in the generated TypeScript code.
-   **LLM Integrations**: The application uses `@llamaindex/openai`, `@llamaindex/anthropic`, and `@llamaindex/google` to connect to various language models.
-   **Testing**: The project is set up with [Vitest](mdc:https:/vitest.dev) for testing.

## File-Specific Details

This section clarifies the roles of key files and directories not fully detailed in the high-level architecture diagram.

-   `app/api/`: This directory contains the backend serverless functions that the interactive runner (`RunView.tsx`) calls. For instance, when a `promptLLM` node is executed in the UI, it makes a POST request to `app/api/llm/call/route.ts` to get the model's response.

-   `src/components/nodes/`: This directory contains the React components for each type of node that can be dragged onto the canvas (e.g., `PromptLLMNode.tsx`, `DecisionNode.tsx`, `UserInputNode.tsx`). These components define the appearance and configuration options of each node in the editor.

-   `src/components/CompileView.tsx`: This component provides the user interface for triggering the TypeScript compilation and viewing/downloading the generated code.

-   `src/lib/llm-utils.ts`: Contains utility functions for configuring and preparing the LLM providers, abstracting some of the setup required for making LLM calls.

-   `src/lib/initial-graph.ts`: Defines the default set of nodes and edges that are loaded when a user first visits the application or clears the canvas.

---
> Source: [run-llama/flow-maker](https://github.com/run-llama/flow-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
