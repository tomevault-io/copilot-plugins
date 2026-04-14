## aws-terraform-atmos

> Conventions for Ansible playbooks configuring AKS clusters


# Ansible Playbook Conventions

- Target `localhost` with `connection: local` and `gather_facts: false`
- Use `kubernetes.core` collection for all K8s operations
- Pass `kubeconfig` path to every `kubernetes.core.k8s` task
- Variables flow from Atmos stacks via `vars:`
- Inline Kubernetes manifests as `definition:` blocks (no separate YAML files)
- Always set `state: present` explicitly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rykelley)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rykelley)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
