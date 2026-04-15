## vllm-container-ngc

> This project provides the configuration to build a Docker container for serving the DeepSeek-R1-Distill-Qwen-1.5B model using NVIDIA's NGC vLLM container.

# vLLM NGC Container - DeepSeek-R1

This project provides the configuration to build a Docker container for serving the DeepSeek-R1-Distill-Qwen-1.5B model using NVIDIA's NGC vLLM container.

## Overview

The primary goal is to package the NGC vLLM server with the `DeepSeek-R1-Distill-Qwen-1.5B` model (1.5B parameters) into a container image that can be easily deployed. The build process is managed by Google Cloud Build.

## Project Goals and Workflow

### Purpose

The goal of this project is to containerize DeepSeek-R1-Distill-Qwen-1.5B (1.5B parameters) using NVIDIA's NGC vLLM container for the fastest possible inference, with a focus on minimizing cold start times. The primary issue being addressed is the slow initial response after periods of inactivity, which is solved by performing model loading during the build process.

### Current Workflow (Manual)

1.  **Build**: A `Dockerfile` is used to create the container image. The build is triggered manually using `gcloud builds submit`.
2.  **Monitor**: The build logs in Cloud Build are monitored manually for errors.
3.  **Deploy**: Upon a successful build, the new container image is deployed manually to an existing service in Google Cloud Run.

### Desired Automation and Testing

-   **Automated Deployment**: Automatically deploy the container to Cloud Run if the Cloud Build process completes successfully.
-   **Post-Deployment Test**: After a successful deployment, run an automated test that calls the Cloud Run service endpoint to verify the LLM is responsive and provides a valid output.

## Building the Container

The container is built using Google Cloud Build and the configuration is defined in `cloudbuild.yaml`.

### Prerequisites

1.  A Google Cloud Project with the Cloud Build API and Secret Manager API enabled.
2.  The `gcloud` CLI installed and authenticated.
3.  A Hugging Face token with access to the DeepSeek-R1 model stored in Google Secret Manager. The secret must be named `HF_TOKEN`.

### Build Command

To start the build, run the following `gcloud` command from the root of the repository:

```bash
gcloud builds submit --config cloudbuild.yaml
```

**Important**: Do NOT pass `HF_TOKEN` via `--substitutions`. The token is automatically injected from Secret Manager. User-defined substitutions must start with an underscore (e.g., `_MY_VAR`).

### How It Works

The `cloudbuild.yaml` file defines the build steps:

-   **Builder**: It uses the standard Docker builder from `gcr.io/cloud-builders/docker`.
-   **Secrets**: It securely injects the `HF_TOKEN` from Secret Manager into the Docker build process. This is required to download the model from the Hugging Face Hub during the `docker build` command.
-   **Image**: The final container image is tagged and pushed to Google Artifact Registry. The image path is defined by the `_IMAGE` substitution variable:
    `us-central1-docker.pkg.dev/${PROJECT_ID}/vllm-deepseek-r1-repo/vllm-deepseek-r1-1-5b`
-   **Machine Type**: The build is configured to run on a high-CPU machine (`E2_HIGHCPU_8`) to ensure the build process is fast and efficient.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/patelmm79)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/patelmm79)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
