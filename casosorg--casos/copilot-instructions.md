## casos

> A Kubernetes management platform with a Go backend and React frontend.

# casos

A Kubernetes management platform with a Go backend and React frontend.

## Project Structure

- `main.go` — entry point
- `controllers/` — HTTP controllers (Beego routing)
- `object/` — business logic and data models
- `routers/` — route configuration
- `proxy/` — kube-proxy related logic
- `conf/app.conf` — backend configuration
- `web/` — React frontend (see below)

## Frontend (web/)

Stack: React 18 + shadcn/ui (Radix + Tailwind v4), built with Vite.

**Package manager: yarn.** The lock file is `web/yarn.lock`. Do not use npm or pnpm.

```bash
# Install dependencies
cd web && yarn install

# Start dev server (port 8002)
cd web && yarn start

# Production build (outputs to web/build/)
cd web && yarn build

# End-to-end tests (starts its own backend and dev server)
cd web && yarn ui:test
```

`-tags embed` compiles `web/build/` into the binary, so run the production
build before a standalone build. See `web/FRONTEND.md` for the component
contract and the selector hooks the Playwright suite depends on.

Registry is set to yarnpkg. Configure it via:

```bash
yarn config set registry https://registry.yarnpkg.com
```

or add to `web/.yarnrc`:

```
registry "https://registry.yarnpkg.com"
```

The dev server proxies API requests to the backend at `http://localhost:20080` (configured under `server.proxy` in `web/vite.config.js`, and overridable with `BACKEND_URL`).

## Backend

Language: Go 1.26, module path `github.com/casosorg/casos`.

```bash
# Run
go run main.go

# Build
go build -o casos.exe .
```

## Notes

- All yarn/frontend commands must be run inside `web/`, not the repo root.
- Do not commit `web/node_modules/`.

## Code Style

- Don't add Go tests unless asked. Verify changes with `go build ./...` and
  `go vet ./...` instead.
- Keep comments sparse. Comment only what the code can't say itself — a
  non-obvious constraint, a workaround, an unexpected invariant. No comments
  that restate the line below them, and no doc comments on self-explanatory
  functions.

---
> Source: [casosorg/casos](https://github.com/casosorg/casos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
