## linux-vm-provisioner

> This repo is organized around a three-phase workflow:

# AGENTS

## Purpose

This repo is organized around a three-phase workflow:

1. refresh provider defaults
2. provision a VM and run Ansible

Keep those phases separate. Do not mix slow account discovery into the hot provisioning path.

## Source of Truth

- AWS defaults live in [aws.yaml](/Users/lee/Projects/git/linux-vm-provisioner/skills/provision-cloud-vm/runtime/providers/aws.yaml)
- Ansible bootstrap lives in [ansible](/Users/lee/Projects/git/linux-vm-provisioner/skills/provision-cloud-vm/runtime/ansible)
- end-to-end AWS provisioning lives in [provision_aws_vm.sh](/Users/lee/Projects/git/linux-vm-provisioner/skills/provision-cloud-vm/runtime/scripts/provision_aws_vm.sh)
- root `scripts/` contains thin wrappers for shell convenience
- skill instructions live in [skills/provision-cloud-vm](/Users/lee/Projects/git/linux-vm-provisioner/skills/provision-cloud-vm)

## Provisioning Rules

- Use existing AWS key pairs only
- Use the stored VPC and subnet only
- Reuse the stored security group instead of creating one per VM
- Resolve AMIs dynamically at launch time
- Treat GCP as future work until a provider file exists

## Phase 3 Rules

- Install `llama-server` into `$HOME/.local/bin` for the Ansible-managed user
- Install Docker and use a shell wrapper to pull and run the llama.cpp container
- Keep Hugging Face credentials in `$HOME/.env`

## Editing Guidance

- Prefer updating skill runtime defaults in [aws.yaml](/Users/lee/Projects/git/linux-vm-provisioner/skills/provision-cloud-vm/runtime/providers/aws.yaml) instead of hardcoding cloud identifiers in scripts
- Keep scripts small and composable
- Preserve the distinction between repo-local templates and remote runtime state
- When rerunning only part of the Ansible bootstrap, edit [main.yml](/Users/lee/Projects/git/linux-vm-provisioner/skills/provision-cloud-vm/runtime/ansible/roles/common/tasks/main.yml) to comment out already-completed steps, then rerun the normal [setup](/Users/lee/Projects/git/linux-vm-provisioner/skills/provision-cloud-vm/runtime/ansible/setup) entrypoint. Do not use ad hoc temporary playbooks that import role task files directly.

---
> Source: [sloppycoder/linux-vm-provisioner](https://github.com/sloppycoder/linux-vm-provisioner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
