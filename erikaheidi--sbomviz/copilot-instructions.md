## sbomviz

> All agents and contributors working in this repository must follow these rules:

# AGENTS

## Mandatory execution policy

All agents and contributors working in this repository must follow these rules:

1. Run the application only through the containerized development environment.
2. Install dependencies only inside the container image.
3. Never install project dependencies on the host machine.
4. Never run the application on the host machine.

## Approved commands

Use only container-based commands for local development:

```bash
docker compose up --build
docker compose down
```

If an agent needs to execute Python commands, it must run them in the container context, not on host.

---
> Source: [erikaheidi/sbomviz](https://github.com/erikaheidi/sbomviz) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
