## 216labs

> Auto-deploy single-app changes by default


# Single-App Auto Deploy

When a task primarily changes one app (for example one folder under `products/` or `internal/` for a single service):

1. Commit and push to `main` as usual.
2. Automatically run a targeted droplet rollout for that app in the same task, without waiting for a separate user prompt to deploy.
3. Default to fast single-app rollout settings:
   - `DEPLOY_RUNTIME_APPS="<app_id>"`
   - Use `DEPLOY_IMAGE_SOURCE=local` (local build + ssh transfer) by default for single-app commits.
4. Verify the public URL responds (`200` or expected redirect) and report the exact link.

Do not apply this auto-deploy behavior to clearly multi-app or infra-wide changes unless explicitly requested.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/6cubed)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/6cubed)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
