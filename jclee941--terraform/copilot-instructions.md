## terraform

> **Generated:** 2026-07-01

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-07-01
**Commit:** c6a5636
**Branch:** master

## OVERVIEW
Homelab infrastructure-as-code monorepo for `jclee.me`. Terraform provisions Proxmox LXC/VM fleet resources, generated service configs, Cloudflare/GitHub-style external integrations, and validation tooling with 1Password-backed secrets and GitHub Actions CI/CD.

- **Domain:** `jclee.me`
- **Subnet:** `192.168.50.0/24`
- **Public access:** Native `cloudflared-homelab` on cliproxy (LXC 114), routing each hostname directly to a service IP:port
- **Terraform:** 1.10.5 (`>= 1.7, < 2.0`)
- **Workspaces:** numeric prefixes; Make aliases resolve nested `terraform/` roots where used
- **Module entry points:** 10 under `modules/{proxmox,shared,cloudflare,elasticstack}`

## STRUCTURE
```text
terraform/
├── 80-jclee/                 # Personal workstation skeleton
├── 100-pve/                  # Tier 0 Proxmox orchestrator and host SSoT
├── 105-elk/                  # Tier 1 ELK Terraform, templates, scripts
├── 200-oc/ 215-synology/ 220-youtube/
├── 300-cloudflare/           # Independent Cloudflare Terraform, secret inventory, scripts, Workers
├── 310-safetywallet/ 400-gcp/
└── modules/ tests/ scripts/ docs/ .github/
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Add or resize LXC/VM | `100-pve/terraform/locals.tf` + `100-pve/envs/prod/hosts.tf` | `hosts.tf` is the host/IP/VMID SSoT. |
| Change service config | `{NNN}-{svc}/templates/*.tftpl` | Rendered centrally by `100-pve`; never edit outputs. |
| New public route | `300-cloudflare/terraform/` | Direct hostname-to-service IP:port ingress for the Cloudflare Tunnel on cliproxy (114). |
| ELK pipeline/indexing | `105-elk/templates/logstash.conf.tftpl` + `modules/elasticstack/` | Keep ILM and auth assumptions intact. |
| Cloudflare DNS/tunnel/Workers | `300-cloudflare/terraform/` + `modules/cloudflare/` | Access resources are removed; scripts and Workers have separate child scopes. |
| Secret retrieval | `modules/shared/onepassword-secrets/` | Values come from 1Password, not committed files. |
| CI/CD or PR policy | `.github/AGENTS.md` + `.github/workflows/` | Many workflows delegate to `jclee941/.github`. |
| Test behavior | `tests/AGENTS.md` | Native `terraform test`; provider-mocked by default; Make targets run a subset. |
| Documentation policy | `docs/AGENTS.md` | ADRs append-only; runbooks actionable. |

## CODE MAP
| Area | Entry Points |
|------|--------------|
| Core fleet | `100-pve/terraform/main.tf`, `100-pve/envs/prod/hosts.tf` |
| Tier 1 apps | `105-elk/terraform/main.tf` |
| External providers | `215-synology/main.tf`, `300-cloudflare/terraform/main.tf` |
| Modules | `modules/proxmox/*/main.tf`, `modules/cloudflare/tunnel/main.tf`, `modules/elasticstack/*/main.tf` |
| Tooling | `scripts/validate-docs/main.go`, `scripts/audit-workflows.go`, `300-cloudflare/scripts/collect.go` |
| Workers | `300-cloudflare/workers/*/src/index.ts` |

## CONVENTIONS
- Active Terraform workspaces usually keep `.tf` files under `{workspace}/terraform/`; `215-synology/` is the flat exception.
- `make` aliases are the command contract: `pve`, `elk`, `synology`, `cloudflare`, `gcp`, etc.
- `snake_case` for Terraform identifiers; single-instance resources use `resource "x" "this"`.
- Templates and scripts use `kebab-case`; app logic belongs in `templates/*.tftpl`, not inline cloud-init.
- Variables and outputs need descriptions; variables need explicit types.
- Local backend state exists beside workspaces; `105-elk` and `100-pve/terraform` carry state/plan artifacts as explicit exceptions, not a pattern to copy.
- CI concurrency is the serialization mechanism.
- 1Password vault `homelab` feeds Terraform via the shared module and environment variables.

## ANTI-PATTERNS
- No local `make apply` or local drift check; deployment and drift detection are CI/CD paths.
- No hand-editing `100-pve/terraform/configs/`, rendered templates, or guest files managed by Terraform.
- No hardcoded service IPs; use `module.hosts.hosts[name].ip`, variables, or template inputs.
- No committed `.tfvars`, `.env`, API keys, tunnel tokens, private keys, or secret-bearing `data/` outputs.
- No mutable GitHub Action tags when adding or reviewing workflow dependencies; existing workflows still contain mixed tag pinning.
- No manual Proxmox UI changes for Terraform-managed guests.
- No direct resource duplication in workspaces when a local module owns the abstraction.

## COMMANDS
- Planning/validation: `make plan SVC=pve`, `make validate SVC=pve`, `make fmt`, `make lint`, `make lint-docs`
- Tests: `make test`, `make test-unit`, `make test-integration`, `make test-workspace`
- Ops/docs: `make verify`, `make backup`, `make setup`, `make docs`, `make security`

---
> Source: [jclee941/terraform](https://github.com/jclee941/terraform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
