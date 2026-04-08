---
name: ansible
description: Run Ansible playbooks and manage AWX/Tower via API. Automate infrastructure configuration. Use when this capability is needed.
metadata:
  author: openclaw
---
# Ansible / AWX
Infrastructure automation.
## Environment
```bash
export AWX_URL="https://awx.example.com"
export AWX_TOKEN="xxxxxxxxxx"
```
## List Job Templates
```bash
curl "$AWX_URL/api/v2/job_templates/" -H "Authorization: Bearer $AWX_TOKEN"
```
## Launch Job
```bash
curl -X POST "$AWX_URL/api/v2/job_templates/{id}/launch/" \
  -H "Authorization: Bearer $AWX_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"extra_vars": {"host": "webserver"}}'
```
## Get Job Status
```bash
curl "$AWX_URL/api/v2/jobs/{jobId}/" -H "Authorization: Bearer $AWX_TOKEN"
```
## Run Ansible CLI
```bash
ansible-playbook -i inventory.yml playbook.yml
ansible all -m ping -i inventory.yml
```
## Links
- Docs: https://docs.ansible.com

---
> Converted and distributed by [TomeVault](https://tomevault.io) | [Claim this content](https://tomevault.io/claim/openclaw/skills)
<!-- tomevault:3.0:skill_md:2026-04-07 -->
