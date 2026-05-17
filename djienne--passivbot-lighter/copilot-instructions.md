## passivbot-lighter

> - **Always use `docker compose`** to manage the lighter bot container (not raw `docker run`)

# Passivbot Lighter - Development Notes

## Docker Deployment

- **Always use `docker compose`** to manage the lighter bot container (not raw `docker run`)
- The `docker-compose.yml` mounts `./:/app/` as a volume, so code changes are picked up on restart without rebuilding
- Rebuild image: `docker compose build passivbot-lighter-live`
- Restart: `docker compose up -d passivbot-lighter-live`
- Logs: `docker logs passivbot-lighter-live --tail 50 -f`
- The `passivbot-hype-live` container runs a separate strategy -- never touch it

---
> Source: [djienne/passivbot_lighter](https://github.com/djienne/passivbot_lighter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
