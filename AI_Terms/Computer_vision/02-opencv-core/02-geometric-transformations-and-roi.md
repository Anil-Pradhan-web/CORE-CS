# Geometric Transformations & Region of Interest (ROI)

## 1. What are Geometric Transformations?

Geometric transformations alter the spatial orientation, size, or perspective of an image without changing its underlying content.

Common use cases:
- Preprocessing images to a fixed input size for **YOLO11n** ($640 \times 640$).
- Rotating tilted scanned pages for **OCR**.
- Isolating a Region of Interest (**ROI**) to inspect defects in **SSIM Anomaly Detection**.

---

## 2. Image Resizing & The Interpolation Trap

```python
import cv2

# Method 1: Resize to exact dimensions (Width=640, Height=640)
# Notice: OpenCV takes (width, height), NOT (height, width)!
resized = cv2.resize(img, (640, 640), interpolation=cv2.INTER_LINEAR)

# Method 2: Resize by scaling factors (e.g., 50% scale)
half_sized = cv2.resize(img, (0, 0), fx=0.5, fy=0.5, interpolation=cv2.INTER_AREA)
```

### Which Interpolation flag should you use? (Interview Goldmine)

| Interpolation Flag | When to Use | Why |
|---|---|---|
| `cv2.INTER_AREA` | **Downsampling (Shrinking)** | Computes pixel area relations; avoids moiré patterns and aliasing. |
| `cv2.INTER_LINEAR` | **Upsampling (Enlarging)** | Bilinear interpolation; fast, smooth, default choice. |
| `cv2.INTER_CUBIC` | **Upsampling (High Quality)** | Bicubic interpolation over $4 \times 4$ neighborhood; sharper but slower. |
| `cv2.INTER_NEAREST` | **Segmentation Masks** | Preserves exact integer class IDs without creating intermediate float values. |

---

## 3. Region of Interest (ROI)

An **ROI** is a sub-region of an image that you crop out for focused processing.

```
       (x1, y1)
          ┌────────────────┐
          │  Target Object │
          │     (ROI)      │
          └────────────────┘
                       (x2, y2)
```

```python
# Crop bounding box: x1=120, y1=80, x2=340, y2=260
# In NumPy: [y_min : y_max, x_min : x_max]
roi = img[80:260, 120:340]

# Run OCR or Anomaly Detection strictly on the ROI!
# This speeds up execution by 5x-10x compared to processing the whole 4K image.
```

---

## 4. Image Rotation (Warp Affine)

To rotate an image around an arbitrary center point:

```python
import cv2

height, width = img.shape[:2]
center = (width // 2, height // 2)

# 1. Generate 2D rotation matrix: (center, angle_in_degrees, scale)
# Angle: positive = counter-clockwise, negative = clockwise
rotation_matrix = cv2.getRotationMatrix2D(center=center, angle=45, scale=1.0)

# 2. Apply affine transformation
rotated_img = cv2.warpAffine(img, rotation_matrix, (width, height))
```

---

## 5. Aspect Ratio Preservation (Letterboxing)

If you directly squeeze a $1920 \times 1080$ widescreen image into a $640 \times 640$ square, objects become unnaturally tall and thin, reducing detection accuracy.

**Letterboxing** resizes the image while maintaining its aspect ratio, padding the remaining borders with grey or black pixels.

```
Direct Squish (Bad)          Letterboxing (Good)
┌──────────────┐             ┌──────────────┐
│ (Stretched)  │             │ ░░░░░░░░░░░░ │  <-- Top Padding
│    /\        │             ├──────────────┤
│   |  |       │             │   Original   │
│    \/        │             │ Aspect Ratio │
│              │             ├──────────────┤
└──────────────┘             │ ░░░░░░░░░░░░ │  <-- Bottom Padding
                             └──────────────┘
```

> [!NOTE]
> Ultralytics YOLO models (including **YOLO11n**) automatically apply letterboxing during preprocessing so you don't lose object aspect ratios.
