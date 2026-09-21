## rules-for-treesitter

> Rules for AI (Cursor) to follow when working with Tree-sitter libraries and queries in RepoMap-Tool.

# 🌳 **TREE-SITTER WORKFLOW PROTOCOL**

## **Before Engaging with Tree-sitter Underlying Libraries or Troubleshooting:**
**ALWAYS provide step-by-step reasoning that:**
1. States what Tree-sitter related action you're about to take
2. Explains which rules from this document apply
3. Shows how you've considered those rules in your approach
4. Outlines your specific plan and next steps

---

## 🛠️ **TROUBLESHOOTING & DEVELOPMENT PROTOCOL FOR TREE-SITTER**

### **🚨 CRITICAL: Tree-sitter Source Code Management**
- **ALWAYS download Tree-sitter language parser source code** into the `tmp/` directory when:
    - Troubleshooting issues related to Tree-sitter bindings.
    - Developing or debugging Tree-sitter queries (`.scm` files).
    - Inspecting the grammar or scanner implementation of a language parser.
    - Rebuilding or verifying Tree-sitter language bindings locally.
    - Deeply investigating unexpected parsing behavior.
- **NEVER modify Tree-sitter source code directly within the `tmp/` directory for production use.** Changes should be upstreamed or managed as patches if absolutely necessary.
- **ALWAYS ensure the `tmp/` directory is respected by `.gitignore`** to prevent accidental commits of downloaded sources.
- **PREFERRED method for downloading:** Use `git clone` to retrieve the official Tree-sitter language repositories (e.g., `tree-sitter-python`, `tree-sitter-javascript`) into `tmp/`.

### **Workflow for Tree-sitter Source Download:**
1.  **Identify the problematic Tree-sitter parser**: Determine which language parser (e.g., `tree-sitter-python`) is causing issues or requires deeper inspection.
2.  **Navigate to the `tmp/` directory**: Ensure the current working directory is `tmp/`.
3.  **Clone the repository**: Execute `git clone <repository_url>` (e.g., `git clone https://github.com/tree-sitter/tree-sitter-python tmp/tree-sitter-python`).
4.  **Perform necessary inspection/troubleshooting**: Access the downloaded source code for debugging, query development, or analysis.
5.  **Remove source when done**: If the source code is no longer needed, remove the directory from `tmp/` to keep the workspace clean.

### **Purpose of `tmp/` for Tree-sitter:**
-   **Isolation**: Provides a dedicated, ignored space for external source code.
-   **Debugging**: Allows direct access to grammar files, `parser.c`, `scanner.c` for deep debugging.
-   **Development**: Facilitates local testing and refinement of `.scm` queries against the official grammar.
-   **Stability**: Helps stabilize Tree-sitter queries and bindings by providing a consistent reference point for investigation.

### **Tree-sitter Query & Grammar Development**
- **ALWAYS use `tests/scm_parsing_foundation/`** for focused Tree-sitter query and grammar development.
- **Each language will have a dedicated subfolder** (e.g., `tests/scm_parsing_foundation/python/`).
- **The `grammar.js` file for the respective language parser should be copied into its subfolder** within `tests/scm_parsing_foundation/` to serve as a reference and guide for query development.
- **This directory facilitates isolated testing and refinement** of `.scm` queries against a known grammar, ensuring stability and accuracy.
- **Do NOT modify the `grammar.js` files directly in `tests/scm_parsing_foundation/`** unless you are specifically working on grammar modifications and understand the implications. These are primarily for reference.

## 🔗 **RELATED REPOMAP-TOOL RULES**
- **FUNDAMENTAL: TREE-SITTER PARSING ARCHITECTURE**: Reinforces the primary use of Tree-sitter in this project. This rule ensures that if underlying Tree-sitter issues arise, a standardized troubleshooting process is followed.
- **🚨 CRITICAL: Absolute Path Standardization**: When cloning into `tmp/`, the resulting paths will naturally be absolute. This aligns with the project's requirement for all file paths to be absolute throughout the system, simplifying interaction with the downloaded sources.

---
> Source: [xynova/repomap-tool](https://github.com/xynova/repomap-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
