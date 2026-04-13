## pydantic-crash-course

> This is a **crash course documentation repository**, not a codebase. The markdown files in `docs/` are educational content for a Pydantic tutorial video series.

# Claude Instructions

This is a **crash course documentation repository**, not a codebase. The markdown files in `docs/` are educational content for a Pydantic tutorial video series.

## What this is

- Written documentation that accompanies a YouTube crash course
- 8 chapters teaching Pydantic from basics to structured LLM output
- Target audience: Python developers who know basics but haven't used type hints or data models
- Focus: 80/20 principle (teach what's used 80% of the time)
- Prerequisite: Foundational Python knowledge (link to [Python for AI](https://python.datalumina.com) for those who need it)

## Writing style

Follow the style from [python.datalumina.com](https://python.datalumina.com):

**Do:**
- Short, direct sentences
- Real-world examples (API configs, users, orders)
- Isolated code blocks focused on one concept
- "Common mistakes" sections
- Clear navigation between chapters

**Don't:**
- Em dashes (use commas or periods instead)
- "Why X matters" or "Why X for Y" headings (AI pattern)
- "It's worth noting", "Let's dive in", "Here's the thing"
- Generic examples (cats, dogs, foo, bar)
- Long paragraphs

## Teaching principles

**Introduce before you use:**
- Don't use `Field()` before explaining what it does
- Don't show `@property` or `@classmethod` without context
- Don't reference function type hints before the function section
- When using `model_config`, explain it's a reserved name Pydantic looks for

**Examples must be realistic:**
- `age="unknown"` is realistic API garbage, `age="twenty-five"` is not
- Show errors people actually encounter, not contrived edge cases

**Don't teach bad practices:**
- Don't write custom validators for things Pydantic already handles (like email with `EmailStr`)
- When showing a pattern, ensure it's the recommended approach

**Beginner awareness:**
- Explain that plain classes with type hints don't get `__init__` (need `@dataclass` or `BaseModel`)
- Clarify that Pydantic handles mutable defaults safely (unlike regular Python classes)
- Note when something is Pydantic-specific vs standard Python

## Structure

```
README.md                     # Course overview (main entry point)
docs/
  01-introduction.md          # Problem and installation
  02-type-hints.md            # Type hints foundation
  03-your-first-model.md      # BaseModel basics
  04-validation-and-fields.md # Field constraints
  05-nested-models.md         # Complex data
  06-pydantic-settings.md     # Environment variables
  07-structured-llm-output.md # Structured output
  08-summary.md               # Key learnings and next steps
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/daveebbelaar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/daveebbelaar)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
