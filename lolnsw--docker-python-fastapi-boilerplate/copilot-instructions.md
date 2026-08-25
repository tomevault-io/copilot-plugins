## python-build

> Python Build, Env and Packages


When setting up the project:
- Do not use Poetry
- Do not use setup.py
- Do use pyproject.toml
- I have already created the python venv, never ask me to create it

Configuration
- The configuration of the app should be in config/config.yaml
- Secrets should be in .env
- The app should take the config/config.yaml by default. However it can be started with a file in another location



For tests:
- Only use pytest
- Do not use MagicMock or AsyncMock. If there no other choice but to use those 2 packages, ask me first.
- It has to have its own config file and .env located in ./tests/config.py and /tests/.env respectively


For DockerBuild:
- Needs to run on arm and amd64


Python package to use and not use:
- Use FastAPI for all API
- do not use Flask
- do not use Pydantic

---
> Source: [lolnsw/docker-python-fastapi-boilerplate](https://github.com/lolnsw/docker-python-fastapi-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
