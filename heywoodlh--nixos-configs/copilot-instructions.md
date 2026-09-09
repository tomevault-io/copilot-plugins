## nixos-configs

> - `flake.nix`: stores variables for the deployments.

# flakes/kube layout

- `flake.nix`: stores variables for the deployments.
- `templates`: stores templated Kubernetes manifests.
- `manifests`: stores rendered Kubernetes manifests.
  - `manifests/apps.yaml`: full list of apps that get defined in ArgoCD.
- `render.sh`: helper script to render manifests. Apps that will get codified
  in ArgoCD should be added to the list of apps in `render.sh`.

---
> Source: [heywoodlh/nixos-configs](https://github.com/heywoodlh/nixos-configs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
