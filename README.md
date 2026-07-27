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

## Quickstart

```bash
pip install -r requirements.txt

# Train
python src/train.py --data data/circuit_detection.yaml --epochs 300 --imgsz 1280

# Evaluate on test set
python src/evaluate.py --weights models/best.pt --data data/circuit_detection.yaml

# Run inference on a new image
python src/infer.py --weights models/best.pt --source path/to/pcb_image.jpg
```

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

## Known Limitations

- Small components (resistors, resistor networks, jumpers) are under-detected due to limited pixel footprint and a modest training set size.
- Two classes (Ferrite Bead, iC) have too few training examples to learn reliably.

## Future Work

- Expand dataset to 1,500–2,000+ images, balanced across classes.
- Push input resolution to 1536–1920px.

## License

MIT (see `LICENSE`).
