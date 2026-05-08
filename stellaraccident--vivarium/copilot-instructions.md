## vivarium

> Workspace for managing a KVM/QEMU virtualization lab with GPU passthrough,

# Vivarium — VM Lab Management Workspace

Workspace for managing a KVM/QEMU virtualization lab with GPU passthrough,
virtiofs shared filesystems, and mixed Linux/Windows guest VMs.

## Context
- **Host OS**: Fedora Workstation (KDE Spin recommended, see docs/host-os-selection.md)
- **Hypervisor**: KVM/QEMU with libvirt
- **User**: stella (works at AMD)
- **CPU**: AMD Genoa/Bergamo (EPYC server-class)

## GPUs (3x AMD)
- **Radeon Pro W7900** (RDNA3 / Navi 31, `1002:7448`) → host GPU (daily desktop). Most flaky device reset of the three — stays on host where reset bugs don't matter.
  - PCI: `0000:03:00.0` / Audio: `0000:03:00.1` (`1002:ab30`) / IOMMU group 35 / PCIe 4.0 x16
- **RX 9070** (RDNA4 / Navi 48, `1002:7550`) → passthrough to fedora-dev (ROCm/compute). Has broken PCIe bus reset — fixed via patched `vendor-reset` module ([stellaraccident/vendor-reset](https://github.com/stellaraccident/vendor-reset)) with `NO_BUS_RESET` flag. See `docs/gpu-passthrough.md` for details.
  - PCI: `0000:43:00.0` / Audio: `0000:43:00.1` (`1002:ab40`) / IOMMU group 49 / PCIe 5.0 x16
- **Radeon VII** (Vega 20, `1002:66af`) → passthrough to win11 (Bob). Known reset bug, fixed by `gnif/vendor-reset` DKMS module (BACO reset for Vega 20).
  - PCI: `0000:85:00.0` / Audio: `0000:85:00.1` (`1002:ab20`) / IOMMU group 21 / PCIe 3.0 x16

## Workspace Layout

```
vivarium/
├── CLAUDE.md              # You are here
├── README.md              # Project overview and status
├── docs/                  # Reference documentation (deep dives)
│   ├── host-os-selection.md
│   ├── gpu-passthrough.md
│   ├── virtiofs.md
│   └── windows-guest.md
├── procedures/            # Step-by-step runbooks (agent-executable)
│   ├── 01-host-install.md
│   ├── 02-vfio-setup.md
│   ├── 03-virtiofs-setup.md
│   ├── 04-linux-guest.md
│   └── 05-windows-guest.md
├── configs/               # Template configuration files
│   ├── host/              # Host-side configs (modprobe, dracut, grub)
│   └── libvirt-templates/ # VM definition XML templates
└── scripts/               # Helper scripts (IOMMU checker, etc.)
```

## Key Conventions

- **Procedures are ordered**: numbered sequentially, each builds on the prior
- **Procedures are agent-friendly**: each step is a concrete command or check with expected output
- **Docs are reference material**: consult when a procedure says "see docs/foo.md for details"
- **Configs are templates**: copy and customize per-machine, never use blindly
- **Hardware-specific values** (PCI IDs, IOMMU groups, CPU topology) must be discovered at runtime — procedures include discovery commands

## Important Notes

- GPU passthrough requires IOMMU support in BIOS (Intel VT-d / AMD-Vi)
- The host GPU and passthrough GPU(s) must be in separate IOMMU groups
- virtiofs requires shared memory backing for guest RAM
- Windows guests need virtio-win drivers (ISO from Fedora project)
- inotify does NOT work across the virtiofs boundary — file watchers in guests won't see host-side changes

---
> Source: [stellaraccident/vivarium](https://github.com/stellaraccident/vivarium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
