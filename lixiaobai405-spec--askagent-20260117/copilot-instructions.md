## askagent-20260117

> - This repository contains the Property Strategy Agent Streamlit application.

# AskAgent Project Instructions

## Scope

- This repository contains the Property Strategy Agent Streamlit application.
- Preserve the M1-M6 workflow and the existing JSON/Markdown persistence format.
- Make the smallest necessary change and avoid unrelated refactors.

## Environment

- Python requirement: 3.10 or newer.
- Local verified Conda environment: `askagent-20260117` with Python 3.12.
- Local Web entrypoint: `src/frontend/app.py`.
- CLI entrypoint: `main.py`.
- Install normal development requirements with `requirements.txt`.
- Use `requirements.deploy.txt` for the verified server dependency versions.

## Secrets and runtime data

- Never commit or print `.env`, API keys, tokens, credentials, or private keys.
- Use `.env.example` to document variable names and safe example values.
- Do not commit files under `data/sessions/` or `data/output/`.
- Treat generated reports, logs, prompts, responses, and uploaded project data as sensitive.
- Real model calls may incur cost; obtain explicit approval before running them.

## Verification

- Compile Python sources:

  ```bash
  python -m compileall -q main.py src
  ```

- On Windows, verify the launcher:

  ```powershell
  & .\tests\start-bat.Tests.ps1
  ```

- For Web changes, start Streamlit in the project Conda environment and verify the affected flow.
- Do not claim a real API workflow passed unless it was executed in the current verification run.

## Deployment

- Server application directory: `/opt/apps/askagent`.
- Persistent runtime data directory: `/opt/apps/askagent-data`.
- Conda prefix: `/opt/conda-envs/askagent`.
- systemd service: `askagent.service`.
- Streamlit listens only on `127.0.0.1:8501`.
- The isolated Nginx public endpoint listens on HTTP port `8502`.
- Do not modify, stop, restart, or overwrite the existing `demo3`, `teacher`, `time`, or
  `gzmat.cn` applications or their configurations.
- Before changing systemd, Nginx, secrets, or deployment directories, perform read-only
  inspection, explain the impact and rollback path, and obtain user confirmation.

---
> Source: [lixiaobai405-spec/askAgent_20260117](https://github.com/lixiaobai405-spec/askAgent_20260117) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
