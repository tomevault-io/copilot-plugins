## snow-claude

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Project Context: SNow-Claude Skills Expansion**

Adding a new `snow-story` skill to the SNow-Claude framework. This skill acts as a User Story Analyzer & Solutions Architect that takes raw agile user stories (via chat or file), applies ServiceNow platform guidelines (`SYSTEM.md`), retrieves relevant context, and breaks the story down into concrete, implementation-ready ServiceNow artifacts (Client Scripts, Business Rules, Script Includes, etc.). The skill integrates cleanly, ensuring SNow-Claude preserves its default behavior when the skill is not explicitly invoked.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Core Runtime
- **LLM**: Claude AI (Anthropic)
- **Interface**: Claude.ai (Web), Claude Projects, Claude Desktop, or Claude API.
## Project Framework
- **Architecture**: Meta-prompting and context engineering system.
- **Implementation**: Composable markdown modules (.md files).
- **Tooling**: Zero-dependency system. No compilation or installation required.
## Target Platform (ServiceNow)
- **Engine**: Rhino JavaScript (ES5+) with some ES6 support in newer releases.
- **Releases**: Washington DC, Xanadu, Vancouver.
- **Key ServiceNow APIs**:
## Configuration & Management
- **Version Control**: Git / GitHub
- **Documentation**: Markdown-based system
- **Deployment**: Copy-paste or manual upload to Claude Projects
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## 1. Naming Standards
| Artifact | Pattern | Example |
|---|---|---|
| Scoped Account | `x_<vendor>_<app>` | `x_acme_itsm_ext` |
| Business Rule | `<Table> - <Action> <Description>` | `Incident - Before Set Priority` |
| Script Include | `PascalCase` class name | `ApprovalMatrixUtils` |
| Client Script | `<Table> - <Type> <Description>` | `Incident - onChange Category` |
| System Property | `x_<vendor>_<app>.<category>.<name>` | `x_acme_itsm_ext.ui.hide_button` |
## 2. JavaScript Standards (Rhino Engine)
- **Variables**: Use `var` only. `let` and `const` have inconsistent support in older ServiceNow versions (Rhino engine).
- **Comparison**: Always use strict equality (`===` and `!==`).
- **Structure**:
- **Formatting**: Always use semicolons and 2 or 4 space indentation.
## 3. Logging & Error Handling
- **Structured Logging**: Always include the source identifier in brackets.
- **Error Handling**: Use `try/catch` blocks for complex logic, integrations, and database operations.
## 4. Security & Performance
- **No Hardcoding**: Never hardcode `sys_id`, URLs, or credentials. Use System Properties or Aliases.
- **Data Access**: 
- **Queries**: 
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

## Design Philosophy
## Architectural Layers
### 1. The Core Persona (SYSTEM.md)
- Expert identity as a Senior ServiceNow Architect/Developer.
- Universal coding standards (logging, error handling, naming).
- Quality gate protocols (25-point mental checklist).
### 2. Behavioral Roles (prompts/)
- `architect.md`: Design-first approach, trade-off analysis.
- `developer.md`: High-efficiency code generation from specs.
- `reviewer.md`: Critical analysis against platform anti-patterns.
- `debugger.md`: Systematic root-cause analysis and platform diagnostics.
### 3. Knowledge Base (contexts/)
- `_base.md`: Essential platform APIs and scoping rules.
- Specialized modules (e.g., `business-rules.md`, `gliderecord.md`) for specific artifact types.
### 4. Specification Engine (specs/)
## Data Flow
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hawks6) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
