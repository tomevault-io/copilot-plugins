## ail-updated

> This project is a VS Code extension for the "Architectural Intelligence Ledger" (AIL). Its goal is to analyze codebases and generate a queryable knowledge graph.

## Current Project State

This project is a VS Code extension for the "Architectural Intelligence Ledger" (AIL). Its goal is to analyze codebases and generate a queryable knowledge graph.

**Layer 1 (Repository Ingestion & Language Detection) is fully implemented.** The code is located in `src/layer1/` and consists of an `orchestrator.ts` file that runs 8 sequential checkpoint modules found in `src/layer1/checkpoints/`. The orchestrator manages the data flow between checkpoints and creates the final analysis manifest in the `.ail/` directory.

## Project Overview

This is a Visual Studio Code extension named "AIL-extension" built with TypeScript. Based on project documentation, AIL stands for "Architectural Intelligence Ledger". The extension's purpose is to analyze a software project's codebase to create a queryable knowledge graph, helping developers understand the architecture and history of the code.

The extension activates when a workspace is opened and provides a command to trigger an analysis. The analysis results are intended to be displayed in a custom webview panel within the editor.

**Key Technologies:**
*   TypeScript
*   VS Code Extension API
*   esbuild for bundling
*   ESLint for linting
*   Mocha for testing

## Architecture & Implementation

### Layer 1: Repository Ingestion & Language Detection (Implemented)
*   **Location:** `src/layer1/`
*   **Orchestration:** `orchestrator.ts` serves as the entry point for this layer. It creates the `.ail/layer1/analysis` directory structure and runs all checkpoints in sequence, passing results from one to the next.
*   **Implemented Checkpoints:** The core logic is broken down into 8 distinct, modular checkpoints located in `src/layer1/checkpoints/`:
    1.  `cp1_workspace.ts`: Gets the root path of the open VS Code workspace.
    2.  `cp2_filescanner.ts`: Scans all files in the workspace, ignoring directories like `node_modules` and `.git`.
    3.  `cp3_language_detector.ts`: Counts files by extension to determine language distribution.
    4.  `cp4_framework_scanner.ts`: Detects frameworks by analyzing manifest files (`package.json`, `requirements.txt`, etc.).
    5.  `cp5_entrypoint_finder.ts`: Heuristically finds potential application entry points (`main.py`, `index.js`, etc.).
    6.  `cp6_metrics.ts`: Calculates basic metrics like total lines of code (LOC).
    7.  `cp7_assemble_manifest.ts`: Combines the outputs of all previous checkpoints into a final `metadata.json` manifest file.
    8.  `cp8_notify.ts`: Shows a VS Code notification to the user upon successful completion of the analysis.

### Layer 2: AST-Based Static Analysis Engine (Planned)
This layer is responsible for deep analysis of the source code to build a structural graph of the entire repository. It moves beyond file names and dependencies to understand the code's internal logic.

#### **Core Concept: The Two-Pass System**
The analysis is performed in two main passes to accurately map cross-file relationships.

*   **Pass 1: Entity Extraction (The Inventory).** The engine iterates through every source file identified in Layer 1. For each file, it uses the **Tree-sitter** parser to generate an Abstract Syntax Tree (AST). It then "walks" this tree to find all definable entities.
    *   **Action:** When a function, class, or API endpoint is found, its official "blueprint" object is created and added to the **final `nodes` list**.
    *   **Simultaneously, it creates temporary "lookup" data structures:**
        1.  **Entity Registry:** A simple dictionary (`name` -> `file_path`) for fast lookups.
        2.  **Raw Calls List:** A list of every function call encountered, noting the caller and the name of the callee.
        3.  **Raw Imports List:** A list of every import statement.

*   **Pass 2: Relationship Resolution (The Connection).** After the first pass, the `nodes` list is complete, but the connections are unresolved. The engine now iterates through the temporary "raw" lists.
    *   **Action:** For each raw function call, it uses the Entity Registry to find the file location of the callee. With this information, it can now create the official "edge" object, linking the unique IDs of the caller and callee functions.
    *   **Result:** This process builds the final `edges` list, containing all `CALLS`, `IMPORTS`, and `INHERITS` relationships.

#### **Technical Architecture**
The analysis will be handled by a Python backend service, called by the VS Code extension.

*   **Flow:** `VS Code Extension (TypeScript)` -> `HTTP Request` -> `FastAPI Backend (Python)`
*   **Rationale:**
    *   Tree-sitter's Python bindings are mature and well-suited for complex AST manipulation.
    *   Python has excellent graph theory libraries like `NetworkX` for post-analysis (e.g., circular dependency detection).
    *   This keeps the resource-intensive parsing work out of the main VS Code extension thread, ensuring the editor remains responsive.

#### **Implementation Checkpoints**
1.  **Python FastAPI server:** Set up the basic web server skeleton.
2.  **Tree-sitter Python parser:** Integrate Tree-sitter and its language grammars to parse a single file into an AST.
3.  **Entity extractor:** Implement the "First Pass" logic to walk an AST and build the `nodes` list and temporary registries.
4.  **Call graph builder:** Implement the "Second Pass" logic for resolving function calls.
5.  **Import graph builder:** Implement the "Second Pass" logic for resolving imports.
6.  **Circular dependency detector:** Use a library like `NetworkX` on the import graph to find cycles.
7.  **JSON assembler:** Combine the final `nodes` and `edges` lists into the target `graph.json` format.
8.  **VSCode integration:** Add logic to the TypeScript extension to call the FastAPI backend and save the resulting JSON.

#### **Frontend Visualization Concept: The View Switcher**
The final graph will contain different types of nodes and edges. The React frontend will feature a "View Switcher" with tabs/buttons to provide different architectural views by filtering the `edges` list:
*   **[Function Calls]:** Shows only `CALLS` edges.
*   **[Import Graph]:** Shows only `IMPORTS` edges.
*   **[Class Hierarchy]:** Shows `DEFINES` and `INHERITS` edges related to classes.

#### **Key Libraries**
*   **Python:** `fastapi`, `tree-sitter` (with language grammars), `networkx`, `pydantic`.
*   **VSCode (TypeScript):** `axios` or `node-fetch` for making HTTP requests.

#### **Hackathon Scope (What to Skip)**
To ensure focus, the following advanced features will be skipped for the initial implementation of Layer 2:
*   Database model/schema extraction.
*   Exception handler (`try/catch`) analysis.
*   Deep data-flow analysis.
*   Code churn metrics (requires Layer 3 Git history).

## Building and Running

### Building
*   **Compile for development:** `npm run compile`
*   **Package for production:** `npm run package`

### Running and Debugging
1.  Run `npm install` to install dependencies.
2.  Open the project in Visual Studio Code.
3.  Press `F5` to open a new Extension Development Host window.
4.  Open the Command Palette (`Cmd+Shift+P`) and run "Run AIL Analysis".

### Testing
*   **Run tests:** `npm test`

## Development Conventions

*   **Code Style:** Uses ESLint (`eslint.config.mjs`). Run with `npm run lint`.
*   **Type Checking:** Uses TypeScript's strict checks. Run with `npm run check-types`.
*   **Entry Point:** The main extension activation is in `src/extension.ts`, which calls the Layer 1 orchestrator.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RajRudra06)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RajRudra06)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
