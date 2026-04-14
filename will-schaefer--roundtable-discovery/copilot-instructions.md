## roundtable-discovery

> Roundtable Discovery is a Python library that generates high‑value discovery questions by fanning a prompt out to multiple models and consolidating the results. It is designed for integration into an existing CLI chat.

# Gemini AI Context & Guidelines

## 1. Project Overview
Roundtable Discovery is a Python library that generates high‑value discovery questions by fanning a prompt out to multiple models and consolidating the results. It is designed for integration into an existing CLI chat.

## 2. Tech Stack & Architecture
- **Languages:** Python
- **Frameworks:** None (library module)
- **Infrastructure:** Local execution (no service yet)

## 3. Universal Coding Standards
- **Consistency:** Strictly adhere to the existing code style (indentation, naming conventions).
- **Clarity:** Prefer readable, explicit code over concise but obscure "one-liners."
- **Typing:** Use Python type hints where practical.
- **Error Handling:** Catch errors specifically; avoid bare `except` or generic catch blocks.
- **Immutability:** Prefer immutable data structures where possible.

## 4. Testing Strategy
- **Requirement:** New features and bug fixes should have tests.
- **Style:** Use descriptive test names that explain the behavior being verified.
- **Scope:** Prioritize unit tests for logic.
- **Command:** `PYTHONPATH=src python -m pytest`

## 5. Documentation & Comments
- **Code Comments:** Focus on the *why*, not the *what*. Explain non-obvious logic.
- **Commit Messages:** Use conventional commits (e.g., `feat: add adapter`, `fix: scoring edge case`).
- **README:** Keep `README.md` updated with setup and run instructions.

## 6. Coordination Workflow (required)
- Read `docs/coordination/workflow/START_HERE.md` before starting.
- Use `docs/coordination/workflow/HANDOFF_TEMPLATE.md` and write your update in the current round folder.
- Add one creative idea to `docs/coordination/Idea_Storm_Room.md` for each round.
- Update `docs/coordination/DECISIONS.md`, `RISKS.md`, and `OPEN_QUESTIONS.md` if needed.
- Run `python scripts/aggregate_handoffs.py` after updating handoffs.
- Update `docs/coordination/NEXT_ACTIONS.md` with the next agent’s priorities.
- For non-trivial tasks, run parallel agents and consolidate results into a single handoff.
- Do not offer optional "if you want" steps; follow the workflow as written.

## 7. AI Agent Workflow
- **Planning:** Propose a plan for complex changes before implementation.
- **Safety:** Do not modify configuration files or delete code without explicit confirmation or clear reasoning.
- **Verification:** Run tests after significant changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/will-schaefer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/will-schaefer)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
