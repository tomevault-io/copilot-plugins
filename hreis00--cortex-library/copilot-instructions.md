## cortex-library

> This file is the authoritative guide for AI agents (GitHub Copilot, OpenAI Codex, Claude, and similar tools) working inside the **Cortex Library** repository.

# AGENTS.md

This file is the authoritative guide for AI agents (GitHub Copilot, OpenAI Codex, Claude, and similar tools) working inside the **Cortex Library** repository.

## What Is This Repository?

Cortex Library is a centralized, curated collection of reusable AI copilot configurations — agents, instructions, prompts, and skills — designed to be shared across many different repositories and technology stacks.

Every artifact in this library must be:

- **Immediately usable** — no placeholder text, no TODOs, no filler content.
- **Broadly applicable** — either technology-agnostic or clearly scoped to a specific language/framework in its name and frontmatter.
- **Opinionated with reasoning** — rules and patterns must reflect widely accepted best practices, not arbitrary preferences.

---

## Repository Layout

```
cortex-library/
├── AGENTS.md                                  ← you are here
├── README.md                                  ← human-facing overview
└── .github/
    ├── agents/                                ← custom agent mode definitions
    │   └── code-reviewer.agent.md
    ├── instructions/                          ← scoped coding rules (always-on)
    │   └── global-standards.instructions.md
    ├── prompts/                               ← parameterized, reusable prompts
    │   └── pr.prompt.md
    └── skills/                               ← on-demand domain knowledge
        └── git-conventional-commits/
            └── SKILL.md
```

---

## File Conventions

### Agents — `.github/agents/*.agent.md`

Custom agent modes that define a specific persona, toolset, and behavior.

| Field       | Requirement                                                                                                                                |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| Filename    | `<kebab-case-purpose>.agent.md`                                                                                                            |
| Frontmatter | `name` (string), `description` (string); optionally `model`, `tools` (array of tool names), `argument-hint` (usage hint shown to the user) |
| Body        | Purpose, Capabilities, Behavioral Instructions (concrete, not vague), Example interaction                                                  |

> Do **not** write instructions like "be helpful" or "be thorough". Write rules that constrain and direct specific behaviors.

### Instructions — `.github/instructions/*.instructions.md`

Always-on rules that are injected into every interaction matching the `applyTo` glob.

| Field       | Requirement                                                                                   |
| ----------- | --------------------------------------------------------------------------------------------- |
| Filename    | `<scope>.instructions.md` (e.g., `global-standards`, `python`, `react`)                       |
| Frontmatter | `applyTo` (string glob or array of globs); optionally `name` (string), `description` (string) |
| Body        | Actionable rules only — no suggestions, no template placeholders. Rules must be enforceable.  |

### Prompts — `.github/prompts/*.prompt.md`

Parameterized slash-command prompts for focused, repeatable tasks.

| Field       | Requirement                                                                                                                                          |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Filename    | `<task>.prompt.md` (e.g., `pr`, `debug-session`, `generate-tests`)                                                                                   |
| Frontmatter | `name`, `description`; optionally `model`, `tools` (array), `argument-hint`, and `mode` / `agent` to set the prompt mode (`ask`, `edit`, or `agent`) |
| Body        | Actual prompt text (fully written), `## Variables` section defining each `{{placeholder}}`, concrete `## Example`                                    |

### Skills — `.github/skills/<name>/SKILL.md`

On-demand domain knowledge packages, invoked explicitly by filename reference.

| Field       | Requirement                                                                                                               |
| ----------- | ------------------------------------------------------------------------------------------------------------------------- |
| Directory   | `skills/<kebab-case-name>/`                                                                                               |
| Frontmatter | `name`, `description`; optionally `argument-hint` (usage hint shown to the user)                                          |
| Body        | Core Concepts, Best Practices with rationale, Patterns with real code examples, Anti-Patterns, Tools/Commands, References |

> Code examples in skills must use a real language — not pseudocode.

---

## Quality Rules

When an AI agent adds or modifies any file in this repository, it must enforce:

1. **No placeholder content.** Every file must be fully written and immediately usable.
2. **Valid YAML frontmatter.** All keys must be correct for the file type. Validate before writing.
3. **File naming conventions.** See per-type rules above. Inconsistent naming breaks discoverability.
4. **No duplication.** Search existing files before creating new ones.
5. **Real examples.** Every agent, prompt, and skill must contain at least one concrete, worked example.
6. **No sub-directory READMEs that duplicate this file.** If a sub-directory README exists, it must add context not available here.

---

## How to Add New Content

1. Choose the correct category based on the decision criteria below.
2. Confirm no existing file already covers the same purpose.
3. Follow the naming and frontmatter conventions for the category.
4. Write complete content — no TODOs, no "replace this with" placeholders.
5. If the new file introduces a new category or changes a convention, update this `AGENTS.md` accordingly.

### Choosing the Right Category

| I need to...                                                 | Use             |
| ------------------------------------------------------------ | --------------- |
| Apply permanent rules to all files of a type                 | `instructions/` |
| Create a specialized agent mode with specific tools/behavior | `agents/`       |
| Package a repeatable task with variable inputs               | `prompts/`      |
| Bundle domain knowledge for on-demand reference              | `skills/`       |

**Instructions vs. Skill?** If the content should always apply → Instructions. If it's invoked for a specific task → Skill.

**Prompt vs. Skill?** Single focused task with inputs → Prompt. Multi-step workflow with bundled assets → Skill.

---

## Working in This Repo

- Apply `.github/instructions/global-standards.instructions.md` to all code and content edits.
- Use `.github/agents/code-reviewer.agent.md` when reviewing pull requests.
- Do **not** create boilerplate, template stubs, or "example" files. Write the real thing.
- Do **not** modify `.git/` or any git internals.
- Prefer editing existing files over creating new ones when expanding coverage of an existing topic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hreis00)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hreis00)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
