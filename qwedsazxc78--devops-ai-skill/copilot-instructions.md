## devops-ai-skill

> @.gemini/agents/horus.md

# GEMINI.md (Google Gemini CLI)

@.gemini/agents/horus.md
@.gemini/agents/zeus.md

Read `docs/PROJECT.md` first — it contains all shared project context.

## Gemini Specifics

- Agent definitions: `.gemini/agents/horus.md` and `.gemini/agents/zeus.md`
- Extension config: `.gemini/extensions/devops/gemini-extension.json`
- Skills are in `skills/` directory — each has `SKILL.md` with workflow steps
- Use `run_shell_command` for bash execution

## Agent Selection

Detect the repository type to choose the right agent:

- For **Terraform, Helm, GKE** → follow Horus workflows from `prompts/horus/`
- For **Kustomize, ArgoCD, GitOps** → follow Zeus workflows from `prompts/zeus/`
- **Unsure** → run detection logic from `prompts/shared/repo-detect.md`

## Horus Commands (IaC — Terraform + Helm + GKE)

| Command | Pipeline |
|---------|----------|
| *full | Full check (RUNS CLI) + report |
| *upgrade | Upgrade Helm chart versions |
| *security | Security audit (file analysis) |
| *validate | Validation (fmt + file analysis) |
| *scaffold | Scaffold new Helm module |
| *cicd | Improve CI/CD pipeline |
| *health | Platform health check |

## Zeus Commands (GitOps — Kustomize + ArgoCD)

| Command | Pipeline |
|---------|----------|
| *full | Full pipeline + YAML/MD reports |
| *pre-merge | Pre-MR essential checks |
| *health | Repository health assessment |
| *review | MR review pipeline |
| *scaffold | Service scaffold (interactive) |
| *diagram | Generate architecture diagrams |
| *status | Tool installation check |
| *gateway-migrate | Migrate NGINX Ingress to Gateway API (default Traefik, opt-in GKE via `--gateway-class gke-l7-*`) — per-hostname DNS cutover |
| *nginx-to-traefik | `prompts/zeus/nginx-to-traefik.md` |
| *nginx-to-gateway | `prompts/zeus/nginx-to-gateway.md` |
| *ingress-to-gateway | Auto-detect source class (nginx/traefik) then migrate to Gateway API |
| *ingress-migration-advisor | Read-only EOL planner: scores services and recommends a migration path per service |
| *install-traefik | GitOps-flavored Traefik bootstrap/new-env/upgrade — edits common.traefik/, plan-only |
| *decommission-nginx | GitOps-flavored ingress-nginx decommission — archives module + ArgoCD prune, plan-only |
| *retire-nginx | Post-migration nginx retirement — controller app delete + `$patch: delete` base nginx Ingresses, per-env or all |
| *migration-quickstart | 30-second orientation: decision tree + 8-command table + sample invocations (no scan, no prompts) |

---
> Source: [qwedsazxc78/devops-ai-skill](https://github.com/qwedsazxc78/devops-ai-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
