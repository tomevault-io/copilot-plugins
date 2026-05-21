## aspire-rules

> 1. Reference file [aspire_references.md](mdc:ai/aspire_references.md) for service endpoints and configuration.

## Aspire Rules

1. Reference file [aspire_references.md](mdc:ai/aspire_references.md) for service endpoints and configuration.
2. Setup Alias:
   - run `sudo ifconfig lo0 alias 127.0.0.2`
   - run `sudo ifconfig lo0 alias 127.0.0.3`
   - run `sudo ifconfig lo0 alias 127.0.0.4`
3. Workflow:
   - Run Docker: `docker compose -f station/src/Aevatar.Aspire/docker-compose.yml up -d`
   - Make sure all services in [docker-compose.yml](mdc:station/src/Aevatar.Aspire/docker-compose.yml) are running. If not, run docker again
   - Start Aspire [Program.cs](mdc:station/src/Aevatar.Aspire/Program.cs)
   - When prompted with login URL (e.g., `https://localhost:18888/login?t=token`), open it
   - Get authentication key from AuthServer API (`http://localhost:7001/connect/token`)
   - Use this key for all authenticated operations (e.g. `http://localhost:7002` endpoints)

---
> Source: [aevatarAI/aevatar-station](https://github.com/aevatarAI/aevatar-station) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
