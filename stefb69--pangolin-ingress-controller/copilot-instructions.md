## pangolin-ingress-controller

> Auto-generated from all feature plans. Last updated: 2025-11-30

# pangolin-ingress-controller Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-11-30

## Active Technologies
- Go 1.21+ + controller-runtime v0.17+, k8s.io/api networking/v1 (002-sso-multipath-support)
- N/A (Kubernetes CRDs only) (002-sso-multipath-support)
- Go 1.24.0 + controller-runtime v0.17.0, k8s.io/api v0.29.0 (003-multi-host-ingress)
- N/A (Kubernetes CRDs via controller-runtime) (003-multi-host-ingress)

- Go 1.21+ + controller-runtime v0.17+, client-go, golang.org/x/net/publicsuffix (001-pic-mvp)

## Project Structure

```text
src/
tests/
```

## Commands

# Add commands for Go 1.21+

## Code Style

Go 1.21+: Follow standard conventions

## Recent Changes
- 003-multi-host-ingress: Added Go 1.24.0 + controller-runtime v0.17.0, k8s.io/api v0.29.0
- 002-sso-multipath-support: Added Go 1.21+ + controller-runtime v0.17+, k8s.io/api networking/v1

- 001-pic-mvp: Added Go 1.21+ + controller-runtime v0.17+, client-go, golang.org/x/net/publicsuffix

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stefb69) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
