## homelab-k12n-gitops

> Always read and follow the instructions in `CLAUDE.md` - it contains the complete project documentation, coding standards, and operational procedures.

# Cursor AI Rules for Homelab K12n GitOps Project

## Primary Reference
Always read and follow the instructions in `CLAUDE.md` - it contains the complete project documentation, coding standards, and operational procedures.

## Kubernetes Operations

Use `kubectl` and `flux` CLI tools directly for all Kubernetes operations.

### Common Commands

**Restart deployment:**
```bash
kubectl rollout restart deployment/<app-name> -n <namespace>
kubectl rollout status deployment/<app-name> -n <namespace>
```

**View logs:**
```bash
kubectl logs -n <namespace> deployment/<app-name>
```

**FluxCD reconciliation:**
```bash
flux reconcile kustomization <app-name>
```

**Get resources:**
```bash
kubectl get pods -n <namespace>
kubectl describe deployment/<app-name> -n <namespace>
```

## Git Commit Format

**ALWAYS use HEREDOC for multi-line commit messages:**

```bash
git commit -m "$(cat <<'EOF'
type: description

Detailed explanation...

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"
```

## Code Style

- Rust: Run `cargo fmt` before committing
- TypeScript: No unused variables (lint will fail)
- Docker: ALWAYS use multi-stage builds with dependency caching
- Never use emojis unless explicitly requested
- Follow conventional commits format

## Architecture Rules

- **homelab-api**: READ-ONLY API - never add write operations (INSERT/UPDATE/DELETE)
- **Sealed Secrets**: User creates manually - provide kubectl + kubeseal command snippets only
- **Database migrations**: Use transaction for multi-step operations

## For More Details

Read `CLAUDE.md` for complete documentation including:
- Infrastructure overview
- Application architecture
- Deployment workflows
- Docker best practices
- Database patterns
- Authentication flow
- Recent changes

---
> Source: [bokristoffersson/homelab-k12n-gitops](https://github.com/bokristoffersson/homelab-k12n-gitops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
