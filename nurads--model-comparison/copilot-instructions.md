## model-comparison

> Master workflow for AI model comparison evaluation (Model A vs Model B)


# Model Comparison Workflow

You are assisting with an AI model comparison evaluation. Two models (A and B) respond to the same prompt, and responses are evaluated across multiple axes.

## Working Directories

- Model A: See `model-a-working-dir.txt` for the current path
- Model B: See `model-b-working-dir.txt` for the current path

## Workflow Steps

1. **Write Prompt** → Automatically read `issue.txt`, rewrite it into a clean prompt, save to `Prompt.md`. Uses `@prompt-writing` rule. No manual editing needed.
2. **Send to Both Models** → Send the content of `Prompt.md` to both VS Code instances (Model A and Model B).
3. **Evaluate Responses** → Use `@evaluation-guide` rule. Fill in `evaluations/round-N.md` using the template.
4. **Score Each Axis** → Use `@evaluation-guide` for allowed values and format.
5. **Write Pros/Cons** → Free-text paragraphs, no code references, no comparison language.
6. **Generate Follow-Up** → Use `@follow-up-questions` rule. Derive from the selected model's cons.
7. **Validate** → Use `@validation-checklist` rule before submitting.

## Key Constraints

- **Interface design** axis is **always N/A** — never evaluate it.
- Every evaluation must have both pros AND cons for each model — zero perfect responses.
- Never explain how code works, never quote code, never use vague praise.
- All evaluations use paragraph form, describe qualities and outcomes.
- Follow-ups are PR review comments, not scope expansions.
- Ask the model to **commit all final changes** before ending interaction.

## Quick Commands

- `evaluate` → Start a new evaluation round
- `write-prompt` → Draft/rewrite a prompt
- `follow-up` → Generate a follow-up question for the selected model
- `validate` → Run the full validation checklist
- `compare` → Side-by-side diff of model outputs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nurads) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
