<h1 align="center">Energy-Aware Benchmarking of Vision Models for Vehicle Rear-Camera Blind-Spot Detection on Raspberry Pi 4B</h1>

<p align="center"><strong>Accuracy, latency, and energy comparison of six YOLO detectors for onboard blind-spot detection on a Raspberry Pi 4B (4&nbsp;GB).</strong></p>

---

## Overview
This repository contains the code for an accuracy and energy analysis of six lightweight object detectors for rear-view vehicle detection:

**YOLOv4-tiny · YOLOv5 · YOLOv8 · YOLOv11 · YOLOv12 · YOLO26**

The study covers:

- Training the six models on a 12-class rear-view vehicle dataset
- Evaluation with precision, recall, mAP@50, mAP@50-95, confusion matrices, and precision-recall curves
- Latency and energy analysis under a Raspberry Pi 4B (4 GB) deployment profile
- Multi-criteria ranking of the models with Analytic Hierarchy Process (AHP) weights and a sensitivity analysis

> **Note:** The Raspberry Pi latency and energy values are **estimates**, not measurements on a physical board. They are obtained from CPU measurements on a constrained host, mapped to the board with a published YOLOv4-tiny reference speed, and converted to energy with a utilization-based power model (see [Energy Analysis](#energy-analysis)).

---

## Dataset
The models are trained on the **Rear-View Vehicle Object Detection Dataset**, published on Mendeley Data (DOI: [10.17632/9bg9326x7j.1](https://data.mendeley.com/datasets/9bg9326x7j/1)).

| Item | Value |
|---|---|
| Images | 1,678 (train 1,467 / valid 140 / test 71) |
| Annotations | 4,370 |
| Classes (12) | 2-wheelers, bus, cargo van, jeep, minibus, person, sedan, SUV, tricycle, truck, utility vehicle, van |
| Preprocessing | Auto-orient |
| Augmentation (training, 3 outputs per example) | Zoom 0–20%, rotation ±15°, shear ±10°, saturation ±25%, brightness ±15%, exposure ±10%, blur up to 1.5 px, noise up to 0.1% |

The dataset is derived from the *Back Camera New* dataset by THESIS1 (Roboflow Universe, CC BY 4.0). Some images were re-annotated, and the annotations were reviewed and corrected.

---

## Models and Training
| Model | Framework | Variant | Input size | Batch size | Training length |
|---|---|---|---|---|---|
| YOLOv4-tiny | Darknet | tiny | 416 | 64 | 24,000 iterations |
| YOLOv5 | YOLOv5 repository | small (s) | 640 | 16 | 50 epochs |
| YOLOv8 | Ultralytics | small (s) | 640 | 16 | 50 epochs |
| YOLOv11 | Ultralytics | small (s) | 640 | 16 | 50 epochs |
| YOLOv12 | Ultralytics | small (s), random initialization | 640 | 16 | 50 epochs |
| YOLO26 | Ultralytics | medium (m) | 640 | 16 | 50 epochs |

YOLOv5, YOLOv8, YOLO11, and YOLO26 start from COCO-pretrained weights. YOLO12 is trained from its architecture definition without pretrained weights.

---

## Energy Analysis
The notebook `energy_analysis` in `codebase_final/` performs the analysis in these steps:

1. **Environment setup** – a CPU-only Raspberry Pi 4B profile (GPU hidden, threads limited to the board's core count or the host's, 4 GB memory budget with 500 MB reserved for the OS).
2. **Model complexity** – parameters (fused), GFLOPs, and weight size at 640×640.
3. **Inference benchmark** – pre-processing, inference, and post-processing times on the test set (OpenCV DNN for the Darknet model, ONNX Runtime for the others), with CPU utilization and memory.
4. **Latency mapping** – host latency scaled to the Raspberry Pi 4B using a published YOLOv4-tiny reference speed, plus a second, complexity-based (GFLOPs) estimate.
5. **Power and energy modeling** – linear utilization-based power model with low, typical, and high scenarios; energy per inference, detection-only energy, inferences per Wh, and continuous operation at 1, 2, and 5 FPS.
6. **Multi-criteria decision analysis** – accuracy, latency, energy, and memory combined with AHP weights for three scenarios (safety-first, balanced, energy-first), and a sensitivity analysis over 10,000 random weight vectors.

---

## Repository Structure
```
codebase_final/
├── energy analysis                 # Raspberry Pi 4B latency and energy analysis + AHP decision analysis
└── notebook/                       # Training notebooks for the six models
    ├── blindspot_yolov4_tiny.ipynb
    ├── blindspot_yolov5.ipynb
    ├── blindspot_yolov8_final.ipynb
    ├── blindspot_yolov11.ipynb
    ├── blindspot_yolov12.ipynb
    └── blindspot_yolo26.ipynb
```
---

## Requirements
- Python 3.10+ (tested with Python 3.13)
- `ultralytics`, `roboflow`, `onnx`, `onnxruntime`, `onnxslim`, `psutil`, `pyyaml`, `numpy`, `pandas`
- **OpenCV 4.x** (`opencv-python` and `opencv-python-headless` below version 5). OpenCV 5.0 does not provide `cv2.dnn.readNetFromDarknet`, which the YOLOv4-tiny benchmark needs.
- [Darknet](https://github.com/AlexeyAB/darknet) for training YOLOv4-tiny, and the [YOLOv5 repository](https://github.com/ultralytics/yolov5) for training and exporting YOLOv5

---

## Usage
The notebooks are written for Google Colab and mount Google Drive for the trained model files.

1. **Install the dependencies** and restart the runtime:
```bash
   pip uninstall -y opencv-python opencv-python-headless opencv-contrib-python
   pip install "opencv-python<5" "opencv-python-headless<5" ultralytics roboflow onnx onnxruntime onnxslim psutil pyyaml
```
2. **Download the dataset** from [Mendeley Data](https://data.mendeley.com/datasets/9bg9326x7j/1), or through Roboflow. Do not commit your Roboflow API key to the repository. Store it as a secret or an environment variable.
3. **Train the models** by running the notebooks in `codebase_final/notebook/`.
4. **Run the energy analysis** with `codebase_final/energy_analysis.ipynb`. Set the paths of the trained weights (and the YOLOv4-tiny `.cfg` file), then run the cells in order.

---

## Limitations
- Latency and energy are estimated, not measured on a physical Raspberry Pi 4B. Measuring them with a power meter is future work.
- The reference speed used for the latency mapping was measured with a different framework (ncnn) on a board clocked at 1950 MHz.
- The dataset is small and imbalanced, and adverse weather, night, and highway conditions were not evaluated separately.
- Training settings differ across models (YOLOv4-tiny uses iterations, YOLOv12 has no pretrained weights, and YOLO26 is the medium variant).

---

## Preprint
- https://arxiv.org/abs/2601.01274

---

## Citation
If you use the dataset, please cite:

```bibtex
@misc{rearviewdataset,
  author       = {Haque, Md Sadman and Robin, Robiul Awoul and Hafiz, Fahim},
  title        = {Rear-View Vehicle Object Detection Dataset},
  year         = {2026},
  howpublished = {Mendeley Data, Version 1},
  doi          = {10.17632/9bg9326x7j.1}
}
```
