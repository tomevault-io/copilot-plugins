## gpt-agreement-payment

> This repository centers on two Python subprojects:

# Repository Guidelines

## Project Structure & Module Organization
This repository centers on two Python subprojects:

- `CTF-pay/`: main Stripe Checkout replay tooling. Key entrypoints are `card.py`, `hcaptcha_auto_solver.py`, `hcaptcha_bridge_helper.py`, `local_mock_gateway.py`, and `retry_house_decline.py`.
- `CTF-reg/`: account registration and auth helpers used by `CTF-pay` auto-registration flows (`auth_flow.py`, `sentinel.py`, `http_client.py`, `config.py`).
- `flows.gz` and related `flows*` artifacts: mitmproxy captures used as protocol ground truth. Treat them as read-only inputs.
- `CTF-pay/test_runs/`: saved replay outputs and comparison logs.

## Build, Test, and Development Commands
Use Python directly; no Makefile or package manager wrapper is defined.

- `python CTF-pay/card.py auto --config CTF-pay/config.auto.json` — run the default end-to-end checkout replay.
- `python CTF-pay/card.py fresh --fresh-only --config CTF-pay/config.auto.json` — generate a fresh checkout only.
- `python CTF-pay/card.py auto --config CTF-pay/config.offline-replay.json --offline-replay` — validate logic without external requests.
- `python CTF-pay/card.py auto --config CTF-pay/config.local-mock.json --local-mock` — exercise the local mock gateway.
- `python CTF-pay/hcaptcha_bridge_helper.py http://127.0.0.1:PORT/index.html` — debug bridge pages interactively.
- `~/.venvs/ctfml/bin/python CTF-pay/hcaptcha_auto_solver.py <bridge-url>` — run the visual solver.

## Coding Style & Naming Conventions
Follow existing Python style:

- 4-space indentation, no tabs.
- `snake_case` for functions, variables, and JSON keys; `PascalCase` for dataclasses/exceptions.
- Keep new config files in the existing `config.*.json` pattern.
- Prefer small helper functions over adding more inline logic to already large scripts such as `CTF-pay/card.py`.
- No formatter config is checked in; match surrounding style and keep imports grouped logically.

## Testing Guidelines
There is no dedicated `tests/` suite today. Validate changes with replay-based checks:

- Prefer `--offline-replay` or `--local-mock` before live runs.
- Review `CTF-pay/log.txt` and save reproducible outputs under `CTF-pay/test_runs/`.
- For new automated tests, use `pytest` with filenames like `test_*.py` and keep fixtures small.

## Commit & Pull Request Guidelines
Git history is currently minimal (`Initial import of Gpt-Agreement-Payment project`), so use clear imperative commit subjects, e.g. `Add local mock retry logging`.

PRs should include:

- purpose and affected flow (`fresh`, `auto`, `local-mock`, solver, or auth);
- exact command(s) used for verification;
- sanitized logs or screenshots when UI/challenge behavior changes;
- notes on any config or secret handling changes.

## Security & Configuration Tips
Configs may contain tokens, cookies, mail credentials, and proxy settings. Do not commit real secrets. Keep `flows*` artifacts immutable, redact sensitive values in shared logs, and prefer copying sample configs over editing working credentials in place.

---
> Source: [DanOps-1/Gpt-Agreement-Payment](https://github.com/DanOps-1/Gpt-Agreement-Payment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
