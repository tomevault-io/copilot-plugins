## docker-rebuild

> Auto rebuild Docker containers after code changes


# Docker Rebuild After Changes

After making any code changes to the project, always rebuild and restart the Docker containers so the changes are reflected immediately:

```bash
docker compose down && docker compose up -d --build
```

Run this from the project root (`/Users/miguelmedeiros/code/unforgit`).

---
> Source: [MiguelMedeiros/unforgit](https://github.com/MiguelMedeiros/unforgit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
