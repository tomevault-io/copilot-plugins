## backend-venv

> Backend environment setup and activation


# Backend Env Setup

* Venv location: `backend/.venv`
* If missing, create it:

```bash
cd backend
uv venv || python3 -m venv .venv
```

* Activate:

```bash
source .venv/bin/activate
```

* Install deps (also creates venv if needed):

```bash
uv sync --native-tls
```

* Use uv when installing dependencies.
* Add deps: `uv add <pkg>` • Deactivate: `deactivate`
  

---
> Source: [MylesThomas/betting](https://github.com/MylesThomas/betting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
