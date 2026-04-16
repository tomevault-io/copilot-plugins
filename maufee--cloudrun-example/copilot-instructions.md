## cloudrun-example

> - **Description:** A simple Python Flask web application created to serve as a "Hello World" example for deployment to Google Cloud Run.

# Gemini Context for cloudrun-example Project

## Project Overview
- **Description:** A simple Python Flask web application created to serve as a "Hello World" example for deployment to Google Cloud Run.
- **Status:** The application is complete and deployed, with a robust, secure, and automated CI/CD pipeline.

## CI/CD Pipeline
- **Platform:** GitHub Actions
- **Trigger:** Pushes to the `main` branch.
- **Jobs:**
    1.  `test-and-lint`: Runs `ruff` and `pytest` to ensure code quality.
    2.  `deploy`: On success, deploys the application to Google Cloud Run.
- **Authentication:** Uses Workload Identity Federation for secure, keyless authentication between GitHub Actions and Google Cloud.
- **Deployment Method:** Source-based deployment using Google Cloud Buildpacks.

## GCP Infrastructure (Managed by Terraform CDK)
- **`github-cd-sa`:** A dedicated IAM Service Account for the GitHub Actions workflow to authenticate with GCP.
- **`cloud-build-sa`:** A dedicated IAM Service Account for the Cloud Build process to execute the build and deployment, following the principle of least privilege.
- **`githubCdDeployer`:** A custom IAM role with the minimal permissions required to deploy a new version of a Cloud Run service (`run.services.get`, `run.services.update`).
- **Workload Identity Federation:** A pool and provider are configured to trust the GitHub Actions OIDC provider.

## Technical Stack
- **Language:** Python 3.13
- **Framework:** Flask
- **Dependency Manager:** `uv` is used for local development, with dependencies defined in `pyproject.toml`.
- **Infrastructure as Code:** Terraform CDK for Python is used to manage GCP resources.
- **Production Dependencies:** A `requirements.txt` file is compiled from `pyproject.toml` for use by Cloud Run.
- **Production Server:** Gunicorn, configured via the `Procfile`.
- **Development Server:** The standard Flask development server.

## Key Decisions & History
- The project started as a basic Flask app.
- A full CI/CD pipeline was built using GitHub Actions and Google Cloud Run.
- The infrastructure setup was migrated from a shell script (`scripts/setup_gcp_cd.sh`) to a declarative Infrastructure as Code approach using Terraform CDK for Python.
- The principle of least privilege was followed, resulting in the creation of two dedicated service accounts and a custom IAM role with minimal permissions.
- After extensive debugging of IAM permissions, the final, working configuration was achieved by granting the correct roles to the correct service accounts, particularly for the Cloud Build and Cloud Run services.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maufee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
