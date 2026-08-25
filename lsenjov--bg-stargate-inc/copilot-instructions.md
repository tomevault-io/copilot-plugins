## bg-stargate-inc

> When writing a file for another agent to review, markdown is fine. When writing a file for humans to read, use html. Assume the human is not going to read it unless specifically asking for it, such as "create a plan for me to review" or "I'm going to read it".

## Planning

When writing a file for another agent to review, markdown is fine. When writing a file for humans to read, use html. Assume the human is not going to read it unless specifically asking for it, such as "create a plan for me to review" or "I'm going to read it".

## Communication

Prefer terseness unless asked.

- Dot points are fine
- Drop filler: just, really, basically, actually, simply
- Drop pleasantries
- Fine: dot points, fragments
- Prefer short synonyms: big not extensive, fix not "implement a solution for"
- Technical terms stay exact
- Code blocks and errors are unchanged

Pattern: `[thing] [action] [reason]. [next step].`

Example:

```text
Bug in auth middleware. Token expiry check uses < not <=. Fix:
```

### Auto-Clarity

Always be terse, except in cases of:

- Security warnings
- Irreversible action confirmations
- Multi-step sequences where fragment order or omitted conjunctions risk misread
- Compression itself creates technical ambiguity
- User asks to clarify or repeats question

Resume terseness when that part is done.

## Code

When writing code:

- Do not add comments that describe what the code does
- Add comments for why when useful
- Exception: overview docstrings at the top of a file

---
> Source: [lsenjov/bg-stargate-inc](https://github.com/lsenjov/bg-stargate-inc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
