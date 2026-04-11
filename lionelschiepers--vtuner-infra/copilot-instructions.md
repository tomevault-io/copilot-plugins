## vtuner-infra

> This repository contains the infrastructure for a home media streaming setup, managed with Docker. It consists of the following services:

# vTuner Infrastructure

## Project Overview

This repository contains the infrastructure for a home media streaming setup, managed with Docker. It consists of the following services:

*   **Pi-hole:** A dns service. This service is used to redirect official vTuner address to local YTuner service running on the same container network.
*   **Traffic Server:** A caching forward proxy that sits in front of the other services. It is configured to cache HTTP requests and resoles requests using Pi-hole.
*   **YTuner:** A service that simulate vTuner API for free (new).
*   **YCast:** A service that simulate vTuner API for free (obsolete - unmaintained).

## Building and Running

The services are built and deployed as Docker containers.

### Building

The `build.cmd` script builds and pushes the Docker images for the services.

```bash
.\build.cmd
```

### Running

The services are run as Docker containers.

*   **Pi-hole:** The `PiHole/setup.cmd` script runs the Pi-hole container.

## Development Conventions

*   The services are packaged as Docker containers.
*   The `build.cmd` script is used to build and push the Docker images.
*   The services are configured through a combination of configuration files and environment variables.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lionelschiepers)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lionelschiepers)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
