# Inference & Production Optimization with ONNX

## 1. Running YOLO11n Inference in Python

```python
import cv2
from ultralytics import YOLO

# 1. Load trained model weights
model = YOLO("best.pt")

# 2. Run prediction on an image, video file, or webcam stream (source=0)
results = model.predict(
    source="test.jpg", 
    conf=0.35,        # Confidence threshold: filter predictions < 35%
    iou=0.45,         # NMS IoU threshold: suppress overlapping boxes
    save=False,
    imgsz=640
)

# 3. Extract detections
for result in results:
    boxes = result.boxes
    for box in boxes:
        # Bounding box in [x1, y1, x2, y2]
        x1, y1, x2, y2 = box.xyxy[0].cpu().numpy()
        confidence = float(box.conf[0])
        class_id = int(box.cls[0])
        class_name = model.names[class_id]

        print(f"Detected {class_name} ({confidence*100:.1f}%) at [{x1:.0f}, {y1:.0f}, {x2:.0f}, {y2:.0f}]")
```

---

## 2. Why PyTorch `.pt` is Bad for Edge Production

While PyTorch is excellent for development and training, deploying `.pt` files in production has major drawbacks:
1. **Bloated Dependencies:** Requires installing full PyTorch (`torch`), which takes $2\text{--}4\text{ GB}$ of disk space.
2. **CPU Sluggishness:** PyTorch's default CPU execution is unoptimized compared to dedicated inference runtimes.
3. **Hardware Lock-in:** Difficult to port to C++, mobile platforms (iOS/Android), or browsers.

---

## 3. Exporting YOLO11n to ONNX

**ONNX (Open Neural Network Exchange)** is an open industry standard that freezes your neural network into a hardware-agnostic computation graph.

```python
from ultralytics import YOLO

model = YOLO("best.pt")

# Export to ONNX format
# simplify=True optimizes graph nodes, removing redundant operations
onnx_path = model.export(format="onnx", imgsz=640, simplify=True)
print(f"Exported model to: {onnx_path}")  # Creates 'best.onnx' (~5.5 MB)
```

---

## 4. Running High-Speed Inference with ONNX Runtime

**ONNX Runtime (`onnxruntime`)** is a lightweight engine ($<50\text{ MB}$ installation) that runs $2\times$ to $3\times$ faster on standard Intel/AMD CPUs.

```bash
pip install onnxruntime
```

```python
import cv2
import numpy as np
import onnxruntime as ort

# 1. Initialize ONNX Runtime session
session = ort.InferenceSession("best.onnx", providers=['CPUExecutionProvider'])
input_name = session.get_inputs()[0].name

# 2. Preprocess input image to (1, 3, 640, 640) float32
img = cv2.imread("test.jpg")
resized = cv2.resize(img, (640, 640))
input_tensor = resized.transpose(2, 0, 1).astype(np.float32) / 255.0
input_tensor = np.expand_dims(input_tensor, axis=0)

# 3. Blazing fast inference without PyTorch!
outputs = session.run(None, {input_name: input_tensor})
# Output shape: (1, 4 + num_classes, 8400 candidate anchors)
```

---

## 5. Performance Comparison

| Deployment Runtime | Framework Size | CPU Latency (YOLO11n) | FPS on Intel i5 CPU |
|---|---|---|---|
| **PyTorch (`.pt`)** | ~2,500 MB | ~35 ms | ~28 FPS |
| **ONNX Runtime (`.onnx`)** | ~60 MB | **~16 ms** | **~62 FPS** |
| **TensorRT (NVIDIA GPU)** | ~800 MB | **~1.2 ms** | **>800 FPS** |
