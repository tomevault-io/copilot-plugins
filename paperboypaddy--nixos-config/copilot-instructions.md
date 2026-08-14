## nixos-config

> Declarative NixOS configuration for the phnet.xyz home network. The goal is to

# Project Context for AI Agents

## What this is

Declarative NixOS configuration for the phnet.xyz home network. The goal is to
migrate services away from Docker/Portainer toward NixOS and eventually
Kubernetes. Started as a learning project for the Nix language.

## Owner / Environment

- Owner: Padraic (Paperboypaddy on GitHub)
- Domain: phnet.xyz
- This repo will be pushed to a public/private GitHub repo named `nixos-config`
  (remote TBD — user will create it and provide the URL)
- Develop on `main`, push upstream when instructed

## Network Overview (homelab)

- Router/Firewall: Opnsense (192.168.8.1) — also does internal DNS (Unbound)
- Hypervisor: Proxmox at 192.168.9.136 (Server.lan); also 192.168.8.21 and 192.168.8.22
- Storage: TrueNAS Scale at 192.168.9.238 (SMB/NFS/iSCSI)
- Docker hosts (being migrated away from):
  - Media-Services: 192.168.8.69 (Traefik + Arr stack, Immich, NetBox, etc.)
  - Local-Portainer: hosts internal utilities (Vaultwarden, Grafana, Uptime Kuma)
- Reverse proxy: Traefik v3 with Cloudflare DNS-01 certs
- Identity: planning to run LLDAP (LDAP) + SSSD for Linux auth, Authentik for web SSO
- VLANs: currently flat single VLAN (192.168.8.0/23); segmentation planned
- Old compose repo: https://github.com/Paperboypaddy/phnet-compose-files (reference only)

## Plan / Milestones

1. Stand up a NixOS VM on Proxmox (2 vCPU / 4GB RAM / 32GB disk) as a sandbox
2. Learn the Nix language + flakes with a minimal, documented config
3. Deploy LLDAP inside the NixOS VM as the first real service
4. Replicate the config pattern to more hosts
5. (Later) Use NixOS as the base for Kubernetes nodes

## Tech decisions

- Flakes enabled (nix flakes + nix-command experimental features)
- Single-file hosts/<hostname>/configuration.nix layout to start, modularize later
- Imperative bootstrap only where unavoidable; everything else declarative
- `nixos-rebuild switch` for deployment; keep config in sync with what's on the VM

## Conventions

- Format: `nixfmt` or `alejandra`
- Never put secrets in the repo; use sops-nix or age encryption for secrets
- Document non-obvious Nix idioms with short comments
- Every host gets a short comment header describing its purpose

---
> Source: [Paperboypaddy/nixos-config](https://github.com/Paperboypaddy/nixos-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
