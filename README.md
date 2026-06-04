# Periapical Lesion Detection in Panoramic Radiographs

Automated detection of periapical lesions in dental panoramic radiographs using YOLO-based object detection models. This project includes a structured diagnostic investigation into the factors limiting detection performance, alongside experiments comparing multiple architectures and task formulations.

---

## Overview

Periapical lesions are radiographic indicators of apical periodontitis. Detecting them automatically in panoramic radiographs is challenging due to their small size, low contrast, and subtle appearance. This study trains and evaluates several detection configurations — varying model architecture, input resolution, class structure, and preprocessing — to identify what drives performance and what limits it.

The key finding is that detection performance is primarily constrained by two factors: **lesion size and radiographic visibility**, and **the complexity of grading lesions into severity classes**. Addressing both together raises test precision from ~53% (three-class baseline) to ~93% (single-class, large-lesion focused configuration).

---

## Dataset

**Panoramic Radiographs with Periapical Lesions**
- Source: Viet Do, Mendeley Data, Version 3, April 2024
- DOI: [10.17632/kx52tk2ddj.3](https://doi.org/10.17632/kx52tk2ddj.3)
- License: CC BY 4.0
- 3,926 images (originals + augmented), annotated in Pascal VOC XML format
- Three lesion severity classes: `Periapical_Type3`, `Periapical_Type4`, `Periapical_Type5`

| Split      | Images | Annotations |
|------------|--------|-------------|
| Train      | 2,746  | 4,671       |
| Validation | ~589   | 1,033       |
| Test       | ~590   | 1,037       |

Annotations were converted from Pascal VOC XML to YOLO format during preprocessing.

---

## Experiments

The study ran in two phases: initial YOLO baselines, then a structured diagnostic investigation across four hypotheses.

| Group         | Configuration                          | mAP@0.50 | Precision | Recall |
|---------------|----------------------------------------|----------|-----------|--------|
| Baseline      | YOLOv9m — 3-class, 640px              | 0.515    | 0.534     | 0.505  |
| Baseline      | YOLOv10m — 3-class, 640px             | 0.476    | 0.506     | 0.500  |
| Baseline      | YOLOv11m — 3-class, 640px             | 0.483    | 0.538     | 0.520  |
| Hypothesis 1  | Faster R-CNN + MobileNetV3 FPN        | 0.030    | ~0.00     | ~0.00  |
| Hypothesis 2  | YOLO11l — tiled, 1280px (full-img test)| 0.044   | 0.092     | 0.140  |
| Hypothesis 3  | Single-class (Type3 only)             | 0.511    | 0.514     | 0.541  |
| Hypothesis 3  | Single-class (Type5 only, 640px)      | 0.557    | 0.562     | 0.580  |
| Hypothesis 3  | Two-class (Type3 + Type4)             | 0.496    | 0.515     | 0.485  |
| Hypothesis 3  | Balanced three-class (undersampled)   | 0.380    | 0.441     | 0.442  |
| Hypothesis 3  | **Single-class merged (full dataset)**| **0.724**| 0.719     | 0.647  |
| Hypothesis 4  | **YOLOv10l — Type5 only, 1024px**     | **0.842**| 0.934     | 0.773  |

---

## Setup

### Requirements

```bash
pip install ultralytics torch torchvision opencv-python albumentations
```

For Faster R-CNN experiments:
```bash
pip install torchvision  # includes torchvision.models.detection
```


## Key Findings

- **Model choice is not the bottleneck**: YOLOv9m, YOLOv10m, and YOLOv11m all produce similar moderate results (~0.48–0.52 mAP@0.50) on the three-class task.
- **Class complexity matters**: Merging all severity labels into one class raised mAP@0.50 from 0.515 to 0.724 — a ~40% relative gain.
- **Lesion size matters more**: Focusing on the largest lesion class (Type5) at sufficient resolution reached 0.842 mAP@0.50 and 0.934 precision.
- **Tiling requires matched train/test scale**: The high-resolution tiled model achieved strong validation performance but collapsed on full-image test evaluation due to the distribution mismatch.
- **Balancing via undersampling is counterproductive**: The balanced three-class setup (0.380 mAP@0.50) performed worst by discarding supervision for the dominant classes.

---

## Hardware

| Experiment             | GPU          |
|------------------------|--------------|
| Most experiments       | Tesla T4     |
| High-resolution tiled  | NVIDIA A100  |

---

## Citation

If you use this dataset, please cite:

```
Viet Do. Panoramic Radiographs with Periapical Lesions. Mendeley Data, V3, 2024.
https://doi.org/10.17632/kx52tk2ddj.3
```

---

## License

Dataset: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)  
Code: MIT License
