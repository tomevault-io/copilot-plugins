## yet-another-homelab-os

> This file outlines the development plan and key features for the `yethos` project, a script-based installer for setting up a homelab environment.

# Gemini Project: yet-another-homelab-os (yethos)

This file outlines the development plan and key features for the `yethos` project, a script-based installer for setting up a homelab environment.

## Project Goal

The primary goal is to create a single, idempotent bash script that automates the setup of a secure, accessible homelab environment on a standard Linux distribution. The script will handle the installation and configuration of essential services, including Docker, Traefik, Cloudflared, and TinyAuth.

## Core Components & Technologies

- **Orchestration:** Bash script (`install.sh`) will be the main entry point. Python may be used for more complex tasks if needed.
- **Containerization:** Docker and Docker Compose will be used to run all services.
- **Reverse Proxy:** Traefik will be used to manage and route incoming traffic to the appropriate services.
- **Tunneling:** Cloudflared will provide a secure tunnel to expose services to the internet without opening ports.
- **Authentication:** TinyAuth will be integrated as a Traefik middleware to provide authentication for services.

## Key Features

1.  **Automated Docker Setup:** The script will check for and install Docker and Docker Compose if they are not present.
2.  **Traefik Deployment:** Deploy Traefik via Docker Compose with a pre-configured setup for service discovery using Docker labels.
3.  **Cloudflared Integration:**
    - Install and configure `cloudflared` as a service.
    - Automatically create a Cloudflare tunnel.
    - Configure Traefik to route traffic from the tunnel.
4.  **TinyAuth Deployment:**
    - Deploy TinyAuth via Docker Compose.
    - Generate necessary configurations.
5.  **Middleware Integration:** Configure Traefik to use TinyAuth as an authentication middleware for services that require it.
6.  **Service Discovery:** Services will be exposed and configured automatically through Docker labels in their `docker-compose.yml` files.
7.  **Idempotency:** The installation script should be safe to run multiple times.

## Project Structure

```
/
├── .gitignore
├── GEMINI.md
├── README.md
├── install.sh
├── docker-compose.yml
└── config/
    ├── traefik/
    │   └── traefik.yml
    └── tinyauth/
        └── ...
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/calganaygun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/calganaygun)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
