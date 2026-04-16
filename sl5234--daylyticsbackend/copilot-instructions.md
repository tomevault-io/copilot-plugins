## daylyticsbackend

> Before making any code changes, you MUST:

# Code Change Workflow Rules

Before making any code changes, you MUST:

1. **Read Documentation First:**
   - Read `DESIGNS.md` to understand architecture and design decisions
   - Read `AGENTS.md` to understand codebase best practices and anti-patterns
   - Read `README.md` to understand codebase guide and setup

2. **Check for Deviations:**
   - Before implementing changes, verify they align with:
     - Architecture patterns in DESIGNS.md
     - Best practices in AGENTS.md
     - Guide and setup in README.md

3. **If Deviation Detected:**
   - Reconsider the change: Is the change necessary?
   - If deviation is intentional and justified:
     - Update the relevant MD file(s) to reflect the change
     - Document why the deviation was necessary

After making any code changes, you MUST:

1. **Read Documentation First:**
   - Read `DESIGNS.md` to understand architecture and design decisions
   - Read `AGENTS.md` to understand codebase best practices and anti-patterns
   - Read `README.md` to understand codebase guide and setup

2. **Check for Deviations:**
   - Before implementing changes, verify they align with:
     - Architecture patterns in DESIGNS.md
     - Best practices in AGENTS.md
     - Guide and setup in README.md

3. **If Deviation Detected:**
   - Verify the change still aligns with documentation
   - Update documentation if architecture/patterns changed

5. **Ensure build and run suceeds:**
   - Read `README.md` `How to build and run` section.
   - Run steps of that section.


## Key Principles to Follow:

- **Architecture**: Follow layered architecture (Routes → Services → External APIs)
- **Python Style**: Follow AGENTS.md anti-patterns and best practices
  - **CRITICAL**: Remove all trailing whitespaces from code files
  - **CRITICAL**: No unnecessary indentation after new lines
- **Configuration**: Use environment variables via `app/config.py`
- **Type Safety**: Use Pydantic models and type hints everywhere
- **Service Separation**: One responsibility per service

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sl5234) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
