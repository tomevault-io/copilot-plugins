## fyp

> This project is a multi-stage deep learning pipeline for classifying chest X-ray images. It is designed to first triage images into "normal" and "abnormal" categories, then use a set of specialist models to identify specific pathologies in the abnormal images, and finally, use a consensus model to produce the final prediction. The project is written in Python and uses the PyTorch deep learning framework. The codebase is structured to be run on a high-performance computing (HPC) cluster, as indicated by the `nscc_scripts` directory.

# GEMINI.md

## Project Overview

This project is a multi-stage deep learning pipeline for classifying chest X-ray images. It is designed to first triage images into "normal" and "abnormal" categories, then use a set of specialist models to identify specific pathologies in the abnormal images, and finally, use a consensus model to produce the final prediction. The project is written in Python and uses the PyTorch deep learning framework. The codebase is structured to be run on a high-performance computing (HPC) cluster, as indicated by the `nscc_scripts` directory.

The pipeline consists of three main stages:

*   **Stage 1: Triage Model:** A ResNet-50 model is trained to classify images as "normal" or "abnormal".
*   **Stage 2: Specialist Models:** Several models (ResNet-50, DenseNet-121, or ViT) are trained to detect specific pathologies (e.g., Pneumonia, Atelectasis). These models use pathology-specific image preprocessing techniques to improve performance.
*   **Stage 3: Consensus Model:** A Multi-Layer Perceptron (MLP) is trained on the outputs of the triage and specialist models to produce a final, consolidated prediction.

## Building and Running

The project is run using a series of shell scripts that execute the Python training scripts for each stage of the pipeline. The main scripts are located in the `nscc_scripts` directory.

### Dependencies

The required Python packages are listed in the `requirements.txt` file. They can be installed using pip:

```bash
pip install -r requirements.txt
```

### Running the Pipeline

The following scripts are used to run the different stages of the pipeline. Note that the scripts expect a specific directory structure and the data to be located in a base directory (referred to as `SCRATCH_BASE_PATH` in the scripts).

**Stage 1: Train the Triage Model**

```bash
# Navigate to the Stage1_scripts directory
cd nscc_scripts/Stage1_scripts

# Run the training script
./train.sh
```

**Stage 2: Train the Specialist Models**

```bash
# Navigate to the Stage2_scripts directory
cd nscc_scripts/Stage2_scripts

# Run the training script for each pathology
./specialist_train_final.sh --pathology <pathology_name> --architecture <architecture_name> --base_dir <path_to_base_dir>
```

*   `<pathology_name>`: The name of the pathology (e.g., `pneumonia`).
*   `<architecture_name>`: The model architecture to use (e.g., `resnet50`).
*   `<path_to_base_dir>`: The path to the base directory where the data is located.

**Stage 3: Train the Consensus Model**

```bash
# Navigate to the Stage3_scripts directory
cd nscc_scripts/Stage3_scripts

# Run the training script
./run_consensus_train.sh --base_dir <path_to_base_dir>
```

*   `<path_to_base_dir>`: The path to the base directory where the data is located.

## Development Conventions

*   **Code Style:** The Python code generally follows the PEP 8 style guide.
*   **Modularity:** The project is well-structured, with different stages of the pipeline separated into their own directories and scripts.
*   **Configuration:** The scripts use a `CONFIG` dictionary to manage hyperparameters and file paths.
*   **Command-line Arguments:** The training scripts use `argparse` to accept command-line arguments for things like pathology, architecture, and base directory.
*   **Logging:** The scripts use `tqdm` to display progress bars and print informative messages to the console.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AyushmanMalla) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
