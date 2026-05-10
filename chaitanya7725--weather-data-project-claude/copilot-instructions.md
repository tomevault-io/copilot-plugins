## weather-data-project-claude

> Validates an Open-Meteo API response dict. Raises `ValueError` if:

# Weather Data Fetcher

Python CLI tool that pulls weather forecast data from the Open-Meteo API
and exports it to CSV. No API key required.

## Tech stack

- Python 3.10+
- requests (HTTP client)
- pytest + pytest-mock + pytest-cov (testing)
- Open-Meteo API (free, no key required)

## Commands

- Install deps: `pip install -r requirements.txt`
- Run the script: `python src/weather_fetcher.py`
- Run all tests: `pytest tests/ -v` (coverage report printed automatically)
- Run a single test: `pytest tests/test_weather_fetcher.py::ClassName::test_name -v`
- View HTML coverage report: open `htmlcov/index.html` after running tests

## CLI arguments

| Argument | Default | Valid range | Description |
|---|---|---|---|
| `--latitude` | `30.2672` | −90 to 90 | Latitude in decimal degrees |
| `--longitude` | `-97.7431` | −180 to 180 | Longitude in decimal degrees |
| `--days` | `7` | 1–16 | Number of forecast days to fetch |
| `--output` | `weather_output.csv` | any path | Output CSV file path |

Invalid coordinates cause an immediate error and exit code 1 — no network call is made.

## Project structure

- `src/__init__.py` — makes `src` an importable package
- `src/weather_fetcher.py` — main module (see functions below)
- `tests/test_weather_fetcher.py` — unit tests (72 tests, ~97% coverage)
- `pytest.ini` — pytest config; sets `pythonpath`, enables coverage on every run
- `requirements.txt` — pinned dependencies
- `weather_output.csv` — generated output (gitignored)
- `htmlcov/` — HTML coverage report (gitignored)

## Public functions

### `validate_response(data: dict) -> None`
Validates an Open-Meteo API response dict. Raises `ValueError` if:
- `daily` key is missing
- any required field (`time`, `temperature_2m_max`, `temperature_2m_min`, `precipitation_sum`) is absent or empty
- any temperature value is outside −90 to 60°C or is null

### `fetch_weather(latitude, longitude, days=7, timeout=10) -> dict`
Fetches the forecast from the API. Validates coordinates first (raises
`ValueError` immediately without hitting the network if out of range).
Retries up to 3 times with exponential backoff (1s, 2s, 4s) on:
- `Timeout`
- `ConnectionError`
- HTTP 429 (respects `Retry-After` header if present)

Non-429 HTTP errors and JSON decode errors are raised immediately without retry.

### `export_to_csv(data: dict, filename="weather_output.csv") -> str`
Writes forecast data to CSV atomically (write to a `.tmp` file, then rename
into place) so a mid-write crash never leaves a partial file. Always UTF-8.
CSV columns: `date`, `temp_max`, `temp_min`, `precipitation`.

### `main() -> None`
CLI entry point. Parses arguments, calls `fetch_weather` and `export_to_csv`,
prints the output path on success, or prints the error and exits with code 1.

## Conventions

- Type hints on all function signatures
- Docstrings on all public functions
- Tests mock all external HTTP calls — never hit the real API in tests
- CSV output always includes headers: `date`, `temp_max`, `temp_min`, `precipitation`
- Secrets must never be hardcoded — use environment variables

## Known constraints

- Open-Meteo free tier: max 10,000 requests/day
- Script runs as a one-shot CLI, not a long-running service

---
> Source: [Chaitanya7725/weather-data-project-claude](https://github.com/Chaitanya7725/weather-data-project-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
