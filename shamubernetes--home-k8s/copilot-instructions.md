## home-k8s

> - **Configuration**: Cluster configuration and patches live under `talos/`.


### Talos conventions

- **Configuration**: Cluster configuration and patches live under `talos/`.
  - Authoritative config: [talos/talconfig.yaml](mdc:talos/talconfig.yaml)
  - Cluster-generated artifacts: `talos/clusterconfig/` (contains `talosconfig` and per-node YAMLs)
  - Patches: `talos/patches/global/` and `talos/patches/controlplane/`
- **Helmfile**: [talos/helmfile.yaml](mdc:talos/helmfile.yaml) orchestrates boot-time system components where applicable.
- **Secrets**: Keep Talos secrets encrypted (e.g., [talos/talsecret.sops.yaml](mdc:talos/talsecret.sops.yaml)); never commit decrypted secrets.
- **Editing**: Use `sops` to edit encrypted files; preserve 2-space indentation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shamubernetes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
