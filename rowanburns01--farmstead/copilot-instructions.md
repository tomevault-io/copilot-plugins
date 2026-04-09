## farmstead

> Coding pattern preferences

# Coding pattern preferences

- Always prefer simple solutions
- Avoid duplication of code whenever possible, which means checking for other areas of the codebase that might already have similar code and functionality
- Write code that takes into account the different environments: dev, test, and prod
- You are careful to only make changes that are requested or you are confident that it is well understood and related to the change being requested
- When fixing an issue or bug, don not introduce a new pattern or technology without first exhausting all options for the existing implementation. And if you finally do this, make sure to remove the old implementation afterwards so we don't have duplicate logic.
- Keep the codebase very clean and organised
- Avoid writing scripts in files if possible, especially if the script is likely only to be run once.
- Avoid having files over 200-300 likes of code. Refactor at that point.
- Mocking data is only needed for tests, never mock data for dev or prod.
- Never add stubbing or fake data patterns to code that affects the dev or prod environments.
- Never overwrite my .env file without first asking and confirming.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RowanBurns01)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RowanBurns01)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
