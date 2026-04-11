## ev3ps4controlled

> - EV3 robot controlled by a PS4 controller using Pybricks.

# AGENTS.md

## Project overview
- EV3 robot controlled by a PS4 controller using Pybricks.
- Hardware access goes through `DeviceManager` for safe/optional device handling.
- Local development can run without an EV3 by keeping hardware calls mocked or guarded.

## Setup commands
- Python 3.10+ recommended.
- python -m venv .venv
- source .venv/bin/activate
- pip install -r requirements-test.txt

## Dev environment tips
- Keep hardware-specific imports/usage behind guards so code can run on a laptop.
- Prefer dependency injection for sensors/motors to simplify mocking in tests.
- Use `pytest -q` for quick feedback; full run uses coverage and writes to `htmlcov/`.
- When iterating on EV3 code, commit small changes and keep logs concise for on-brick debugging.

## Testing instructions
- Run from repo root: `pytest`
  - Uses settings in `pytest.ini`; coverage reports to `htmlcov/` and XML.
- To target a subset: `pytest tests/test_<area>.py -k "<pattern>"`.
- If hardware is unavailable, mock device classes to avoid import/runtime errors.
- Keep coverage healthy; add/adjust tests when touching controllers, device manager, or safety logic.

## Code style
- Follow PEP 8; prefer descriptive names and small functions.
- Add type hints where practical; keep public APIs documented with docstrings.
- Centralize hardware interactions in `DeviceManager` and avoid duplicated port mappings.
- Default to standard library/testing tools; avoid adding new deps unless necessary.

## PR instructions
- Ensure `pytest` passes before opening/merging.
- Keep PRs scoped; document hardware assumptions/limitations in the description.
- Use imperative, concise titles (e.g., "Add gyro fallback handling").

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pentagram-Software)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Pentagram-Software)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
