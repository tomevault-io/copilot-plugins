## ansible-collection-selfhosted

> Ansible collection of self-hosted app roles (`galaxy.yml`, `roles/*`).

# AGENTS.md — `artyorsh.selfhosted`

Ansible collection of self-hosted app roles (`galaxy.yml`, `roles/*`).

## Before editing

- Read the touched role’s `README.md`, `defaults/main.yml`, and `tasks/`.
- Prefer new behavior behind defaults; document variables in the role README.

## Validation

```bash
ansible-playbook .github/ansible/playbook-ci.yml --syntax-check
```

## Downstream impact

Breaking or behavior-changing edits to defaults, tags, or required variables should be documented (role README, changelog, or release notes) so anyone consuming this collection can adjust their playbooks and pins.

---
> Source: [artyorsh/ansible-collection-selfhosted](https://github.com/artyorsh/ansible-collection-selfhosted) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
