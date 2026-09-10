## fastgateway

> ﻿# Repository Guidelines

﻿# Repository Guidelines

FastGateway is a self-hosted reverse-proxy and tunnel gateway: a .NET 10 control plane (JWT-protected Minimal API plus per-server Kestrel/YARP gateways) with a React 19 dashboard and an outbound tunnel agent.

## Project Structure & Module Organization

- `src/FastGateway` — main host. Key areas: `Gateway/` (YARP route/cluster construction, TCP/UDP `StreamProxyManager`), `Services/` (Minimal API endpoint groups), `Middleware/` (client-IP, statistics, timeout, access control, failover), `Tunnels/` (server-side tunnel registration and streams), `Infrastructure/` (JWT helpers, AOT JSON contexts, IP policy matching, settings), plus `Dto/`, `Options/`, `BackgroundTask/`, `Extensions/`.
- `.github/workflows/` — CI: `docker-image.yml` builds the container images, `release.yml` publishes a portable single-file build (`-p:PublishAot=false`).
- `src/TunnelClient` — standalone tunnel agent. Reads `tunnel.json`, registers via `/internal/gateway/Server/register`, forwards traffic to local services over HTTP/2 or WebSocket.
- `src/Core` — shared stream wrappers and entities/enums used by server and agent.
- `src/Certes` — vendored, AOT-compatible ACME client. Must stay free of Newtonsoft.Json/BouncyCastle.
- `web` — React 19 + TypeScript + Vite dashboard. Its `dist` output lands in `wwwroot` via `build.bat` (into `publish/wwwroot`) or via the Dockerfile's `web` stage (`COPY --from=web /web/dist ./src/FastGateway/wwwroot` before `dotnet publish`).
- No test projects exist yet; place new ones under `tests/`.

## Build, Test, and Development Commands

```bash
dotnet build FastGateway.sln                                   # build backend
dotnet run --project src/FastGateway/FastGateway.csproj        # run host (http://localhost:5202)
dotnet run --project src/TunnelClient/TunnelClient.csproj -- -c ./src/TunnelClient/tunnel.json
cd web && npm install && npm run dev                           # dashboard dev server, proxies /api to :5202
cd web && npm run build                                        # type-check (tsc -b) + production build
cd web && npm run lint                                         # ESLint, zero warnings allowed
docker compose -f docker-compose.yml up -d --build             # full stack
```

Native AOT is enabled by default; pass `-p:PublishAot=false` for a portable single-file build (as the release workflow does).

## Coding Style & Naming Conventions

- C#: 4-space indentation, file-scoped namespaces, PascalCase types/methods; follow existing patterns in `Services/` and `Middleware/`.
- TypeScript/React: functional components, PascalCase component files, camelCase functions/variables; ESLint enforces style with `--max-warnings 0`.
- Comments: brief and high-signal — explain intent, invariants, or non-obvious behavior (e.g. AOT `JsonTypeInfo` constraints); never restate the code.
- Keep `src/Certes` AOT-safe: no reflection-heavy serializers or dynamic loading.

## Testing Guidelines

No automated tests exist yet. If you add them, use xUnit under `tests/` and name tests `<Method>_<Scenario>_<Expectation>`. Run with `dotnet test FastGateway.sln`.

## Commit & Pull Request Guidelines

- Conventional Commits: `feat:`, `fix:`, optionally scoped (`fix(acme):`). Descriptive Chinese or English summaries are both accepted (see `git log`).
- One logical change per commit; squash feature branches before merging.
- PRs: reference the linked issue, describe the change and how it was verified, include screenshots for UI changes, and ensure `npm run lint` and `dotnet build` pass.

---
> Source: [239573049/FastGateway](https://github.com/239573049/FastGateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
