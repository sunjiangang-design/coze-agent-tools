# MFG Vision QC - Industrial Visual Quality Control Demo

> Based on Ultralytics YOLOv8 + Anomalib, quick-start template for manufacturing visual QC
> Demo scenario: vibrating screen industry, adaptable to other domains

## Quick Start

```bash
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
python scripts/02_train_yolo.py --demo
```

## Train with Your Data

```bash
python scripts/01_prepare_data.py --src ./raw_images --dst ./data --mode yolo
python scripts/02_train_yolo.py --data configs/yolo_v8s.yaml --epochs 100
python scripts/04_export_model.py --model models/train/best.pt --format onnx
python scripts/05_inference.py --source test.jpg --model models/train/best.pt
```

## Anomaly Detection (no defect samples needed)

```bash
python scripts/03_train_anomalib.py --normal-dir data/anomalib/normal
```

## Vibrating Screen Parameters

| Parameter | Value | Reason |
|-----------|-------|--------|
| imgsz | 1280 | Thin wires need high resolution |
| conf | 0.4 | Lower threshold, prefer over-detection |
| mosaic | 0.3 | Reduced to preserve small defects |

## Deployment Options

| Solution | Hardware | Latency |
|----------|----------|---------|
| GPU + TensorRT | NVIDIA T4 | ~5ms |
| CPU + ONNX | Intel i5 | ~80ms |
| Jetson Nano | Jetson | ~30ms |

## Resources

- [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics)
- [Anomalib](https://github.com/openvinotoolkit/anomalib)
- [Knowledge Base](https://scnns5gzrku4.feishu.cn/wiki/My35wb54Ui4mGjkwUUycBgGxnsb)

License: MIT
