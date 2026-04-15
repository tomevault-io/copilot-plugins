## data-processing-at-scale

> Standards for writing student lab handouts — tone, setup, dependencies, and structure


# Lab Writing

## Tone and register

Students are engineers. Do not:
- Add "verify the install" steps with expected output
- Explain how to run `ls` or what a missing file means
- Add "if X fails, do Y" fallback prose for trivial operations

Do:
- Give the command, trust they can run it
- Use fallback notes only for genuinely fragile steps (e.g., external URLs that may change)

## Setup section

Keep setup short. One subsection per concern (install, dataset, prep script).  
Do not list every package with its purpose — just the install command.

Dependencies must be justified by the exercises that use them. If only one bonus exercise needs a package, mark it optional at that exercise, not in the main install block.

## Fill-in blanks and tables

Do not add fill-in placeholder tables (`\_\_\_\_ MB`) unless the user explicitly asks for a worksheet format.  
Replace with prose: "Compare the sizes of the four files" or "Note the `Bytes Read` value in each plan."

## Predictions

"Before running: predict X" is a valid pedagogical device — keep it as prose, not a blank to fill in.

## Exercise structure

- Task first (the code or command to run), then questions.
- Questions should require reasoning, not just reading a number off the screen.
- Mark bonus exercises clearly with `(bonus)` in the heading.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vbergeron)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vbergeron)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
