## iitgn-aug-26-lab

> Welcome to the **IIT Gandhinagar Undergrad AI Harness Lab** repository.

# AGENTS.md — IIT Gandhinagar AI Harness Lab

Welcome to the **IIT Gandhinagar Undergrad AI Harness Lab** repository.

This repository serves as a pedagogical and experimental testbed designed to teach undergraduate students at IIT Gandhinagar how to design, architect, and implement their own **AI agent harnesses** from first principles.

---

## 🎯 Lab Objectives & Philosophy

1. **First-Principles Understanding**: Demystify how agent harnesses work under the hood—including model adapters, context window management, tool calling protocols, structured outputs, memory systems, subagent orchestration, and eval harnesses.
2. **Pedagogical Clarity**: Code must be modular, well-typed, thoroughly documented, and easy for students to study, modify, and extend.
3. **Reproducibility & Rigor**: Every module should come with clear unit tests, end-to-end integration tests, and reproducible benchmarks.

---

## 🏗️ Workflow & Working Conventions

To maintain high engineering standards and support structured learning, all work in this repository follows two core workflows:

```
.agents/
├── plans/               # Detailed blueprints for larger features & milestones
│   └── <feature-plan>.md
└── skills/              # Repeatable workflows, runbooks, and automation scripts
    └── <skill-name>/
        ├── SKILL.md     # Instructions and metadata for the skill
        └── scripts/     # Executable helper scripts (Python/Bash/Node)
```

---

## 📋 1. Planning for Large Pieces of Work (`.agents/plans/`)

Whenever tackling a non-trivial feature, architectural overhaul, lab assignment creation, or milestone implementation, **create a plan document first** in `.agents/plans/`.

### When to Create a Plan
- Introducing new core harness abstractions (e.g., dynamic context compaction, streaming tool execution, multi-agent communication).
- Adding support for new LLM backends / providers.
- Designing new lab assignments or student exercises.
- Large-scale refactors or infrastructure additions (e.g., benchmarking harness, tracing/observability).

### Plan Document Standard Structure
Create a markdown file at `.agents/plans/<plan-name>.md` containing:
- **Title & Overview**: Problem statement and high-level goal.
- **Learning Objectives**: Concepts students should learn from this component.
- **Architectural Design**: Module boundaries, interfaces/data models, state machines, and sequence flow.
- **Implementation Steps**: Phased, bite-sized tasks with clear definitions of done.
- **Verification & Testing**: Unit tests, mock evaluations, and edge cases to test.
- **Student Exercises & Extensions**: Suggested open-ended challenges or assignments for students.

---

## 🛠️ 2. Reusable Skills for Repeatable Tasks (`.agents/skills/`)

For repeatable workflows, runbooks, and automated development procedures, encapsulate them into **skills** within `.agents/skills/<skill-name>/`.

### When to Create a Skill
- Scaffolding a new harness module, lab exercise, or tool definition.
- Running standard benchmark suites across local and remote models.
- Generating synthetic datasets or prompt evaluations for test suites.
- Validating student submissions against test harnesses.
- Automating repository maintenance, linting, type-checking, or documentation generation.

### Skill Folder Structure
Each skill should be structured as follows:
```
.agents/skills/<skill-name>/
├── SKILL.md             # Required: YAML frontmatter + runbook instructions
└── scripts/             # Optional: executable helper scripts
    ├── run.py
    └── ...
```

### `SKILL.md` Format
```markdown
---
name: <skill-name>
description: <Short 1-2 sentence description of what the skill does and when to use it>
---

# <Skill Title>

## Purpose
Explain when and why this skill should be executed.

## Prerequisites
Dependencies, environment variables, or CLI tools required.

## Step-by-Step Instructions
Concrete instructions and commands the agent/developer should execute.

## Helper Scripts
Documentation for any companion scripts located in `scripts/`.
```

---

## 🎓 3. Guidelines for Adding New Demo Steps (`XX-<step-name>/`)

When creating a new incremental lab step (e.g. `03-tool-definition/`, `04-tool-execution/`):

### 1. Absolute Minimum & Concise Code
- **Classroom Demo First**: These scripts are live teaching tools for undergraduate students learning to code. The code should be readable from top to bottom in under 1 minute.
- **No Over-Engineering**: Avoid defensive corner-case handling, retry loops, custom HTTP header gymnastics, complex class hierarchies, or multi-flag argument parsers.
- **Pass Arguments Directly**: Read inputs from simple CLI arguments (`sys.argv`) or interactive `input()` prompts.

### 2. Standard Step Structure
Each step directory must contain:
```
XX-<step-name>/
├── main.py        # The minimal, self-contained demonstration script (~30-50 lines)
└── README.md      # Concise learning objectives, core concept breakdown, and run commands
```
- **No Per-Step Dependency/Env Files**: Do **not** create per-step `requirements.txt` or `.env` files. All dependencies remain in the root `requirements.txt`, and API keys are read from the shell environment.

### 3. Reuse `common` Library
Always import shared utilities from the root [`common/`](file:///Users/championswimmer/Development/Teaching/iitgn-ai-harness/common/__init__.py) package:
- `get_client()`: Pre-configured OpenRouter OpenAI client.
- `DEFAULT_MODEL`: Standardized free model selector.
- `loading`: Console loading animation (`with loading("Thinking"): ...`).

### 4. Running from Repository Root
All steps should be executed from the top-level repository root:
```bash
python XX-<step-name>/main.py [optional_args]
```

### 5. Type-Checker Compatibility
Keep type annotations clean and Pyright-compliant (e.g. annotate `messages: list[ChatCompletionMessageParam]` when using chat message lists).

---

## 🧑‍💻 Coding & Contribution Guidelines for AI Agents

When assisting students and instructors in this repository:

1. **Keep Demos Minimal & Pedagogical**: Prioritize readability and core concept demonstration over production defensive boilerplate.
2. **Reuse Shared Helpers**: Always leverage `common` for auth and client creation across all step folders.
3. **Follow Link Conventions**: When referencing files or symbols, use markdown links (e.g., [`common/__init__.py`](file:///Users/championswimmer/Development/Teaching/iitgn-ai-harness/common/__init__.py)).

---
> Source: [championswimmer/iitgn-aug-26-lab](https://github.com/championswimmer/iitgn-aug-26-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
