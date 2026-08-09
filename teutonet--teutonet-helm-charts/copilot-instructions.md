## teutonet-helm-charts

> - `.Values.ingress.provider` is the sole source of truth for which controller HelmRelease actually deploys (`templates/ingress/nginx.yaml`/`traefik.yaml`) — never gate that on dual-mode detection.

# base-cluster ingress architecture

- `.Values.ingress.provider` is the sole source of truth for which controller HelmRelease actually deploys (`templates/ingress/nginx.yaml`/`traefik.yaml`) — never gate that on dual-mode detection.
- "Dual-mode" (`base-cluster.ingress.dualMode`/`hasNginx`/`hasTraefik` in `_helpers.tpl`) only widens *supporting* config — cert-manager's `enableGatewayAPI`, gateway-api CRDs, Grafana dashboards — for a manual provider migration. It is not a supported deployment topology.

---
> Source: [teutonet/teutonet-helm-charts](https://github.com/teutonet/teutonet-helm-charts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
