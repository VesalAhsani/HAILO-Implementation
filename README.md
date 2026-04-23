# HAILO-Implementation

````markdown
# YOLOv8 to Hailo Conversion Guide for Raspberry Pi 5 AI HAT+

This guide explains how to convert a trained YOLOv8n model into a Hailo `.hef` file for inference on Raspberry Pi 5 with AI HAT+ using Hailo-8.

## Pipeline Overview

```text
best.pt → best.onnx → best.hef → Raspberry Pi 5 Hailo inference
````

## Requirements

### Model and Data

```text
best.pt
calib_images/
```

The `calib_images/` folder should contain approximately 300–500 representative images.

Notes:

* Labels are not required for calibration images.
* Calibration images should match the real deployment environment.
* For drone detection, include sky backgrounds, small drones, different lighting conditions, and negative samples.

### System

Use Ubuntu or WSL2 Ubuntu.

Required:

```text
Python 3.8+
Internet access
Hailo Dataflow Compiler
Hailo Model Zoo
Ultralytics
```

## Installation

### 1. Install system dependencies

```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv git wget curl
```

### 2. Install Ultralytics

```bash
pip install ultralytics
```

### 3. Install Hailo Dataflow Compiler

The Hailo Dataflow Compiler must be downloaded from the Hailo Developer Zone:

```text
https://hailo.ai/developer-zone/
```

After downloading the `.deb` file, install it with:

```bash
sudo dpkg -i hailo_dataflow_compiler*.deb
sudo apt --fix-broken install -y
```

Verify installation:

```bash
hailo --version
```

### 4. Install Hailo Model Zoo

```bash
git clone https://github.com/hailo-ai/hailo_model_zoo.git
cd hailo_model_zoo
pip install -e .
```

Verify installation:

```bash
hailomz --help
```

## Step 1: Export YOLOv8 to ONNX

Run this in the folder containing `best.pt`:

```bash
yolo export model=best.pt imgsz=640 format=onnx opset=11 simplify=True
```

Expected output:

```text
best.onnx
```

## Step 2: Compile ONNX to HEF

Run:

```bash
hailomz compile yolov8n \
  --ckpt best.onnx \
  --hw-arch hailo8 \
  --calib-path calib_images \
  --classes 1
```

Use `--hw-arch hailo8` for Raspberry Pi 5 AI HAT+ with Hailo-8.

## Step 3: Locate the HEF File

After successful compilation, find the generated `.hef` file:

```bash
find . -name "*.hef"
```

Example output:

```text
best.hef
```

## Step 4: Transfer HEF to Raspberry Pi

Replace `<RASPBERRY_PI_IP>` with the IP address of the Raspberry Pi:

```bash
scp best.hef vesalpi@<RASPBERRY_PI_IP>:/home/vesalpi/drone_inference/
```

## Notes

This guide assumes:

```text
Model: YOLOv8n
Input size: 640
Number of classes: 1
Class name: drone
Target hardware: Hailo-8
```

If using a different YOLOv8 model size, replace `yolov8n` in the compile command with the matching model type, such as:

```bash
hailomz compile yolov8s \
  --ckpt best.onnx \
  --hw-arch hailo8 \
  --calib-path calib_images \
  --classes 1
```

```
```
