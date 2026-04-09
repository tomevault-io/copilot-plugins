## litigation-support

> **Objective:** This document provides the official guidelines for creating and using standardized Cascade Workflows within this project. It ensures all workflows are consistent, discoverable, and follow platform best practices.


# Workflow Rules and Best Practices

**Objective:** This document provides the official guidelines for creating and using standardized Cascade Workflows within this project. It ensures all workflows are consistent, discoverable, and follow platform best practices.

---

## 1. How Workflows Work

Workflows are structured Markdown files that guide Cascade through a series of predefined steps to accomplish a multi-step task. They are a powerful tool for automating repetitive processes and ensuring consistency.

### a. Location

- All workflow files **MUST** be placed in the `.windsurf/workflows/` directory.
- Files **MUST** be named descriptively using `kebab-case` (e.g., `create-backend-package.md`).

### b. Invocation

- Workflows are executed by typing a forward slash (`/`) followed by the filename (without the `.md` extension) in the chat.
- **Example:** To run `create-backend-package.md`, you would type:
  ```
  /create-backend-package
  ```

---

## 2. Workflow File Structure

Workflows are standard Markdown files with two main components: a YAML frontmatter for metadata and the Markdown body for the steps.

### a. YAML Frontmatter

The frontmatter provides essential metadata about the workflow. It **MUST** include:

- `description`: A brief, clear summary of what the workflow does.
- `author`: The person or entity who created the workflow.
- `version`: The version number of the workflow.
- `tags`: A list of relevant keywords for categorization.
- `globs`: A list of file patterns where the workflow is most relevant.

**Example Frontmatter:**
```yaml
---
description: Defines the step-by-step workflow for creating a new backend feature package.
author: Cascade
version: 1.0
tags: ["workflow", "backend", "architecture"]
globs: ["backend/app/**"]
---
```

### b. Markdown Body

The body of the file contains the step-by-step instructions for Cascade.

- Use standard Markdown for all content (headings, lists, code blocks).
- Organize steps logically with numbered headings (`## 1. Step Name`).
- Use numbered or bulleted lists for sub-steps.
- Use code blocks (```) to provide example commands or code snippets.
- Be explicit. Clearly state the actions to be taken and any tools to be used (e.g., `run_command`).

---

## 3. Example Workflow

Here is a condensed example of a well-structured workflow file.

```markdown
---
description: A workflow to set up a new Python project using uv.
author: User
version: 1.0
tags: ["python", "uv", "setup"]
globs: ["pyproject.toml"]
---

# Python Project Setup Workflow

**Objective:** To quickly initialize a new Python project, create a virtual environment, and add initial dependencies using `uv`.

## 1. Initialize the Project

1.  Run `uv init` to create the `pyproject.toml` file.
    ```bash
    uv init
    ```

## 2. Create Virtual Environment

1.  Create a virtual environment in the `.venv` directory.
    ```bash
    uv venv
    ```

## 3. Add Dependencies

1.  Add `fastapi` and `uvicorn` as dependencies.
    ```bash
    uv add fastapi uvicorn
    ```

## 4. Completion

1.  The project is now set up. You can activate the environment with `source .venv/bin/activate`.
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rjmoggach)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rjmoggach)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
