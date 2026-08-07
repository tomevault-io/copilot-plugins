## kanata

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is kanata

kanata is a Swift agent that makes a macOS (Apple Silicon) host appear as a Kubernetes worker node. Every Pod runs as an independent lightweight Linux VM using Apple's Containerization framework. A single `kanata` binary hosts everything: node controllers, pod VMs, gateway VMs, the Kubernetes control plane, and host-side proxies. No separate helper daemons.

## Requirements

- macOS 26 (Tahoe)+, Apple Silicon (arm64), Xcode 26+ (Swift 6.2+)
- mise for pinned tool versions (kind, kubectl, protoc, go, swiftlint)
- Docker only needed for `make build-nat-kernel` and `make build-gw-image`

## Build / Test / Lint Commands

```bash
make bootstrap          # one-time: mise install + fetch kata kernel
make build              # swift build (debug) + codesign VM binaries
make test               # swift test (use this, not bare swift test — handles CLT paths)
swift test --filter <TestTarget>/<TestCase>  # run a single test
swiftlint lint          # lint (Sources + Tests)
xcrun swift-format lint --strict -r Sources/ Tests/  # format check
```

`make build` calls `make sign` automatically — VM binaries need `signing/vm.entitlements` for Virtualization.framework.

## CI

CI runs on GitHub-hosted `macos-26` runners (`.github/workflows/ci.yml`):
- `swiftlint lint`
- `xcrun swift-format lint --strict -r Sources/ Tests/`
- `swift build`
- `make test`

Release (`.github/workflows/release.yml`) runs 5 build jobs in parallel, then a final release job:
- `fetch-kernel` / `build-kanata-probe` / `build-nat-kernel` / `build-gw-image` (`ubuntu-26.04-arm`)
- `build-swift` (`macos-26`) — swift build + codesign
- `release` (`macos-26`, needs all above) — bundle, GitHub Release, Homebrew tap update

## Code Style

- **Indent**: 4 spaces (`.swift-format`)
- **Line length**: 150 (swift-format), warning at 200 / error at 400 (SwiftLint)
- **SwiftLint**: config in `.swiftlint.yml`, only `todo` rule disabled
- **Conventional Commits**: `<type>(<scope>): <description>` — types: feat, fix, docs, test, build, cleanup, refactor

## Architecture (key relationships)

```
kanata (CLI entry point, Proxy/)
  ├── ClusterManager       — per-cluster paths, subnet/port allocation, launchd, kind-compatible config parsing
  ├── ControlPlane         — embedded CP manifests (Resource Bundle), PKI, token substitution
  ├── StaticPodSource      — watches manifest dir → StaticPodEvent stream
  ├── NodeController       — Node registration + Lease heartbeat (N instances in multi-node)
  ├── PodController        — Pod watch → reconcile → PodRuntime (N instances in multi-node)
  │     └── PodRuntime     — VM lifecycle, IP pool, loopback UDS gRPC to PodHost
  │           └── PodHost  — in-process PodShimService per Pod (VM + container)
  ├── KubeletServer        — Hummingbird HTTPS: exec/attach (WebSocket v5), logs, port-forward (SPDY)
  ├── GatewayVM            — per-node gateway VM: nft rules (ProxyReconciler), DNS zone, LB VIP allocation
  ├── StorageController    — default StorageClass, PVC provisioner, metrics-server deployer
  ├── KanataCore           — shared types: config, RawRestClient, exec abstractions, DNS resolver, logging
  ├── KanataShimAPI        — gRPC service contract (hand-written stubs from shim.proto)
  └── VmnetSupport         — in-process vmnet network ownership (1 reservation per cluster)
```

**1 Pod = 1 Linux VM**. IPC between PodRuntime and each PodShimService is gRPC over loopback UDS (`/tmp/kanata-shim-<pod-uid>.sock`).

**IP plan per cluster** (192.168.10x.0/24): .1 = vmnet bridge, .2 = apiserver, .3 = etcd, .4 = CoreDNS, .5+ = gateway VMs, then dynamic pod pool, .200-.250 = LB VIP pool.

**Host-side proxies** (in `Sources/kanata/Proxy/`):
- `APIServerProxy` — persistent mux tunnel (`MuxSession`) to apiserver for kubectl
- `NodePortProxy` — dynamic `0.0.0.0:<nodePort>` listeners, exec relay to pods
- `HostPortProxy` — kind-compatible `extraPortMappings`, exec relay

## Key conventions

- K8s 1.36+ / WebSocket only (no SPDY for exec/attach). Port-forward uses SPDY/3.1.
- `SwiftkubeModel` lacks some fields — `RawRestClient` is used for raw JSON PUT (e.g., NodeStatus with `declaredFeatures`).
- Static pods use `kanata.io/static-pod: "true"` and `kanata.io/static-pod-ip` annotations.
- Gateway VM UID: `"kanata-gateway"` (single-node) or `"kanata-gateway-<nodeName>"` (multi-node).
- `run-node` is an internal subcommand invoked by launchd — not user-facing.

## kanata-probe

`tools/kanata-probe/` — static ARM64 Go binary injected into Pod VMs via virtiofs. Handles httpGet/tcpSocket probes, port-forward, route injection, and `tcp-proxy` (mux tunnel endpoint). Build: `make build-kanata-probe`.

## e2e tests

All e2e tests require real macOS 26 hardware (VMs + launchd). Set `KANATA_E2E_HARDWARE=1`.

```bash
make e2e-multinode    # multi-node cluster tests
make e2e-storage      # PVC provisioning + etcd persistence
make e2e-metrics      # metrics-server + namespace deletion
KANATA_E2E_HARDWARE=1 bash e2e/e2e-pod-lifecycle.sh   # pod lifecycle QA
KANATA_E2E_HARDWARE=1 bash e2e/e2e-ingress.sh         # ingress + LB tests
```

Cluster runtime state: `~/.kanata/clusters/<name>/` (logs, PKI, static-pods, cluster.json).

## ADR process

Decisions in `docs/adr/`. Current highest: ADR-0027 (numbers 0018-0020 were skipped intentionally). New ADRs start as Draft, become Accepted after implementation + review.

---
> Source: [mazrean/kanata](https://github.com/mazrean/kanata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
