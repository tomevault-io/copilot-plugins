## sonarqube-open-server

> This project provides a Docker Compose setup for running a local SonarQube server with a PostgreSQL database. It is designed for easy setup and configuration, allowing developers to quickly get a SonarQube instance running for code analysis.

# Project Overview

This project provides a Docker Compose setup for running a local SonarQube server with a PostgreSQL database. It is designed for easy setup and configuration, allowing developers to quickly get a SonarQube instance running for code analysis.

The main technologies used are:
- **SonarQube:** An open-source platform for continuous inspection of code quality.
- **PostgreSQL:** A powerful, open-source object-relational database system.
- **Docker Compose:** A tool for defining and running multi-container Docker applications.

The architecture consists of two main services:
- `sonarqube`: The SonarQube application server.
- `db`: The PostgreSQL database server.

These services are connected via a dedicated Docker network (`sonarnet`). Data is persisted using Docker volumes.

# Building and Running

The following commands are used to manage the SonarQube environment:

- **Start the services:**
  ```bash
  docker-compose up -d
  ```

- **Stop the services:**
  ```bash
  docker-compose down
  ```

- **View logs:**
  ```bash
  docker-compose logs -f
  ```

- **Restart the services:**
  ```bash
  docker-compose restart
  ```

- **Access SonarQube:**
  - **URL:** http://localhost:9000 (or http://localhost:9001 with the override file)
  - **Default credentials:**
    - **Username:** `admin`
    - **Password:** `admin`

# Development Conventions

- **Configuration:**
  - The primary configuration is managed through `docker-compose.yml` and `.env`.
  - For local development, `docker-compose.override.yml` can be used to customize settings without modifying the main configuration files.
- **Plugins:**
  - Custom plugins can be added by placing the JAR files in the `./plugins/` directory and restarting the `sonarqube` service.
- **Database:**
  - The PostgreSQL database is configured with a default user and password, which should be changed for production environments.
- **Code Analysis:**
  - The `README.md` provides instructions on how to analyze code using SonarScanner CLI, Maven, and Gradle.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mallow-Dev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Mallow-Dev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
