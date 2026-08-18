## tron

> Classic Tron in the browser at https://tron.kodloki.io. Single shared arena:

# tron — multiplayer light-cycles

Classic Tron in the browser at https://tron.kodloki.io. Single shared arena:
everyone who connects plays together, rounds start at 2+ players.

## Architecture

One Node 20 + TypeScript process, one container (`bholcombe/tron-server`):

- `src/server/` — Express (static files, `/healthz`) + `ws` WebSocket server on
  port 3000. Authoritative 15 Hz tick loop in `game.ts`; round state machine
  `waiting → countdown → playing → round_over`.
- `src/client/` — no-framework Canvas 2D client, bundled with esbuild into
  `dist/public/main.js`; `public/index.html` is copied alongside.
- `src/shared/protocol.ts` — message types + grid constants (80×60, 15 Hz).

The arena lives in server memory, so `replicas: 1` with `strategy: Recreate`
is a hard requirement, not a tuning choice. No database, no secrets.

## Commands

- `npm run dev` — build client, run server with tsx watch on :3000
- `npm run build` — client (esbuild) + server (tsc) into `dist/`
- `scripts/deploy.sh` — buildx → push to Docker Hub → bump sha tag in
  `K8s/deployment.yaml` → commit → push (GitHub canonical, gitea mirror);
  ArgoCD auto-syncs. Requires a clean tree.

## Deploy

Standard kodloki conventions: LKE cluster `tow-c1` (kubeconfig
`~/.kube/linode-config`), flat manifests in `K8s/`, ArgoCD Application in
`argocd/tron-app.yaml` watching GitHub `K8s/`, ingress-nginx +
`letsencrypt-prod`, host `tron.kodloki.io` → LB `172.232.176.47`. The ingress
carries the WebSocket annotations (3600 s proxy timeouts,
`websocket-services`).

---
> Source: [brandonholcombe/tron](https://github.com/brandonholcombe/tron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
