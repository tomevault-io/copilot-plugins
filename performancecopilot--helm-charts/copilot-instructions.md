## helm-charts

> This file contains development context and notes for the Performance Co-Pilot Helm charts.

# Claude Development Notes

This file contains development context and notes for the Performance Co-Pilot Helm charts.

## Project Overview

These Helm charts are for deploying Performance Co-Pilot (PCP) services on Kubernetes. PCP is a system performance analysis toolkit that provides real-time and historical metrics collection.

## References

- [PCP Website](https://pcp.io) - Official documentation and guides
- [PCP GitHub Repository](https://github.com/performancecopilot/pcp) - Source code and issues
- [Releases and macOS DMG](https://github.com/performancecopilot/pcp/releases) - Pre-built dmg packages
- [PCP Package Repository](https://packagecloud.io/performancecopilot/pcp) - Pre-built deb/rpm packages
- [Container Documentation](https://github.com/performancecopilot/pcp/blob/main/build/containers/README.md) - PCP containers usage and configuration

## Repository Context

These Helm charts will be hosted at the PCP helm chart repository (quay.io/performancecopilot-helm-charts) alongside the container images (quay.io/performancecopilot).  There are two Helm charts - one for the main PCP container and the other for the archive-analysis container containing Grafana and grafana-pcp.

## Key Design Decisions

### Single Instance Design
- `replicaCount: 1` is fixed - PCP monitors system resources and multiple replicas would create duplicate data
- Autoscaling is disabled since horizontal scaling doesn't make sense for system monitoring

### Container Integration
- Based on `quay.io/performancecopilot/pcp:latest` and `quay.io/performancecopilot/archive-analysis:latest` containers
- Using systemd init (`/usr/sbin/init`) requiring special Kubernetes handling
- Privileged mode required for full system access (for the pcp container only)

### Storage Strategy
- Persistent volumes for `/var/log/pcp/pmlogger` (1Gi) and `/var/log/pcp/pmproxy` (1Gi)
- ConfigMaps for OpenMetrics PMDA configuration
- Optional host filesystem mount for advanced monitoring

## Environment Variables

Key PCP configuration via environment variables:
- `PCP_SERVICES`: Controls which PCP services start (default: pmcd,pmie,pmlogger,pmproxy)
- `PCP_DOMAIN_AGENTS`: Comma-separated list of agents to enable (e.g., "bpf,bpftrace")
- `HOST_MOUNT`: Path to host root mount for host monitoring mode
- `KEY_SERVERS`: Redis/Valkey connection specs

## Host Monitoring Mode

For eBPF and full host monitoring:
```yaml
hostMonitoring:
  enabled: true
hostNetwork: true
env:
  HOST_MOUNT: "/host"
  PCP_DOMAIN_AGENTS: "bpf,bpftrace"
```

## Health Checks

- **Liveness**: `systemctl is-active pmcd` (systemd-aware)
- **Readiness**: TCP socket check on pmcd port (44321)
- Adjusted timing for systemd container startup

## Testing Commands

```bash
# Validate chart
helm lint ./pcp

# Template output
helm template pcp ./pcp

# Install with host monitoring
helm install pcp ./pcp --set hostMonitoring.enabled=true --set hostNetwork=true --set env.HOST_MOUNT="/host"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/performancecopilot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/performancecopilot)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
