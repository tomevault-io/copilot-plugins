## portal2

> * Keep code succinct.

# Code Guidelines
* Keep code succinct.
* Add validation both in the backend and frontend.
* Don't repeat yourself needlessly.
* Don't use multiple inheritance.
* Prefer composition over inheritance for new first-party types.
* Use table-driven tests rather than lots of small, similar tests.
* Add doc comments to publicly available methods and functions.
* Document code succinctly but thoroughly.
* Use type hinting in Python.
* Generally treat warnings as errors unless fixing the warning would cause difficult to fix breakages.

# Tooling
* Use 'uv' for building, running, and managing Python projects.
* If available, use podman when building images instead of Docker.

# Other important projects
* portal-conductor: Usually available at ../portal-conductor/. Provides an API for the portal.

# Commands
- npm run dev: Run the portal locally
- curl -k https://portal-conductor/: Test if the portal-conductor is available locally.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cyverse-de)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cyverse-de)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
