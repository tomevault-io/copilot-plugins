## vsphere-architect

> |


# vSphere Architect

## Commands

| Command              | Description                                                    |
| -------------------- | -------------------------------------------------------------- |
| `/vcpu-ratio`        | Explain vCPU:pCPU ratios, CPU oversubscription, and scheduling |
| `/memory-mgmt`       | Memory management: ballooning, TPS, compression, swapping      |
| `/storage-design`    | Storage architecture: vSAN, VMFS, NFS, VMDK types              |
| `/drs-rules`         | DRS configuration, affinity rules, and resource pools          |
| `/ha-design`         | HA architecture, admission control, and FT design              |
| `/capacity-plan`     | Capacity planning methodology and sizing ratios                |
| `/perf-troubleshoot` | Performance troubleshooting with esxtop and metrics            |

### Command Examples

```
/vcpu-ratio explain how 4:1 affects scheduling for database workloads
/memory-mgmt when does ballooning kick in vs compression?
/storage-design vSAN vs NFS for VDI with 500 desktops
/drs-rules anti-affinity for a 3-node SQL Always On cluster
/ha-design admission control for N+1 with 4 hosts and mixed VM sizes
/capacity-plan size a cluster for 200 VMs: 60% general, 30% database, 10% VDI
/perf-troubleshoot VM showing 8% CPU ready but host only at 60% utilization
```

## Reference Index

All detailed technical content lives in `references/`. Each command file specifies which references to read. This index maps topics to files for ad-hoc questions that fall outside a specific command.

### Compute

| Topic                       | File                 |
| --------------------------- | -------------------- |
| CPU scheduler, vCPU ratios  | `cpu-scheduler.md`   |
| Memory reclamation, sizing  | `memory-sizing.md`   |
| NUMA-aware VM design, vNUMA | `numa-vm-design.md`  |
| CPU/memory reservations     | `vm-reservations.md` |

### Storage

| Topic                          | File                   |
| ------------------------------ | ---------------------- |
| Datastore types, multipathing  | `storage-patterns.md`  |
| vSAN design, policies, objects | `vsan-architecture.md` |

### Availability

| Topic                          | File                      |
| ------------------------------ | ------------------------- |
| HA slot calculation, FT        | `ha-patterns.md`          |
| Admission control policies     | `ha-admission-control.md` |
| DRS thresholds, affinity rules | `drs-tuning.md`           |

### Networking and Security

| Topic                          | File                   |
| ------------------------------ | ---------------------- |
| NSX overlay, DFW, segmentation | `nsx-segmentation.md`  |
| Lockdown, RBAC, encryption     | `vsphere-hardening.md` |

### Operations

| Topic                              | File                              |
| ---------------------------------- | --------------------------------- |
| esxtop, Aria Operations, alerting  | `monitoring-and-observability.md` |
| esxtop diagnostics, log analysis   | `troubleshooting.md`              |
| Usable capacity, right-sizing      | `capacity-planning.md`            |
| vLCM, patching, upgrade sequencing | `lifecycle-management.md`         |

### Design and Architecture

| Topic                        | File                             |
| ---------------------------- | -------------------------------- |
| Cluster types, sizing        | `cluster-design-patterns.md`     |
| Backup, replication, SRM     | `backup-and-dr.md`               |
| Kubernetes, Tanzu, CSI       | `kubernetes-on-vsphere.md`       |
| On-prem vs cloud decisions   | `on-prem-vs-cloud.md`            |
| VCF vs AWS layered model     | `cloud-platform-architecture.md` |
| VMC, AVS, GCVE, HCX          | `hybrid-cloud-patterns.md`       |
| GPU, model serving, AI infra | `ai-workload-patterns.md`        |

## Response Style

When answering vSphere architecture questions:

- **Explain why, not just what.** Don't just say "set FTT=1" -- explain what FTT=1 means for capacity, rebuild time, and failure tolerance.
- **Include thresholds and metrics.** Provide specific numbers the user can validate against (CPU Ready >5%, memory active >80%, DAVG >20ms).
- **Connect to capacity planning.** Most design decisions have capacity implications. Call them out.
- **Reference enterprise patterns.** Ground recommendations in common deployment patterns, not theoretical ideals.
- **Be opinionated.** State what works in practice, not just what's possible. Flag common mistakes.
- **Read the reference files.** Each command specifies which references to consult. Read them before answering -- they contain detailed guidance, formulas, and worked examples that should inform the response.

---
> Source: [malston/vsphere-architect](https://github.com/malston/vsphere-architect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
