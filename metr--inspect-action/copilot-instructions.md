## kubernetes-helm

> - **Important**: We use a Helm chart to create Kubernetes objects. Add new resources to the Helm chart instead of creating them directly with the k8s client

# Kubernetes Operations

- **Important**: We use a Helm chart to create Kubernetes objects. Add new resources to the Helm chart instead of creating them directly with the k8s client
  - The Helm chart is located at `hawk/api/helm_chart/`
- The Hawk API server uses pyhelm3 (https://github.com/azimuth-cloud/pyhelm3) to execute Helm commands. Under the hood, this delegates to the helm CLI
- Only the API server should execute Helm commands using pyhelm3. No other component should do that

---
> Source: [METR/inspect-action](https://github.com/METR/inspect-action) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
