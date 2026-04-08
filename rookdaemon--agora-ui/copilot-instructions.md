## agora-ui

> This repo is a browser-based chat interface and CLI bridge for Agora messaging.

# Agora UI
This repo is a browser-based chat interface and CLI bridge for Agora messaging.

# Way of working
* Always start coding tasks with a git pull.
* Keep increments small, legible, and valuable.
* Prefer simple designs; refactor first when needed.
* Use TDD (red/green/refactor) where practical.
* Apply the boy scout rule: leave code better than you found it.
* Abstract environment dependencies (file, process, time, env, transport) behind interfaces.
* Inject time into business logic—no raw `Date.now()` or `new Date()` in core logic.
* Treat CLI handlers, HTTP servers, workers, and subprocess launchers as thin process shells only.
* Put business logic in services behind interfaces; process shells should only parse input, call services, and map output/errors.
* Services must be unit-testable without spawning processes or opening ports (use injected runners/transports/adapters).
* Prefer service-level unit tests by default; keep real process/port tests minimal and explicitly integration-only.
* End completed tasks with pull, build, lint, test, commit, push. Push often.

# Versioning
* Before committing significant changes, update package version appropriately and ensure build/tests pass.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rookdaemon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rookdaemon)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
