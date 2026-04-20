## backtrader-bots

> - Python 3.13+ only. Use `pyproject.toml` for dependencies.




## Python Standards
- Python 3.13+ only. Use `pyproject.toml` for dependencies.
- Type hints required. Run pyright in `strict` mode.

## Project Structure
- Strategies go in `src/strategies/`, data feeds in `src/adapters/`, brokers in `src/broker/`.
- Keep strategy classes pure: no network calls or I/O.
- Centralize config in `config/*.yaml` and load via pydantic-settings.

## Trading Logic
- Use deterministic random seeds in backtests.
- No wall-clock functions (`datetime.now()`, `sleep`) inside strategies.
- All signals return +1 (long), -1 (short), or 0 (flat).

## Safety & Risk
- Never log secrets. Pull creds from `.env`.
- Implement a risk kill-switch in live mode.
- Unit tests required for indicators and signals.

## Documentation
- Every module has a `README.md` with usage examples.
- Public functions need short docstrings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/syuraj) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
