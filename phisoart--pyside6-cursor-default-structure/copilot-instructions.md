## pyside6-cursor-default-structure

> This document defines the base-level, project-wide rules that apply to all activities within the `pyside6-cursor-default-structure` project. It supersedes format-specific guides like `md-rules.mdc` (for documentation) and `py-rules.mdc` (for Python code) and aims to establish a consistent development environment, documentation philosophy, and optimization principles.

# Project Base Rules

## 1. Overview

This document defines the base-level, project-wide rules that apply to all activities within the `pyside6-cursor-default-structure` project. It supersedes format-specific guides like `md-rules.mdc` (for documentation) and `py-rules.mdc` (for Python code) and aims to establish a consistent development environment, documentation philosophy, and optimization principles.

**Compliance with all rules in this document is mandatory for all project contributors (human and AI).**

---

## 2. Conda Environment Policy: `astromapper`

**This policy is non-negotiable.**

### 2.1. Unified Development Environment
- All development, testing, and execution **must** be performed within the Conda environment named `astromapper`.
- Working in any other environment is not permitted. This ensures that all contributors share the exact same dependencies and runtime conditions.

### 2.2. Procedure for Inactive Environment
- If your terminal does not recognize `conda` or other project-specific commands (e.g., `command not found`), it means the `astromapper` environment is not active.
- In this case, you **must repeatedly execute** the following command until it succeeds:
  ```bash
  source ~/.bash_profile
  ```

### 2.3. In-Environment Execution Principle
- If a piece of code does not work inside the `astromapper` environment, the problem is with the code, not the environment.
- Modifying the code to make it work perfectly in `astromapper` takes precedence over all other tasks. Do not attempt to alter the environment to fit the code.

---

## 3. Documentation Interconnectivity & Context Policy

Our documentation is a deeply integrated knowledge system, not a collection of disparate files.

### 3.1. Mandatory Pre-Development Review
- Before writing or modifying any file, all developers **must** review all relevant existing documents within the `/docs/` directory.
- This is to ensure a full understanding of the project's goals (**[PRD.md](../../docs/PRD.md)**), architecture (**[ARCHITECTURE.md](../../docs/ARCHITECTURE.md)**), and standards (**[CODING_GUIDE.md](../../docs/CODING_GUIDE.md)**), preventing redundant work and context mismatches.

### 3.2. Document Role and Relationship Specification
The following table specifies the role of each key document and its interdependencies. All contributors must understand and maintain this structure.

| File Path | Role & Responsibility | Key Relationships & Purpose |
| :--- | :--- | :--- |
| **[PROJECT_STRUCTURE.md](../../docs/PROJECT_STRUCTURE.md)** | **Master Index**: The top-level guide explaining the purpose of all directories and documents. | Provides links to **all** other documents, acting as the main entry point. |
| **[PRD.md](../../docs/PRD.md)** | **The "What"**: Defines product requirements, features, and business goals. | Delegates the "how" to **[ARCHITECTURE.md](../../docs/ARCHITECTURE.md)** (technical) and **[design/UI_SPEC.md](../../docs/design/UI_SPEC.md)** (design). |
| **[ARCHITECTURE.md](../../docs/ARCHITECTURE.md)** | **The "How" (Structure)**: Designs the system architecture, tech stack, and data flow. | Fulfills requirements from **[PRD.md](../../docs/PRD.md)**; its implementation must follow **[CODING_GUIDE.md](../../docs/CODING_GUIDE.md)**. |
| **[CODING_GUIDE.md](../../docs/CODING_GUIDE.md)** | **The "How" (Code)**: Defines coding conventions, naming rules, and best practices. | Tightly coupled with **[TESTING.md](../../docs/TESTING.md)** to ensure all code is high-quality and verifiable. |
| **[TESTING.md](../../docs/TESTING.md)** | **Quality Assurance**: Defines the project's testing strategy, coverage standards, and tools. | Validates the quality of code that was written according to the **[CODING_GUIDE.md](../../docs/CODING_GUIDE.md)**. |
| **[docs/design/UI_SPEC.md](../../docs/design/UI_SPEC.md)** | **UI Flow & Behavior**: Defines wireframes, user flows, and interaction logic. | Uses standardized building blocks from **[COMPONENTS.md](../../docs/design/COMPONENTS.md)** and visual rules from **[THEME.md](../../docs/design/THEME.md)**. |
| **[docs/design/COMPONENTS.md](../../docs/design/COMPONENTS.md)**| **UI Building Blocks**: Specification for every reusable UI component. | The fundamental parts used to construct the screens defined in **[UI_SPEC.md](../../docs/design/UI_SPEC.md)**. |
| **[docs/design/THEME.md](../../docs/design/THEME.md)** | **Visual Design System**: Defines design tokens like colors, fonts, and spacing. | Ensures a consistent visual identity across **[COMPONENTS.md](../../docs/design/COMPONENTS.md)** and **[UI_SPEC.md](../../docs/design/UI_SPEC.md)**. |

### 3.3. Project Structure Maintenance
- **Single Source of Truth**: The **[PROJECT_STRUCTURE.md](../../docs/PROJECT_STRUCTURE.md)** document is the definitive reference for the project's file and directory layout.
- **Mandatory Updates**: Any addition, deletion, or renaming of files or directories **must** be immediately reflected in this document.
- **Purpose**: This discipline ensures the master index remains accurate, preventing structural drift and keeping all team members aligned.

---

## 4. Optimization and Minimization Philosophy

This project operates on the principle of **"Less is More."**

### 4.1. The Responsibility of Continuous Optimization
- **Eliminate Redundancy**: Actively remove duplicate or unnecessary content from all assets, including code, documentation, and configuration.
- **Enhance Connectivity**: All files and modules should be optimized for easy navigation and cross-referencing. Make the logical connections between files explicit.
- Optimization is not a one-time task; it is a continuous responsibility for every contributor at every stage of the project.

### 4.2. Explicit Instruction for AI Agents

> **AI Tasking Rule**: When instructing an AI to generate or modify content for this repository, you **must** explicitly direct it to prioritize **"optimization"** and **"interconnectivity."** The AI must demonstrate an understanding of the existing file structure and document relationships and explain how its output fits harmoniously within this system.

---

## 5. References to Specific Rule Files

This document covers project-wide base rules. For format-specific rules, consult the dedicated guides below.

- **For Markdown Documentation**: Refer to **[.cursor/rules/md-rules.mdc](./md-rules.mdc)** for rules on style, formatting, and structure of all `.md` files.
- **For Python Code**: Refer to **[.cursor/rules/py-rules.mdc](./py-rules.mdc)** for rules on coding style, structure, and principles for all `.py` files. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phisoart) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
