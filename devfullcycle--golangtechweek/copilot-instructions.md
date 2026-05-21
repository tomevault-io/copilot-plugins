## container-events

> Para inicializar a aplicação ou executar a aplicação com docker

# Quando precisar iniciar a aplicação pela primeira vez ou para retomar o projeto onde parou

- Para inicializar a aplicação, você precisa utilizar o comando docker comopose up -d
    - Caso esteja usando o podman: podman-compose up -d

- Todos os comandos devem ser executados sempre dentro dos containers através do docker comopse exec <container name> <command>

- Nome dos serviços no @docker-compose.yaml

---
> Source: [devfullcycle/golangtechweek](https://github.com/devfullcycle/golangtechweek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
