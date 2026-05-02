## reac55-final-project

> When responding to user always start with .md files you have loaded (for example: used copiot-instructions.md and general-css-skill.md)

# general

When responding to user always start with .md files you have loaded (for example: used copiot-instructions.md and general-css-skill.md)
Never ask me if you can edit files. You are allowed to edit any file, add, or delete files as needed to complete the plans. If you need to ask for clarification on a step, ask the question in the "Ask questions" section of the relevant plan file.
Never repeat logic for example: use type def for types like: "sm" | "md" | "lg", and use it everywhere instead of repeating the string literals.
When asked to do a specific task, never change, rename, or refactor any variable, function, or identifier names, or alter code structure, even if you believe it would improve clarity or quality. Only suggest that there are improvements that could be make, but do not implement them without explicit instructions to do so. Always follow the instructions as closely as possible, and only make changes that are explicitly requested. If you have suggestions for improvements, provide them in a separate message and ask for confirmation before implementing any changes.
You are never allowed to push code to the remote repository in git.

We are using Windows CMD syntax.

# planning

Create a plan file in a folder with a CONSECUTIVE_NUM-NAME.md (example: plan/001-add-theme-toggle.md. Next plans in the same theme will be 002-add-theme-toggle.md, 003-add-theme-toggle.md while new plans will have 001 again) file for each plan. Create the file, never present the content in the response. The file should contain the following sections:

- Steps, a list of incremental steps to complete the plan, each step should be a single sentence. When user gives steps try to seperate it to smaller steps if possible.
- Ask quesitions, a list of as many questions as you can think of with options and a suggested option with reason to ask the user to clarify the plan. Each option should have a description.
- When I answer the question I use "-> MY_ANSWER". If nothing is wrote I accept the suggested option.

When executing the plan steps reference yourself to the relavant plan files and use their context to execute the steps. If you need to ask the user for clarification on a step, ask the question in the "Ask questions" section of the plan file.

# CSS

use the skill css layer

# Coding styles:

- Event handlers should be named onEvent, where Event is the name of the event being handled. For example, a click event handler should be named onClick.
- constants should be named in all uppercase letters with words separated by underscores. For example, MAX_VALUE.
- boolean variables should be named with a prefix of is, has, or can. For example, isActive, hasPermission, canEdit.
- We do not use comments in our code. Instead, we use descriptive variable and function names to make the code self-explanatory. If you find yourself needing to write a comment, consider whether you can rename variables or functions to make the code clearer instead. Use comments only when absolutely necessary, such as to explain complex logic that cannot be easily conveyed through naming alone.
- When writing tests always try to run them yourself and fix yourself before asking for help. If you need to ask for help, provide the error message and any relevant code snippets to help others understand the issue.
- Always prefer the full writing style of arrow function with return {} instead of the concise style. For example, use () => { return x; } instead of () => x;

---
> Source: [ranzohar/reac55-final-project](https://github.com/ranzohar/reac55-final-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
