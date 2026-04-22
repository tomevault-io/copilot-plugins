## cursor-rules-example

> - Three-layer structure: `basic/` (must apply), `modules/` (apply as needed), `workflow/` (apply as needed)

# AI Collaboration Execution Rules

## Basic Requirements
- Three-layer structure: `basic/` (must apply), `modules/` (apply as needed), `workflow/` (apply as needed)
- Use rule format in every file: Basic Requirements, Mandatory Actions, Forbidden Behaviors, Example Code
- Prefer example-driven guidance: link to `.mdc` examples where helpful
- Enforce consistent architecture, terminology, and quality gates across all outputs
- Apply prompt engineering principles: clarity, directness, step-by-step reasoning (see [Prompt Clarity](mdc:basic/prompt-clarity.mdc), [AI Reasoning](mdc:basic/ai-reasoning.mdc))

## Communication Best Practices
- **Golden Rule**: If a human colleague would be confused by your explanation, rephrase it. Be clear and direct.
- **Be Specific**: Avoid vague statements like "ensure high performance" without measurable criteria
- **Ask When Unclear**: When requirements are ambiguous, ask targeted clarifying questions immediately
- **Skip Preambles**: Get straight to the point; avoid unnecessary introductions unless context is needed
- **Show Your Work**: For complex tasks, explain reasoning step-by-step before conclusions

## Mandatory Actions
1. Identify scenario → select relevant rules from `basic/`, `modules/`, and `workflow/`
2. Read example code/snippets → use as reference for style and patterns
3. Apply Mandatory and Forbidden items → do not bypass constraints
4. Validate output → typesafe, readable, testable, maintainable, secure, accessible
5. Resolve conflicts by priority → `basic` > `modules` > `workflow`; document exceptions
6. Record assumptions explicitly when context is missing; avoid fabricating APIs or data
7. For complex multi-step tasks: break down the problem, reason through each step, then provide solution
8. When handling user data/input: use clear separators (XML tags preferred) to distinguish from instructions
9. When uncertain about facts, APIs, or implementation details: admit uncertainty and suggest verification steps

## Forbidden Behaviors
- Mixing roles or ignoring priority; contradicting rule layers
- Vague guidance (e.g., “ensure high performance”) without measurable criteria
- Introducing unapproved dependencies, APIs, or architectures
- Omitting tests, types, or error handling in critical paths

## Example Code
- See [Project Basics](mdc:basic/basic.mdc) and [TypeScript Rules](mdc:basic/ts.mdc) as style references
- See [Services](mdc:modules/service.mdc) for API client patterns

## Change Management
- Place new rules in the correct layer; avoid cross-layer duplication
- Version rule files with dated entries; summarize changes at the top of the file
- Keep rule scope single-responsibility and concise

## Design Principles
- Single responsibility, clear boundaries, dependency direction (lower-level stable > higher-level variable)
- Composability, replaceability, minimal surface area
- Fail fast, explicit errors, idempotency and retry for safe operations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kryst4lskyxx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
