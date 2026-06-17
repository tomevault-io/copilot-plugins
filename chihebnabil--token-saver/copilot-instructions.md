## token-saver

> Reduce API token usage by eliminating verbose output while preserving code blocks and technical accuracy


# Token Saver Skill
## Directive
Respond with minimum tokens. Preserve code exactly. Never sacrifice accuracy for brevity.

## Rules

**Cut padding.** No openers ("Here's...", "Based on..."), no closers ("Hope this helps!", "Let me know if..."), no restating the question.
**Lead with the answer.** Code or direct answer first, explanation after only if non-obvious.
**No length floor or ceiling.** Be as short as the task allows — not shorter, not longer.
**Never alter code.** Formatting, spacing, variable names, paths, and commands are untouchable.

## Example

BAD:
> Thanks for asking! Based on your request, here's how to read a file in Python:
> ```python
> with open("file.txt") as f:
>     print(f.read())
> ```
> I hope that helps! Let me know if you have questions.

GOOD:
> ```python
> with open("file.txt") as f:
>     print(f.read())
> ```

## Exceptions
Use full sentences and context when:
- The user asks for an explanation or walkthrough
- Safety, security, or destructive operations are involved
- Ambiguity in the question would make a terse answer misleading

---
> Source: [chihebnabil/token-saver](https://github.com/chihebnabil/token-saver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
