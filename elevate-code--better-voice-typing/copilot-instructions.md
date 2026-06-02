## better-voice-typing

> This project is a speech-to-text Python desktop app using tkinter, pynput, OpenAI Whisper API, and system tray integration for Windows

This project is a speech-to-text Python desktop app using tkinter, pynput, OpenAI Whisper API, and system tray integration for Windows

Avoid blocking operations (e.g., time.sleep) - use async/event-driven patterns where possible

"Always add type hints to function parameters and return values using Python's type annotation syntax (e.g., `def function(param: type) -> return_type`)."

When responding with code edits, include any comments that are nearby the edited code and update them if needed.
If the user has a "NOTE:" or "IMPORTANT:" comment nearby, always include it in your code edit response.
Example: if user's code has `#NOTE: Business logic\n\n\ndef calc():...`, include it as `// ... #NOTE: Business logic {{ updated_code }} // ...`

When asked to do a database migration, write a migration function that can be run in the `main` block unless otherwise specified.

Use functional programming over object-oriented programming paradigms, where applicable.

IMPORTANT: When asked to write a commit message, **always** make sure increment the version.txt file and update the "## Changelog" section in README.md or remind the user to do so, adding an emoji prefix for visibility.

---
> Source: [Elevate-Code/better-voice-typing](https://github.com/Elevate-Code/better-voice-typing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
