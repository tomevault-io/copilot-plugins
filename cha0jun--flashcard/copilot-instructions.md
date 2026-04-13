## flashcard

> Follow SOLID and KISS principles in ALL code written, modified, or refactored.

# Project Rules

## Coding Principles

Follow SOLID and KISS principles in ALL code written, modified, or refactored.

For detailed guidelines, examples, and the decision checklist, read `skills/solid-kiss/SKILL.md` before writing any code.

### Quick Reference

**SOLID:**
- **SRP**: One class/function = one reason to change
- **OCP**: Extend behavior without modifying existing code
- **LSP**: Subtypes must be substitutable for their base types
- **ISP**: Small, focused interfaces over large general ones
- **DIP**: Depend on abstractions, inject dependencies

**KISS:**
- Readability over cleverness
- Functions under 30 lines
- No premature abstraction — need two concrete uses before creating an interface
- No premature optimization — profile first
- Nesting three levels or fewer

**When SOLID and KISS conflict, KISS wins.** Don't add an interface for one implementation. Don't split a 10-line class into three just for SRP.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cha0jun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cha0jun)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
