## ai-training

> This project aims to fine-tune a Large Language Model (LLM) to create a specialized chatbot or information retrieval system about the Sancaktepe municipality in Istanbul, Turkey.

# Project: Fine-tuning a Turkish Municipality LLM

This project aims to fine-tune a Large Language Model (LLM) to create a specialized chatbot or information retrieval system about the Sancaktepe municipality in Istanbul, Turkey.

## Project Overview

The primary goal is to leverage the documents in the `documents/` directory to train a language model that can answer questions and provide information about the municipality's services, regulations, and general information.

The `plan.md` file outlines a detailed technical plan for this process, which involves:

*   **Model:** Using a base model like `Meta-Llama-3.1-8B-Instruct` or `Mistral-7B-Instruct-v0.3`.
*   **Fine-tuning method:** Employing QLoRA (Quantized Low-Rank Adaptation) for efficient training.
*   **Tooling:** Utilizing the `Unsloth` library for optimized performance on NVIDIA GPUs.

## Dataset

The `documents/` directory contains a collection of Markdown (`.md`) and JSON (`.json`) files that serve as the knowledge base for the LLM. These files cover a wide range of topics related to the Sancaktepe municipality.

## How to Run

This project is now fully automated using Docker Compose. The process handles everything from data preparation and model training to inference.

### Prerequisites

*   An NVIDIA GPU with the latest CUDA drivers installed.
*   Docker and Docker Compose installed on your system.
*   The NVIDIA Container Toolkit installed to enable GPU access for Docker.

### Step 1: The "One-Command" Execution

Simply run the following command in your terminal:

```bash
docker-compose up
```

**What happens next is automatic:**

1.  **First-Time Run (Training):**
    *   If you are running this for the first time, the system will detect that no trained model exists.
    *   It will automatically prepare the dataset and then begin the **fine-tuning process**. You will see logs showing the model download status, followed by a progress bar for the training itself.
    *   This is a long, resource-intensive process that can take many hours.
    *   Once complete, the newly fine-tuned model will be saved to a `my-finetuned-model/` directory in your project folder. The container will then run inference with a default prompt.

2.  **Subsequent Runs (Inference):**
    *   On every subsequent run, the system will detect the `my-finetuned-model/` directory.
    *   It will **skip the training process entirely** and immediately run inference using your already-trained model. This will be much faster.

### Step 2: Running Inference with a Custom Prompt

The default command runs a test inference. To ask your own questions, use the `docker-compose run` command. This will start a new container and run your specific query.

```bash
docker-compose run --rm qlora-llm python3 src/inference.py "Your custom prompt here"
```

For example:
```bash
docker-compose run --rm qlora-llm python3 src/inference.py "Sancaktepe'de hangi parklar var?"
```

### Step 3 (Optional): Exporting to GGUF for Deployment

After your model has been trained (i.e., after the first `docker-compose up` run is complete), you can create a highly optimized, portable GGUF file. This is the recommended format for running the model efficiently with tools like Ollama or LM Studio.

Run the export script:
```bash
python src/export_gguf.py
```
This will create a `sancaktepe-model.gguf` file in your project directory. This single file is your entire fine-tuned model, ready for deployment.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ariftanis)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ariftanis)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
