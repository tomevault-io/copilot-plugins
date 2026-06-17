## gestalt

> Gestalt es la plataforma de orquestación de agentes AI de SouthWest AI Labs. Workspace Rust con VFS, Swarm, Timeline y Tool Registry.

# Gestalt Skill

## Descripción
Gestalt es la plataforma de orquestación de agentes AI de SouthWest AI Labs. Workspace Rust con VFS, Swarm, Timeline y Tool Registry.

## Ubicación
`E:\scripts-python\gestalt-rust`

## Crates

| Crate | Binary | Descripción |
|-------|--------|-------------|
| gestalt_core | lib | VFS, auth, LLM, tools, MCP client |
| gestalt_timeline | `gestalt` | Orchestrator + timeline |
| gestalt_cli | `gestalt_cli` | REPL |
| gestalt_swarm | `gestalt_swarm` | Swarm coordinator |
| synapse-agentic | lib | Tool registry + Hive |

## Uso

### CLI
```bash
cd E:\scripts-python\gestalt-rust
cargo run -p gestalt_timeline --bin gestalt
cargo run -p gestalt_cli
```

### Build
```bash
cargo build --release -p gestalt_timeline
cargo build --release -p gestalt_cli
cargo build --release -p gestalt_swarm
```

## Configuración

```bash
export GESTALT_DATABASE_URL="surrealdb:memory"
export GESTALT_LLM__OPENAI__API_KEY="sk-..."
```

Ver `gestalt_core/src/application/CONFIG.md` para todas las variables.

## Herramientas Disponibles (12+)

- `scan_workspace` — directory tree
- `search_code` — vector similarity search
- `execute_shell` — shell commands
- `read_file` / `write_file`
- `git_status` / `git_log` / `git_branch` / `git_add` / `git_commit` / `git_push`
- `clone_repo` / `list_repos`
- `ask_ai` — query LLM

## Swarm

```bash
cargo run -p gestalt_swarm -- --agents 4
```

## MCP Client

Gestalt tiene un MCP client (no server) que puede conectarse a servers MCP externos.
Configurar en `config/mcp.json`.

## Autenticación

Google OAuth2 + PKCE disponible en `gestalt_core/adapters/auth/`.

---

*SouthWest AI Labs — AI agents that actually execute.*

---
> Source: [iberi22/gestalt](https://github.com/iberi22/gestalt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
