## shankar

> - Host machine: MacBook

# Project Context

## Environment
- Host machine: MacBook
- Dev environment: Virtualized (Multipass Ubuntu VM / Orbstack container)
- Do NOT assume access to host filesystem — all paths are inside the VM/container

## Stack
- Language: Python 3.x
- Environment manager: Conda
- Web framework: Flask
- ORM: SQLAlchemy
- Database: PostgreSQL
- Data: Pandas, NumPy
- Notebooks: Jupyter
- Test runner: pytest

## Common Commands
```bash
# Create / activate conda environment
conda create -n myenv python=3.11
conda activate myenv

# Install dependencies
conda install pandas numpy jupyter flask sqlalchemy psycopg2 

# Run Flask dev server
flask run
# or:
python app.py

# Launch Jupyter notebook
jupyter notebook
# or headless in VM:
jupyter notebook --no-browser --ip=0.0.0.0

# Run tests
pytest

# Freeze environment
conda env export > environment.yml
conda env create -f environment.yml
```

## Project Structure
```
/
├── app.py               # Flask entry point
├── models/              # SQLAlchemy models
├── routes/              # Flask blueprints / route handlers
├── notebooks/           # Jupyter notebooks (exploration / analysis)
├── data/                # Raw and processed data files
├── tests/
├── environment.yml      # Conda environment definition
└── ...
```

## Conventions
- Use snake_case for variables and functions, PascalCase for classes
- Type hints on all function signatures
- SQLAlchemy models live in /models — never write raw SQL unless absolutely necessary
- Notebooks are for exploration only — production logic goes into .py modules
- Pandas DataFrames should have explicit dtypes where possible
- Never commit large data files or .ipynb checkpoint folders

## Current Task / Work in Progress
<!-- Update this as you switch tasks — most useful for context transfer across devices -->
- Working on:
- Last left off at:
- Next step:

## Known Gotchas
- Jupyter in a VM needs --no-browser --ip=0.0.0.0 to be accessible from the host
- psycopg2 may need to be installed via conda-forge: conda install -c conda-forge psycopg2
- Do not use pip inside a conda environment unless the package is unavailable via conda

## AI Tool Notes
- Always use Python — never suggest JavaScript or Node.js alternatives
- This project runs in a VM — do not reference host Mac paths
- Prefer conda over pip for package management
- When in doubt, ask before making structural changes to models or DB schema

---
> Source: [RealAbhishek/shankar](https://github.com/RealAbhishek/shankar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-08 -->
