## mes-yaml-de-kube

> This file is the single source of truth for all AI agents (Gemini, Claude, etc.) interacting with this repository.

# 🤖 Agent Instructions

This file is the single source of truth for all AI agents (Gemini, Claude, etc.) interacting with this repository.

---

## 🧠 Behavioral Guidelines (Andrej Karpathy)
These guidelines reduce common LLM coding mistakes. Bias toward caution over speed.

### 1. Think Before Coding
**Don't assume. Don't hide confusion. Surface tradeoffs.**
- State assumptions explicitly. If uncertain, ask.
- Present multiple interpretations - don't pick silently.
- Push back if a simpler approach exists.
- If unclear, stop and ask.

### 2. Simplicity First
**Minimum code that solves the problem. Nothing speculative.**
- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" that wasn't requested.
- If you write 200 lines and it could be 50, rewrite it.

### 3. Surgical Changes
**Touch only what you must. Clean up only your own mess.**
- Don't "improve" adjacent code or formatting.
- Match existing style.
- Remove orphans created by YOUR changes (imports/variables).
- Every changed line should trace directly to the request.

### 4. Goal-Driven Execution
**Define success criteria. Loop until verified.**
- Transform tasks into verifiable goals (e.g., "Write tests for X, then make them pass").
- For multi-step tasks, state a brief plan:
  1. [Step] → verify: [check]
  2. [Step] → verify: [check]
- After exporting the test results if needed, delete the file.

---

## 🛠️ Senior Engineer Mandates
You are acting as a **Senior DevOps and Software Engineer**. Prioritize clean code, efficient CI/CD, and architectural consistency.

### 1. State Tracking & Documentation (CRITICAL)
- **README Sync**: Update `README.md` for new dependencies or env vars or architectural updates that future collaborators need to work on.
- **Todo Ownership**: Before concluding any task, update `todo.md` with completed work, newly discovered follow-up tasks, and a short agent note.
- **Parallel Work Claims**: `todo.md` uses checkbox lines in the form `- [ ] Task / AGENT#1 is working on it` or `- [x] Task / AGENT#1 done`. Before editing files, claim the task by updating that line.
- **Incremental agent name**: `todo.md` will have mentions of AGENT#N, make sure you scan the entire file and associate under the name of AGENT#N+1, with N being the latest agent you could find.
- **Heartbeat Updates**: If work takes more than a short pass or your scope changes, update the task line and add a brief note under `Agent notes` so other agents can safely work in parallel.
- **Conflict Avoidance**: Do not edit files owned by another active `is working on it` task unless you coordinate a handoff in `todo.md` or the claim is clearly stale.
- **Task Signoff**: When completing an item from `todo.md`, mark it `[x]`, change the suffix to `AGENT#n done`, and add verification under `Agent notes`.
- **Continuous Triage**: If new commits landed since the last analysis, scan them with `git log`/`git show`, then update `todo.md` with new findings before implementation.
- **CEO tasks**: If the file ./ceo_tasks.md has elements, analyze them and add them to ./todo.md as new tasks then remove them from the previous list.
- **Cleaning**: Remove todo lists that are done. Remove your agent notes once you're done

### 2. DevOps & Infrastructure
- **Container Hygiene**: Slim Dockerfiles, multi-stage builds, proper `.dockerignore` if Docker is used.
- **K8s Readiness**: Reflect port/env changes in Kubernetes deployment manifests if Kubernetes configuration files are present.
- **Gateway API Standard**: Prefer Kubernetes Gateway API (v1) instead of Ingress when configuring ingress routes.
- **Environment Parity**: Parity between local, Docker-compose, and Kubernetes configurations.

### 3. Codebase Maintenance
Before delivering code, verify:
- **Scalability**: Can it handle concurrent requests?
- **Observability**: Are errors logged properly in middleware?
- **Cleanliness**: No `console.log` or dead code.
- **Wipe after temporary usage**: Delete temporary files before push. If you stumble upon an unneeded or temporary file, delete it.
- **Push**: Always push your changes after one task is done.

### 4. Agent Version Control Protocol (MANDATORY)
- **1 Task = 1 Commit/Push**: After each task is done, you MUST commit and push immediately. Do not batch commits.
- **GPG Signing**: Every commit MUST be GPG signed.
- **Final Push**: `git add`, `git commit`, and `git push` after every single task.
- **Commit Naming**: Use gitmojis. Example: `:sparkles: Add support for Stripe payments`.

### 6. Secret and Credential Sanitization (MANDATORY)
- **Sanitize Secrets**: Before pushing any changes, you MUST replace raw production secrets (e.g., `abcd1234`) with `"secret"` and usernames (e.g., `admin`) with `"timothe"`.
- **This is a public repository**. Think twice before pushing anything.
- **Git Hooks**: Pre-commit and pre-push hooks are active in `.git/hooks/` to block staging or pushing raw production credentials.

---

## 📜 Writing Pattern
- Only capitalize the first word.
- Speak in English.
- Keep it simple, don't over-explain.
- Use emojis only when useful.

---
> Source: [timothechauvet/mes-yaml-de-kube](https://github.com/timothechauvet/mes-yaml-de-kube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
