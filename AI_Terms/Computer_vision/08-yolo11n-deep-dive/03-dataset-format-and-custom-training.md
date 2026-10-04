# Dataset Format & Custom Training with YOLO11n

## 1. The Standard YOLO Directory Structure

Ultralytics YOLO expects your dataset organized in a strict folder layout with matching base names for images and label text files:

```
my_custom_dataset/
├── data.yaml                # Dataset configuration file
├── images/
│   ├── train/               # Training images (.jpg, .png)
│   │   ├── img001.jpg
│   │   └── img002.jpg
│   └── val/                 # Validation images
│       └── img101.jpg
└── labels/
    ├── train/               # Matching label files (.txt)
    │   ├── img001.txt
    │   └── img002.txt
    └── val/
        └── img101.txt
```

> [!IMPORTANT]
> The image file and its label file **must share the exact same basename**.
> If your image is `images/train/car_01.jpg`, its label must be `labels/train/car_01.txt`.

---

## 2. The YOLO Label Annotation Format

Each line in the `.txt` label file represents one detected object:

$$\mathbf{<\text{class\_id}>\quad <x_{\text{center}}>\quad <y_{\text{center}}>\quad <\text{width}>\quad <\text{height}>}$$

- **`class_id`**: Integer index from $0$ to $(C-1)$ (e.g., $0$ for "Helmet", $1$ for "Vest").
- All coordinates are **floats normalized between $0.0$ and $1.0$**:
  - $x_{\text{center}} = \frac{X_{\text{box center}}}{\text{Image Width}}$
  - $y_{\text{center}} = \frac{Y_{\text{box center}}}{\text{Image Height}}$
  - $\text{width} = \frac{\text{Box Width}}{\text{Image Width}}$
  - $\text{height} = \frac{\text{Box Height}}{\text{Image Height}}$

### Example `img001.txt`:
```
0 0.4521 0.6210 0.2104 0.3541
1 0.7812 0.3421 0.1542 0.2215
```

---

## 3. The `data.yaml` Configuration File

```yaml
# Path to your dataset root folder
path: C:/Users/ANIL/Desktop/my_custom_dataset

# Relative paths to train and val image directories
train: images/train
val: images/val

# Class definitions
names:
  0: Defect_Scratch
  1: Defect_Crack
  2: Normal_Part
```

---

## 4. Training YOLO11n in Python

```python
from ultralytics import YOLO

# 1. Load the pretrained YOLO11n base model (Transfer Learning)
model = YOLO("yolo11n.pt")

# 2. Train on your custom dataset
results = model.train(
    data="data.yaml",      # Path to dataset config
    epochs=50,             # Number of training epochs
    imgsz=640,             # Input image resolution
    batch=16,              # Batch size (reduce to 8 if GPU VRAM is low)
    device=0,              # GPU device ID (0) or 'cpu'
    patience=10,           # Early stopping: stops if no improvement for 10 epochs
    save=True,             # Save checkpoints
    project="runs/detect", # Output folder
    name="yolo11n_custom"  # Run name
)

# 3. Evaluate model performance on validation set
metrics = model.val()
print(f"Validation mAP@50: {metrics.box.map50 * 100:.2f}%")
print(f"Validation mAP@50-95: {metrics.box.map * 100:.2f}%")
```

---

## 5. What Losses Does YOLO11 Track During Training?

1. **`box_loss` (CIoU Loss):** Measures how accurately predicted bounding box coordinates enclose the target object.
2. **`cls_loss` (BCE Loss):** Binary Cross-Entropy measuring how accurately the model classifies each object.
3. **`dfl_loss` (Distribution Focal Loss):** Optimizes continuous bounding box boundaries to handle ambiguous or blurry edges.

After training completes, the best weights are automatically saved to:
`runs/detect/yolo11n_custom/weights/best.pt`.
