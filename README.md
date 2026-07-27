<div align="center">

# 🔍 PCB Component Detection with YOLOv8

### Teaching a neural network to "read" a circuit board the way an engineer does

![Status](https://img.shields.io/badge/status-completed-brightgreen)
![Model](https://img.shields.io/badge/model-YOLOv8m-blue)
![mAP](https://img.shields.io/badge/mAP%400.5-42.6%25-orange)
![Classes](https://img.shields.io/badge/classes-23-lightgrey)
![License](https://img.shields.io/badge/license-MIT-informational)

</div>

---

## 🧩 The Problem

A single circuit board can carry **hundreds of tiny components** — resistors, capacitors, ICs, connectors, switches — packed together in a few square inches. Manually identifying and counting them is slow and error-prone.

**Goal:** train a computer vision model that looks at a photo of a PCB and automatically draws a box around every component, labeling what it is.

```
📷 Raw PCB photo  ──────▶  🧠 YOLOv8 model  ──────▶  📦 Labeled components + confidence scores
```

---

## 🏗️ Architecture

The system is a straightforward **train → evaluate → infer** pipeline built around a YOLOv8m detector.

```mermaid
flowchart LR
    subgraph Data["📁 Data Layer"]
        A[Raw PCB Images] --> B[Labeled Dataset<br/>548 train / 80 val / 44 test]
        B --> C[circuit_detection.yaml<br/>23 class definitions]
    end

    subgraph Train["🧠 Training"]
        C --> D[YOLOv8m backbone<br/>25.8M params]
        D --> E[300 epochs @ 1280x1280<br/>AdamW · reduced mosaic aug]
        E --> F[best.pt<br/>trained weights]
    end

    subgraph Eval["📊 Evaluation"]
        F --> G[Test set: 44 images]
        G --> H[mAP / Precision / Recall<br/>per-class breakdown]
    end

    subgraph Deploy["🚀 Inference"]
        F --> I[New PCB image]
        I --> J[Bounding boxes +<br/>class + confidence]
    end

    style Data fill:#EAF1F3,stroke:#1F4E5C
    style Train fill:#FFF3CD,stroke:#8a6d3b
    style Eval fill:#E6F2F5,stroke:#1F4E5C
    style Deploy fill:#D9EAD3,stroke:#2e6b34
```

**Why YOLOv8?** Single-pass detection (no separate region-proposal step) means it's fast enough for real-time inspection (~96ms/image) while still being accurate enough to fine-tune on a small custom dataset.

**Why 1280px instead of the default 640px?** Many components — resistors especially — are only a handful of pixels wide. Doubling the input resolution was the single biggest lever for making them visible to the network at all.

---

## 📖 The Story So Far

| Stage | mAP@0.5 | What changed |
|---|---|---|
| 🟥 Baseline (YOLOv8n, 100 epochs, 640px) | 19.2% | First working pipeline |
| 🟧 Improved (YOLOv8m, 200 epochs, 640px) | 35.0% | Bigger model, more training |
| 🟩 **Ultimate (YOLOv8m, 300 epochs, 1280px)** | **42.6%** | High-res input + tuned augmentation |

> **+122% improvement** from first working baseline to final model.

---


## 🎯 Per-Class Performance

```
Switch                 ████████████████░░░░  82.4%   🟢 Excellent
EM                      ███████████████░░░░░  76.1%   🟢 Excellent
Diode                   ██████████████░░░░░░  71.0%   🟢 Good
Electrolytic Capacitor  █████████████░░░░░░░  64.6%   🟢 Good
Connector               ██████████░░░░░░░░░░  51.8%   🟡 Moderate
IC                      ████████░░░░░░░░░░░░  40.6%   🟡 Moderate
Resistor                ██░░░░░░░░░░░░░░░░░░   8.9%   🔴 Poor
Resistor Network        █░░░░░░░░░░░░░░░░░░░   5.5%   🔴 Poor
```

**The pattern:** large, visually distinct components (switches, ICs, electrolytic caps) are learned easily. Small, densely-packed, visually-similar components (resistors, resistor networks, jumpers) are the hard part — this is a resolution and dataset-size problem, not a model problem. Full breakdown in [`results/training_report.txt`](results/training_report.txt).

---


## 🗂️ Dataset at a Glance

| | |
|---|---|
| Classes | 23 (Button, Capacitor, Capacitor Jumper, Clock, Connector, Diode, EM, Electrolytic Capacitor, Ferrite Bead, IC, Inductor, Jumper, Led, Pads, Pins, Resistor, Resistor Jumper, Resistor Network, Switch, Test Point, Transistor, Unknown Unlabeled, iC) |
| Train / Val / Test | 548 / 80 / 44 images |
| Annotated instances | 18,602 (validation split) |
| Resolution | 1280 × 1280 |

---


# PCB-Component-Detection-YOLOv8

Automated detection and classification of electronic components (resistors, capacitors, ICs, connectors, switches, etc.) on printed circuit board (PCB) images, using a fine-tuned YOLOv8 model.

**Status:** Completed (internship project) — 23/23 classes trained, model detects components on unseen test images with 42.6% mAP@0.5.

## Overview

| | |
|---|---|
| **Model** | YOLOv8m (25.8M params) |
| **Input resolution** | 1280 × 1280 |
| **Classes** | 23 |
| **Training set** | 548 images |
| **Epochs** | 300 (8.1 hrs on Tesla T4) |
| **mAP@0.5** | 42.6% |
| **Precision / Recall** | 51.4% / 47.5% |
| **Inference speed** | ~96 ms/image |

See [`docs/PCB_Component_Detection_Report.pdf`](docs/PCB_Component_Detection_Report.pdf) for the full write-up, methodology, per-class results, and sample detections.

## Dataset

23 component classes: Button, Capacitor Jumper, Capacitor, Clock, Connector, Diode, EM,
Electrolytic Capacitor, Ferrite Bead, IC, Inductor, Jumper, Led, Pads, Pins, Resistor Jumper,
Resistor Network, Resistor, Switch, Test Point, Transistor, Unknown Unlabeled, iC.

548 train / 80 valid / 44 test images, 18,602 annotated instances (validation split).

## Results Summary

| Component Class | mAP@0.5 |
|---|---|
| Switch | 82.4% |
| EM | 76.1% |
| Diode | 71.0% |
| Electrolytic Capacitor | 64.6% |
| Connector | 51.8% |
| IC | 40.6% |
| Resistor | 8.9% |
| Resistor Network | 5.5% |

Full per-class breakdown in [`results/training_report.txt`](results/training_report.txt) and the [project report](docs/PCB_Component_Detection_Report.pdf).
<img width="2400" height="1200" alt="results" src="https://github.com/user-attachments/assets/7b849fb6-acf3-4535-87c4-c40251beafa1" />

 ## Results Detection
<img width="1961" height="2047" alt="image5" src="https://github.com/user-attachments/assets/fdd5c3f1-5256-4d96-b87b-a1b2c476f8f3" />
<img width="2048" height="1687" alt="image7" src="https://github.com/user-attachments/assets/757482a6-f166-487a-bdad-2164804fea74" />
<img width="2048" height="1375" alt="image3" src="https://github.com/user-attachments/assets/e9308550-8389-469d-9eec-66780a7b3b31" />
<img width="2454" height="2501" alt="image4" src="https://github.com/user-attachments/assets/eb2b3dd5-5f62-4288-8c46-7d3bdb577762" />

 
---

## 📄 Full Report

The complete write-up — methodology, all per-class metrics, and full-size detection images — is in [`docs/PCB_Component_Detection_Report.pdf`](docs/PCB_Component_Detection_Report.pdf).

## 📜 License

MIT — see [`LICENSE`](LICENSE).
