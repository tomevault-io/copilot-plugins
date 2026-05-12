## videopipeline

> - `VideoPipeline` exists for the OpenClaw bot `Gondull` to oversee and manage.

# VideoPipeline Agent Context

## Project Relationship

- `VideoPipeline` exists for the OpenClaw bot `Gondull` to oversee and manage.
- Gondull's OpenClaw home is `/home/sindri/.openclaw` inside WSL.
- From Windows, that same location is `\\wsl.localhost\Ubuntu\home\sindri\.openclaw`.
- Gondull is used through the companion app at `/mnt/c/Users/gsind/Projects/Gondull_Platform`.
- The intended primary control plane is `Gondull_Platform` + `Gondull`/OpenClaw, not direct human-first use of VideoPipeline Studio.
- `VideoPipeline` should be treated as a pipeline service/tooling backend that can still be run locally, but the priority integration path is via `Gondull_Platform`.
- Prefer `/api/actions/*` and other backend interfaces over manual Studio-first workflows when implementing the primary architecture.
- Default orchestrated profile is `profiles/gaming_assemblyai.yaml` unless explicitly overridden.
- In that profile, AssemblyAI is the intended heavy-work backend for speech/audio intelligence.
- `Gondull` is the intended LLM operator for semantic scoring, review, chapter labeling, director picks, and other LLM-driven pipeline tasks via `gondull` / `external_strict` mode.
- When `Gondull` is asked to operate `VideoPipeline` end-to-end, prefer the OpenClaw skill `videopipeline-operator` at `/home/sindri/.openclaw/workspace/skills/videopipeline-operator/SKILL.md`.
- Direct Studio use is secondary and mainly for manual review, debugging, and fallback operation.
- When integration work spans both codebases, it is acceptable to update both `VideoPipeline` and `Gondull_Platform` together.
- If work also touches `Gondull_Platform`, treat it as incomplete until `npm run app:refresh:installed` and `npm run deploy:up` have been run there, then verify the installed desktop app and localhost runtime are in sync and `http://localhost:3000/login` returns HTTP 200.
- Be aware that `Gondull_Platform` `deploy:up` can sometimes exit nonzero because of a Docker recreate race even when the rebuild succeeded; check actual container/runtime health before treating the deploy as failed.
- See `docs/gondull-control-plane.md` for the intended operating model.

## Cross-Machine Context

- This project has been worked on from both the main PC and a laptop.
- The main PC is the primary target environment and should be optimized first.
- Laptop support is best-effort only and should not block improvements for the main PC.
- The two machines have had different local/runtime quirks and may require different settings or workarounds.
- Do not remove or "clean up" machine-specific Windows fixes just because they are not reproducible on the current machine, unless laptop support is being intentionally dropped.
- Prefer environment-scoped settings, launcher/runtime detection, or machine-specific overrides over changing shared defaults when a fix is only validated on one machine.
- Be especially careful around `launcher.py`, `VideoPipeline.spec`, subprocess/console behavior, GPU/runtime startup, and FFmpeg path discovery.
- A known example is Windows `cmd` / console freezing or flashing behavior that appeared on the laptop but not on the main PC.
- See `docs/machine-notes.md` for tracked cross-machine behavior and guardrails.

## Commit Workflow Preference

- For coding work in this repo, prefer finishing one coherent job, running the narrow relevant validation, then committing and pushing instead of stopping at local unpushed completion.
- Treat one finished job as one coherent commit when practical.
- Push to the appropriate branch for the repo state. If `main` is protected or the work is still checkpoint-grade, push to a feature/PR branch instead of forcing partial work onto `main`.
- Do not push broken partial states just to create a checkpoint.
- Do not bundle unrelated dirty-worktree changes into the same commit. Stage and commit only the files that belong to the finished job.
- When integration work spans both `VideoPipeline` and `Gondull_Platform`, prefer one coherent commit per repo rather than mixing unrelated repo state together.

## Secrets and Auth

- This repo uses Infisical as the default secret source when `.infisical.json` is present.
- Do not ask the user to paste raw secret values into chat when a local terminal prompt or file import can be used instead.
- Prefer `cx` over `codex` for new interactive Codex sessions in this repo so repo secrets are available automatically.
- Prefer `with-secrets <command>` for commands that require repo secrets.
- To add a new secret safely, use `infisical-secret-put SECRET_NAME` or `infisical-secret-put SECRET_NAME --from-file /path/to/file`.
- For one-time migration from an existing local env file, prefer `infisical secrets set --file .env.local` or a similar explicit env file import.
- Prefer `infisical login`; if the WSL browser callback is unreliable, use `infisical login -i`.
- On this machine, prefer Infisical Cloud EU unless the user or repo explicitly says otherwise.
- Detailed machine workflow reference: `/home/sindri/.config/codex/README.md`

---
> Source: [gsindri/VideoPipeline](https://github.com/gsindri/VideoPipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
