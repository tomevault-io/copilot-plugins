## index

> │   Caddy   │  (reverse proxy, TLS)

# Appsmith — Architecture Overview

```
                    ┌───────────┐
                    │   Caddy   │  (reverse proxy, TLS)
                    └─────┬─────┘
                          │
             ┌────────────┼────────────┐
             │            │            │
    ┌────────▼───┐  ┌─────▼─────┐  ┌──▼──────┐
    │   Client   │  │  Server   │  │   RTS   │
    │ React SPA  │  │ Spring    │  │ Node.js │
    │ TypeScript │  │ Boot/Java │  │ Express │
    └────────────┘  └─────┬─────┘  └─────────┘
                          │
             ┌────────────┼────────────┐
             │            │            │
        ┌────▼───┐  ┌────▼────┐  ┌───▼──────┐
        │MongoDB │  │Postgres │  │ Temporal │
        │/ Redis │  │         │  │(workflows)│
        └────────┘  └─────────┘  └──────────┘
```

- **Client** (`app/client/`) — React + TypeScript SPA, Redux-Saga, Yarn monorepo
- **Server** (`app/server/`) — Java 17, Spring Boot 3.x (Reactive WebFlux), Maven, MongoDB
- **RTS** (`app/client/packages/rts/`) — Node.js Express real-time server, SCIM, workflow proxy
- **Deploy** (`deploy/`) — Docker, Helm, AWS, Ansible, Caddy reverse proxy
- **Orchestration** — supervisord in Docker, optional Keycloak (SSO)

## EE vs CE Pattern

Enterprise and community edition code coexist in the same repo. Look for `ce/` and `ee/` directories within both `app/server` and `app/client`. Enterprise-specific logic extends or overrides CE implementations.

---
> Source: [tunacosgun/lowcode-studio](https://github.com/tunacosgun/lowcode-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
