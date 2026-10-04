# Intersection over Union (IoU) & Non-Maximum Suppression (NMS)

## 1. What is IoU (Intersection over Union)?

**IoU** is the standard mathematical metric used to evaluate how accurately a predicted bounding box matches the ground truth annotation.

$$\mathbf{\text{IoU} = \frac{\text{Area of Overlap}}{\text{Area of Union}} = \frac{A \cap B}{A \cup B}}$$

```
     Box A (Ground Truth)
     ┌─────────────┐
     │             │
     │      ┌──────┼──────┐  <-- Box B (Prediction)
     │      │//////│      │
     └──────┼──────┘      │
            │             │
            └─────────────┘

    Intersection (////)               Union
    [ Overlapping Area ]       [ Total Combined Footprint ]
```

### IoU Threshold Benchmarks:
- **$\text{IoU} = 1.0$**: Perfect pixel-for-pixel match.
- **$\text{IoU} \ge 0.50$**: Standard benchmark for a "True Positive" detection in PASCAL VOC.
- **$\text{IoU} \ge 0.75$**: High-precision match (used in strict benchmarks like COCO).
- **$\text{IoU} < 0.50$**: Counted as a localization error (False Positive).

---

## 2. Why Do We Need NMS (Non-Maximum Suppression)?

When an object detector scans an image, multiple nearby grid cells and anchors all detect the same object with slightly different coordinates.

If you don't clean them up, **a single car gets 15 overlapping bounding boxes drawn on top of it**!

```
Raw Model Output (15 Boxes for 1 Car) ──► [ NMS Algorithm ] ──► Clean Final Output (1 Box)
```

---

## 3. The NMS Algorithm (Step-by-Step)

1. **Confidence Filtering:** Discard all boxes whose confidence score is below a minimum cutoff (e.g., $\text{conf} < 0.25$).
2. **Sort:** Sort the remaining candidate boxes in descending order of confidence scores.
3. **Pick Winner:** Select the box with the highest confidence and add it to the final detection list.
4. **Suppress Overlaps:** Compare this winner box against all remaining boxes. If the $\text{IoU} > \text{NMS Threshold}$ (e.g., $0.45$), that candidate is considered a duplicate prediction and **suppressed (deleted)**.
5. **Repeat:** Repeat steps 3 and 4 for the remaining boxes until no boxes are left.

---

## 4. Python Implementation of IoU and NMS

```python
import numpy as np

def compute_iou(box1, box2):
    # box format: [x1, y1, x2, y2]
    xA = max(box1[0], box2[0])
    yA = max(box1[1], box2[1])
    xB = min(box1[2], box2[2])
    yB = min(box1[3], box2[3])

    intersection = max(0, xB - xA) * max(0, yB - yA)
    area1 = (box1[2] - box1[0]) * (box1[3] - box1[1])
    area2 = (box2[2] - box2[0]) * (box2[3] - box2[1])
    union = area1 + area2 - intersection

    return intersection / (union + 1e-6)

def non_max_suppression(boxes, scores, iou_threshold=0.45):
    indices = np.argsort(scores)[::-1]  # Sort descending
    selected = []

    while len(indices) > 0:
        current = indices[0]
        selected.append(current)

        if len(indices) == 1:
            break

        # Compute IoU between highest score box and all other remaining boxes
        ious = np.array([compute_iou(boxes[current], boxes[i]) for i in indices[1:]])
        
        # Keep boxes with IoU less than threshold
        remaining = np.where(ious < iou_threshold)[0]
        indices = indices[remaining + 1]

    return selected
```

> [!NOTE]
> In real applications, OpenCV provides a highly optimized C++ implementation:
> `indices = cv2.dnn.NMSBoxes(boxes, scores, score_threshold=0.3, nms_threshold=0.45)`
