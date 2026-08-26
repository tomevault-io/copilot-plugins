## dockernoharm

> Never docker-compose down -v (preserve volumes); never git commit/push without user confirmation


# Docker and Git — Never Do

## Docker
- **NEVER** use `docker-compose down -v`. It removes all Docker volumes, including database data.
- **Always** use `docker-compose down` (without `-v`) to preserve data volumes.

## Git
- **NEVER** run `git commit .` or `git push origin main` without getting positive confirmation from the user.

---
> Source: [dfirtnt/Huntable-CTI-Studio](https://github.com/dfirtnt/Huntable-CTI-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
