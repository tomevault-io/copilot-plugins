## ecashai

> - Product and system architecture: `ARCHITECTURE.md`

# Project Map

- Product and system architecture: `ARCHITECTURE.md`
- Agent workflow and rollback harness: `docs/HARNESS.md`
- Engineering principles: `docs/ENGINEERING.md`
- Verification contract: `docs/TESTING.md`
- Work planning and git workflow: `docs/WORKFLOWS.md`

# Standard Commands

- Install dependencies: `pnpm install`
- Dev server: `pnpm dev`
- Harness check: `pnpm verify:harness`
- Lint: `pnpm lint`
- Unit tests: `pnpm test:unit`
- Build: `pnpm build`
- Default verification: `pnpm verify`
- Full verification: `pnpm verify:full`

# Working Rules

- Read the relevant doc before broad changes.
- Keep changes small enough to review and revert.
- Create a git checkpoint after each meaningful step.
- Prefer repository scripts over ad hoc commands.
- Add or update tests when behavior changes.
- Treat architecture, pricing, billing, auth, and agent isolation changes as high-risk.
- Never store or request user mnemonics server-side.
- Do not read or edit `.env*`, `secrets/**`, dependency directories, or generated output.
- Use multiple agents whenever you judge the work can be split cleanly. This is recommended to speed up research, implementation, and verification. Prefer parallel explorers for independent codebase questions and parallel workers for disjoint implementation slices. Keep ownership clear, integrate their results, and avoid duplicating the same investigation.
- Do not introduce policies that prefer third-party site scraping/crawling tools over Hermes capabilities, or policies that limit tool-call counts, when those policies conflict with user requirements. Use billing, permissions, quotas, and sandboxing to control risk instead.
- Do not bypass or reinterpret an agreed direction because of your own architectural preference. If a safety or scalability concern suggests a different architecture, explain the tradeoff and wait for explicit approval before adding services, changing runtime boundaries, changing tool policy, or replacing the agreed KISS path.

# Current Product Requirements

- This repository is a multi-user web product that calls Hermes Agent on behalf of hosted users. Treat every user prompt as untrusted input.
- The current MVP uses one app-managed Hermes gateway with Hermes' official Docker terminal backend. Keep the boundary simple: Hermes must run with its own `HERMES_HOME`, temp, cache, and generated state under `.data/hermes-runtime` or `/opt/data`.
- Hermes filesystem/process access is a product boundary, not a prompt instruction. Hosted/API Hermes may expose built-in `terminal` and `file` only when `terminal.backend=docker`, `docker_mount_cwd_to_workspace=false`, `docker_volumes=[]`, `docker_forward_env=[]`, and `container_persistent=false`.
- The Docker socket may be mounted only into the trusted Hermes service so Hermes can create its official tool sandbox containers. Do not mount `/var/run/docker.sock` into `web`, do not expose Docker CLI/API as a user tool, and do not let prompts pass raw Docker API payloads or arbitrary host bind mounts.
- Users must not be able to make Hermes or sandbox task containers inspect or modify VPS host files, operator home files, `.env` secrets, other users' uploads, other users' generated files, or the app repository unless that exact path was deliberately granted for the task.
- User uploads/private files stay outside Hermes by default. If a task needs files, copy or grant only the minimum scoped files into a per-run sandbox workspace and clean temporary/generated files by lifecycle policy.
- Hermes browser availability depends on the Hermes `agent-browser/local` provider. Plain Playwright installation or passing Playwright tests does not prove the Hermes browser tool is available.
- Configure Hermes to use happyclaw `gpt-5.5` as the primary model with output token cap `4096`. OpenRouter `deepseek/deepseek-v4-pro` is the fallback model and should keep `provider_routing.only/order: [deepseek]` when OpenRouter is active. The Next app must route chat through Hermes, not call model providers directly for chat.
- Product Hermes must run with `HERMES_HOME`, cwd, temp, cache, and generated files under `.data/hermes-runtime`.
- VPS deployment uses Docker Compose with `web` and `hermes` services. Use the official `nousresearch/hermes-agent:latest` image plus the product bootstrap script; do not depend on a manual `/downloads/hermes-agent` folder on the server.
- Do not reintroduce long-running per-user Docker, `bwrap`, `sandbox-exec`, or a separate sandbox service as the default path unless the product explicitly graduates from this KISS MVP boundary. Short-lived tool containers must be created by Hermes' official Docker backend.
- Production multi-user deployments must keep user-private files outside Hermes by default, grant scoped copies into per-run sandbox workspaces only when needed, and clean generated temp files by lifecycle policy.
- Current production chat may use `HERMES_ISOLATION_MODE=hermes-managed` for the single app-managed Hermes runtime, or `external-managed` for an upstream Hermes gateway. If hosted users can access terminal/file tools, the Hermes API server must pass `session_id` as `task_id` so tool workspaces do not share the upstream default task.

# Engineering Principles

- DIY where the implementation is core product knowledge, security-sensitive, or small enough to own.
- KISS for the first working path: solve the narrow case, encode the invariant, then generalize only when real duplication appears.
- Boring technology beats clever glue for agent-maintained systems.
- The repository is the durable memory. Decisions that should survive chat context belong in docs or tests.

# Definition Of Done

- The relevant docs are updated when behavior, workflow, or architecture changes.
- The default verification command passes, or the final response explains why it could not run.
- Important steps are committed with focused messages.
- New risks, open questions, and rollback points are visible in the final summary.

---
> Source: [alitayin/eCashAI](https://github.com/alitayin/eCashAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
