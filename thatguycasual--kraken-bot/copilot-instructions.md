## kraken-bot

> - **Mandatory Review:** ALWAYS read ALL files in the `memory-bank/` directory at the start of every session or new task. This is critical due to session memory resets.

# .cursorrules: Kraken Trading Bot Project

## Project Intelligence & Guidelines

### 1. Memory Bank Usage

- **Mandatory Review:** ALWAYS read ALL files in the `memory-bank/` directory at the start of every session or new task. This is critical due to session memory resets.
- **Hierarchy:** Understand the flow: `projectbrief.md` -> `productContext.md`, `systemPatterns.md`, `techContext.md` -> `activeContext.md` -> `progress.md`.
- **Update Frequency:** Update relevant Memory Bank files (especially `activeContext.md` and `progress.md`) after significant changes, discoveries, or when requested via "update memory bank".

### 2. Coding & Development

- **Primary Language:** Python (unless otherwise decided and documented in `techContext.md`).
- **Dependency Management:** Use `requirements.txt` (or `pyproject.toml` if using Poetry/PDM - to be confirmed) for Python packages.
- **API Key Security:** NEVER hardcode API keys. Use environment variables (e.g., via `.env` files and `python-dotenv`) and ensure `.env` is in `.gitignore`.
- **Error Handling:** Implement robust error handling for API calls, network issues, and unexpected data.
- **Logging:** Comprehensive logging is crucial. Log key events, decisions, errors, and API interactions.
- **Modularity:** Strive for a modular design. Core components (API, strategy, orders, logging) should be distinct.
- **Configuration:** Externalize configuration (API endpoints, strategy parameters, etc.) into configuration files or environment variables.
- **Kraken API:** Be mindful of rate limits. Implement appropriate backoff/retry mechanisms if necessary.

### 3. Communication & Planning

- **Planner Mode:** When entering "Planner Mode" or using `/plan`:
    1. Read Memory Bank thoroughly.
    2. Analyze existing code (if any) and the request.
    3. Ask 4-6 clarifying questions BEFORE drafting a plan.
    4. Present a comprehensive plan for approval.
    5. Implement approved plan, updating on step completion.
- **Clarification:** If any instruction is unclear, or if multiple implementation paths exist, ask for clarification before proceeding.
- **Tool Usage:** Explain tool usage clearly before making a tool call.

### 4. Documentation (Memory Bank)

- `projectbrief.md`: High-level goals, scope. Should be relatively static.
- `productContext.md`: The "why" and user perspective. Update if problem understanding or UX goals evolve.
- `activeContext.md`: CURRENT focus, recent changes, next steps. Update frequently.
- `systemPatterns.md`: Architecture, key technical decisions, design patterns. Update as system design solidifies or changes.
- `techContext.md`: Technologies, setup, constraints. Update if new tools/libraries are adopted or constraints change.
- `progress.md`: What works, what's left, status, issues. Update regularly to reflect actual progress.

### 5. Future Learnings (To be populated)

- (Example) Preferred Kraken API library: `[Library Name]` because `[Reason]`.
- (Example) Common pitfalls with Kraken API endpoint `[EndpointName]`: `[Details]`.
- (Example) User preference for logging format: `[Format Details]`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Thatguycasual) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
