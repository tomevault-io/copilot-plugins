## radioatelier

> Do not try to start a new dev server, assume it's already running.

Do not try to start a new dev server, assume it's already running.

Don't regenerate convex functions, assume that convex dev environment is already running.

Do not run the build command directly — it is run automatically when needed.

If there were any frontend changes as a result of your run – always run the lint command to make sure there aren't any code style problems.

Use bun as the package manager for the project.

Don't put comments in the code that just describe what the next code block does. Comments should explain why, not what. Only put comments in code when the reasoning behind the current implementation should be clarified to not create confusion.

Follow SOLID, DRY and KISS principles in your work. Don't create huge god-files. Soft cap for line count if a single file is 200, if a file grows bigger – consider possible refactors. The same rule should apply for huge functions: if a function grows too large – consider splitting it or moving into its own module if that function is complex enough. Soft line cap for a single function is 20 lines.

Delete unused or obsolete files when your changes make them irrelevant (refactors, feature removals, etc.), and revert files only when the change is yours or explicitly requested. If a git operation leaves you unsure about other agents' in-flight work, stop and coordinate instead of deleting.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ngranko) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
