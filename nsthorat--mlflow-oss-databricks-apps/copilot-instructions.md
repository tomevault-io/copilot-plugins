## mlflow-oss-databricks-apps

> This project deploys the open-source MLflow server on Databricks Apps, providing a fully integrated MLflow experience with Databricks tracking backend and Unity Catalog model registry.

# MLflow OSS on Databricks Apps - Deployment Guide

## Project Overview

This project deploys the open-source MLflow server on Databricks Apps, providing a fully integrated MLflow experience with Databricks tracking backend and Unity Catalog model registry.

## What This Does

- Runs MLflow server connected to your Databricks workspace
- Uses Databricks for experiment tracking (no local database)
- Uses Unity Catalog for model registry
- Stores artifacts in Unity Catalog volumes
- Provides the full MLflow UI accessible through your browser
- Automatically downloads and serves MLflow UI assets during deployment

## Tech Stack

- **MLflow Server**: Open source MLflow with Databricks integration
- **Python Package Management**: `uv` for fast, reliable dependency management
- **Deployment Platform**: Databricks Apps (serverless hosting)
- **Backend Storage**: Databricks tracking store
- **Model Registry**: Unity Catalog
- **Artifact Storage**: Unity Catalog volumes (`/Volumes/main/default/mlflow-artifacts`)

## Changing MLflow Version

### Using Official MLflow Release (from PyPI)
To use the latest official MLflow release, update `pyproject.toml`:
```toml
dependencies = [
    "mlflow>=3.0.0",  # Uses latest MLflow 3.x from PyPI
    "databricks-sdk>=0.33.0",
    "gunicorn>=22.0.0",
]
```

### Using Custom MLflow Branch (from GitHub)
To use a specific MLflow branch or fork, update `pyproject.toml`:
```toml
dependencies = [
    "mlflow @ git+https://github.com/USERNAME/mlflow.git@BRANCH_NAME",
    "databricks-sdk>=0.33.0",
    "gunicorn>=22.0.0",
]
```

For example, to use the `nik-databricks-tracking` branch from `nsthorat/mlflow`:
```toml
dependencies = [
    "mlflow @ git+https://github.com/nsthorat/mlflow.git@nik-databricks-tracking",
    "databricks-sdk>=0.33.0",
    "gunicorn>=22.0.0",
]
```

After changing the MLflow version, run:
```bash
# Regenerate requirements.txt
uv run python scripts/generate_semver_requirements.py

# Deploy the changes
./deploy.sh
```

## Key Files

- `pyproject.toml` - Python dependencies including MLflow version
- `app.yaml` - Databricks Apps configuration (auto-generated, don't edit directly)
- `start_mlflow.sh` - Startup script that copies UI assets and starts MLflow server
- `deploy.sh` - Deployment script that handles UI assets and uploads to Databricks
- `requirements.txt` - Auto-generated from pyproject.toml, used by Databricks Apps
- `.env.local` - Local environment configuration (Databricks credentials)

## How It Works

1. **Deployment Process** (`./deploy.sh`):
   - Authenticates with Databricks using credentials from `.env.local`
   - Generates `requirements.txt` from `pyproject.toml` (preserves Git URLs)
   - Downloads latest MLflow UI assets from PyPI
   - Removes unnecessary files (source maps, licenses) to meet size limits
   - Syncs all files to Databricks workspace
   - Deploys application to Databricks Apps

2. **Startup Process** (`start_mlflow.sh`):
   - Copies MLflow UI assets to MLflow's expected location (`mlflow/server/js/build`)
   - Starts MLflow server with Databricks backend configuration:
     - `--backend-store-uri databricks` (use Databricks for tracking)
     - `--registry-store-uri databricks-uc` (use Unity Catalog for model registry)
     - `--default-artifact-root /Volumes/main/default/mlflow-artifacts` (UC volumes for artifacts)

3. **Runtime**:
   - MLflow server runs on port 8000
   - Authenticates using Databricks service principal (automatic in Databricks Apps)
   - Serves UI and API endpoints
   - All experiments, runs, and models are stored in Databricks

## Deployment Commands

### 🚨 CRITICAL: ALWAYS Run Deploy with nohup 🚨

**Deployment can take 5-10+ minutes. You MUST ALWAYS run deploy with nohup to a log file:**

```bash
# ✅ CORRECT - ALWAYS use nohup with logging
nohup ./deploy.sh > /tmp/mlflow-deploy.log 2>&1 &

# Monitor the deployment progress
tail -f /tmp/mlflow-deploy.log

# ❌ WRONG - Never run deploy directly
./deploy.sh  # DON'T DO THIS - it will timeout or disconnect
```

**Why this is critical:**
- Deployment involves building UI assets which takes several minutes
- Direct execution may timeout or disconnect, leaving deployment in unknown state
- Log file allows monitoring progress and debugging issues
- Background execution ensures deployment completes even if terminal disconnects

### Other Deployment Commands

```bash
# Initial setup (one time)
./setup.sh

# Check deployment status
databricks apps list

# Get app details
source .env.local && export DATABRICKS_HOST && export DATABRICKS_TOKEN
databricks apps get mlflow-oss
```

**Note**: The long deployment time is because `build_mlflow_ui_assets.sh` needs to run `yarn build` to compile the MLflow UI assets when using a Git branch, which can take 5+ minutes.

## Monitoring Application Logs

### Real-time Log Streaming

**Use the `dba_logz.py` script to stream application logs in real-time:**

```bash
# Stream all logs for 30 seconds
uv run python dba_logz.py https://YOUR-APP-URL --duration 30

# Search for specific terms in logs (e.g., errors)
uv run python dba_logz.py https://YOUR-APP-URL --search "ERROR\|Exception" --duration 60

# Monitor startup messages
uv run python dba_logz.py https://YOUR-APP-URL --search "Application startup\|Uvicorn running" --duration 60

# Continuous streaming (Ctrl+C to stop)
uv run python dba_logz.py https://YOUR-APP-URL --duration 999999
```

**Script options:**
- `app_url`: Base URL of your Databricks app
- `--search`: Filter logs with search query (supports regex)
- `--duration`: How long to stream logs in seconds (default: runs until interrupted)

**Note:** Browser-based logs are available at `https://YOUR-APP-URL/logz` but require OAuth authentication.

## Testing the Deployment

```bash
# Test health endpoint
uv run python dba_client.py https://YOUR-APP-URL/health

# Test experiments API
uv run python dba_client.py https://YOUR-APP-URL/api/2.0/mlflow/experiments/search POST '{}'

# Check UI is served
uv run python dba_client.py https://YOUR-APP-URL/
```

## Environment Variables

The following are automatically set by Databricks Apps:
- `DATABRICKS_HOST` - Your Databricks workspace URL
- `DATABRICKS_TOKEN` - Service principal token for authentication

## Troubleshooting

### UI Not Loading
- Check that `start_mlflow.sh` successfully copies UI assets
- Verify MLflow version in `pyproject.toml` matches UI assets version
- Check logs at `https://YOUR-APP-URL/logz` (requires browser authentication)

### Experiments Not Showing
- Verify `--backend-store-uri databricks` is set in `start_mlflow.sh`
- Check that service principal has proper permissions in Databricks workspace
- Ensure `DATABRICKS_HOST` and `DATABRICKS_TOKEN` are set

### Model Registry Issues
- Verify `--registry-store-uri databricks-uc` is set in `start_mlflow.sh`
- Check Unity Catalog permissions for the service principal
- Ensure Unity Catalog is enabled in your workspace

## Important Notes

- **Always use `uv`**: Never run `python` directly, always use `uv run python`
- **Git URLs in requirements**: The `generate_semver_requirements.py` script preserves Git URLs from pyproject.toml
- **UI Assets**: Automatically downloaded from PyPI during deployment, no need to commit them
- **Service Principal**: Databricks Apps automatically creates and manages the service principal
- **Authentication**: The app requires OAuth authentication to access (handled by Databricks Apps)

## Python Execution Rule

**NEVER run `python` directly - ALWAYS use `uv run`:**

```bash
# ✅ CORRECT - Always use uv run
uv run python script.py
uv run python dba_client.py

# ❌ WRONG - Never use python directly
python script.py
python dba_client.py
```

## Databricks CLI Usage

```bash
# With environment variables (required for apps commands)
source .env.local && export DATABRICKS_HOST && export DATABRICKS_TOKEN
databricks apps list
databricks apps get mlflow-oss
databricks current-user me
```

Remember: This deployment provides full MLflow functionality with Databricks as the backend, eliminating the need for local storage or databases.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nsthorat)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nsthorat)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
