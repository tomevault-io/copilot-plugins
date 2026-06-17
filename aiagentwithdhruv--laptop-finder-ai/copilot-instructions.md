## 90-devops-deployment

> Docker, CI/CD, AWS, Vercel, and VPS deployment rules


Deployment targets:
- Docker for packaging.
- AWS (ECS Fargate, Lambda, S3) for scalable cloud deployment.
- Vercel for frontend and serverless Next.js deployments.
- VPS (Ubuntu/Debian) for cost-effective long-running services.
- CI/CD for validation and release automation.

DevOps rules:
- Applications must be container-friendly.
- Keep builds reproducible.
- Use environment-driven configuration.
- Separate build-time config from runtime config.
- Add health checks where appropriate.
- Keep services stateless unless statefulness is intentional and documented.
- Ensure logs are structured and production-usable.

Docker expectations:
- Keep Dockerfiles clean and optimized.
- Prefer multi-stage builds when useful.
- Avoid baking secrets into images.
- Minimize image size where practical.
- Use .dockerignore to exclude node_modules, .env, .git, checkpoints.

AWS expectations:
- Prefer managed services where appropriate.
- Use least-privilege IAM policies.
- Make networking and environment assumptions explicit.
- Support observability via logs, metrics, and alarms.
- Design for rollback-safe deployments.

Vercel expectations:
- Use vercel.json for project configuration.
- Set environment variables via Vercel dashboard, never in code.
- Use Edge/Serverless functions for API routes.
- Keep build output under Vercel size limits.
- Use preview deployments for PR review.
- Configure custom domains and redirects in vercel.json.
- Use Vercel Analytics and Speed Insights where available.

VPS expectations:
- Use systemd or PM2 for process management.
- Set up Nginx/Caddy as reverse proxy with SSL (Let's Encrypt).
- Use UFW or iptables for firewall — only expose necessary ports.
- Set up unattended-upgrades for security patches.
- Use SSH keys only — disable password auth.
- Keep deployment scripted and repeatable (shell scripts or Ansible).
- Monitor with basic tooling: htop, journalctl, logrotate.

CI/CD expectations:
- Run lint, tests, and build validation before merge/deploy.
- Prevent obviously broken releases.
- Keep deployment config version-controlled.
- Use separate environments where needed, e.g. dev/staging/production.
- Use GitHub Actions for CI, Vercel for frontend CD, and scripts for VPS CD.

Do not:
- Hardcode AWS-specific IDs, Vercel tokens, or VPS IPs in code.
- Mix deployment-only hacks into application logic.
- Assume local-only paths/config in production code.
- Run services as root on VPS.
- Expose database ports publicly on any platform.

---
> Source: [aiagentwithdhruv/laptop-finder-ai](https://github.com/aiagentwithdhruv/laptop-finder-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
