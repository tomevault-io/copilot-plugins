## build-and-run

> How to build and run mirror-gui. Apply when building, starting, stopping, or restarting the app.


# Build & Run

Container engine: **Podman** (Docker is not supported).

## Default: always rebuild with podman

Always rebuild with podman directly:

```bash
podman build -t localhost/mirror-gui .
```

## Run / manage the container

Use `IMAGE_NAME=localhost/mirror-gui` so the script uses the locally built image instead of pulling from the registry:

```bash
IMAGE_NAME=localhost/mirror-gui ./mirror-gui.sh            # start container on port 3000
IMAGE_NAME=localhost/mirror-gui ./mirror-gui.sh --stop
IMAGE_NAME=localhost/mirror-gui ./mirror-gui.sh --restart
./mirror-gui.sh --status
./mirror-gui.sh --logs
```

## Ports

| Context       | Port |
|---------------|------|
| Host (web)    | 3000 (configurable via `WEB_PORT`) |
| Container     | 3001 |

## After code changes

1. Rebuild the image: `podman build -t localhost/mirror-gui .`
2. Restart: `IMAGE_NAME=localhost/mirror-gui ./mirror-gui.sh --restart`

## E2E tests (Playwright)

Before running Playwright tests, install the browsers first:

```bash
npx playwright install chromium
```

Then run the tests (requires the app running on port 3000):

```bash
npx playwright test
```

---
> Source: [openshift/mirror-gui](https://github.com/openshift/mirror-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
