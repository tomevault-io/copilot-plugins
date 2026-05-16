## no-manual-work

> rule "no_manual_work" {

rule "no_manual_work" {
  description = """
  Always write the full implementation. Never ask the user to manually write, edit, comment out, or finish code themselves.
  """

  enforce = true

  applies_to = [
    "code_generation",
    "code_editing",
    "code_refactoring",
    "code_suggestions"
  ]

  behavior {
    when_prompted_for_code = "Generate complete, functional code with all required parts. Do not leave stubs, placeholders, or TODOs."
    when_editing = "Apply full changes without requiring user interaction or manual edits."
    on_partial_request = "Assume full implementation is desired unless explicitly instructed otherwise."
    on_error_or_uncertainty = "Proceed with best-effort implementation. Never defer to user."
    on_edge_cases = "Consider all potential edge cases and ensure the solution is robust and comprehensive."
  }

  forbidden_behaviors = [
    "Suggesting the user should comment something out",
    "Suggesting the user should finish a section of code",
    "Leaving incomplete code blocks or TODOs",
    "Asking the user to manually configure settings or imports",
    "Prompting the user to validate or complete changes manually",
    "Requesting the user to handle errors or exceptions",
    "Instructing the user to test or verify changes without providing automated tests"
  ]

  examples = [
    {
      input = "Add authentication to the app."
      expected_behavior = "Write all auth setup code, route protection, middleware, and client integration."
    },
    {
      input = "Make this component responsive."
      expected_behavior = "Update layout fully with responsive styles. No manual steps required."
    },
    {
      input = "Handle edge cases for user input."
      expected_behavior = "Implement comprehensive input validation and error handling."
    },
    {
      input = "Integrate a new API."
      expected_behavior = "Complete API integration with error handling, data parsing, and UI updates."
    }
  ]
}

---
> Source: [thinktidevibes/2D-Survival-Multiplayer-Game](https://github.com/thinktidevibes/2D-Survival-Multiplayer-Game) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
