## pressure-gauge-detection

> This project is a computer vision project for detecting needles in pressure gauges using the YOLOv8 object detection model. The project uses datasets from Roboflow. There are two datasets included in this project:

# Project Overview

This project is a computer vision project for detecting needles in pressure gauges using the YOLOv8 object detection model. The project uses datasets from Roboflow. There are two datasets included in this project:

1.  **labell.v5-v2.yolov8**: A dataset of 19 images for detecting needles.
2.  **Pressure Gauges.v2-gauge-version-2.yolov8**: A larger dataset of 960 images for the same purpose.

The goal of the project is to train a YOLOv8 model to accurately detect the needle in images of pressure gauges.

# Building and Running

This project uses Python and the `ultralytics` library.

## Setup

It is recommended to use a virtual environment. The project contains a `.venv` directory. To activate it, run:

```bash
source .venv/bin/activate
```

You may need to install dependencies. While a `requirements.txt` is not provided, the key dependency is `ultralytics`. You can install it using pip:

```bash
pip install ultralytics
```

## Training the Model

There are several training scripts in the project.

### Using the `labell` dataset

To train the model with the `labell` dataset, navigate to the `labell.v5-v2.yolov8` directory and run the `train.py` script:

```bash
cd labell.v5-v2.yolov8
python train.py
```

This will train a YOLOv8 model (`yolov8n.pt`) for 120 epochs with an image size of 640. The trained model and results will be saved in the `runs/detect/` directory.

### Using the `Pressure Gauges` dataset

To train the model with the `Pressure Gauges` dataset, you would need to create a similar `train.py` script in the `Pressure Gauges.v2-gauge-version-2.yolov8` directory, making sure the `data.yaml` path is correct.

## Running Inference

After training, you can use the trained model to run inference on new images. You can use the `yolo` command-line interface for this. For example, to run prediction on an image:

```bash
yolo predict model=path/to/your/best.pt source=path/to/your/image.jpg
```

Replace `path/to/your/best.pt` with the path to your trained model (e.g., `labell.v5-v2.yolov8/runs/detect/train/weights/best.pt`) and `path/to/your/image.jpg` with the image you want to test.

# Development Conventions

*   **Datasets**: Datasets are managed using Roboflow and are in YOLOv8 format. Each dataset has a `data.yaml` file that describes the dataset, classes, and paths to the training, validation, and test sets.
*   **Training**: Training is done using the `ultralytics` Python library. Training scripts are provided in the dataset directories.
*   **Models**: The project uses the `yolov8n.pt` pretrained model as a starting point for training.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aadidevcb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aadidevcb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
