# Understanding mAP (Mean Average Precision) — The Gold Standard Metric

## 1. Why Standard Accuracy Doesn't Work for Detection

In image classification, accuracy is simple: did the model predict the right class label? Yes or No.

In object detection, an image might have:
- 3 people, 2 cars, and 1 traffic light.
- The model might detect 2 people accurately, miss the 3rd person, predict 1 ghost car, and place the traffic light box 5 pixels too far to the left.

You cannot evaluate this with a simple percentage. We need **Precision, Recall, and mAP**.

---

## 2. What Counts as a True Positive (TP) in Detection?

A prediction is a **True Positive (TP)** if and only if:
1. **Class Match:** The predicted class matches the ground-truth class.
2. **IoU Threshold:** The IoU between the predicted bounding box and ground-truth box is $\ge \text{IoU Threshold}$ (typically $0.5$).
3. **No Duplicate:** It is the first prediction matched to that ground truth (duplicates are counted as False Positives).

```
Precision = TP / (TP + FP)  ──► "When the model claims an object is there, how often is it right?"
Recall    = TP / (TP + FN)  ──► "Out of all real objects in the world, what percentage did it find?"
```

---

## 3. From Precision-Recall Curve to AP (Average Precision)

As you lower the confidence threshold from $0.99$ to $0.01$:
- **Recall goes up** (the model finds more objects).
- **Precision goes down** (the model makes more noisy false-positive guesses).

Plotting Precision against Recall gives the **PR Curve**:

```
Precision
  1.0 ┌──────────┐
      │          \
      │           \
      │            \
      │             \
  0.0 └──────────────┴────► Recall
     0.0             1.0

  Area Under Curve = Average Precision (AP for Class)
```

**Average Precision (AP)** is the area under the PR curve for a **single class** (e.g., AP for "Car").

---

## 4. What is mAP (mean Average Precision)?

**mAP** is the arithmetic mean of the Average Precision across all $C$ object classes:

$$\mathbf{\text{mAP} = \frac{1}{C} \sum_{i=1}^{C} \text{AP}_i}$$

---

## 5. Decoding the Numbers: `mAP@50` vs `mAP@50-95`

When reading **YOLO11n** training logs, you will see two primary metrics:

### 1. `mAP@50` (or `mAP@0.50`)
- Evaluates detections using a relaxed **$\text{IoU} \ge 0.50$** threshold.
- Rewards the model if it identifies the general area of an object.
- Typically higher (e.g., **$\approx 85\text{--}92\%$**).

### 2. `mAP@50-95` (or `mAP@[0.5:0.95]`)
- The official, strict benchmark used by the COCO competition.
- Averages mAP across **10 different IoU thresholds**:
  $$\text{IoU} \in [0.50, 0.55, 0.60, 0.65, 0.70, 0.75, 0.80, 0.85, 0.90, 0.95]$$
- Penalizes sloppy, loose bounding boxes.
- Typically lower (e.g., **$\approx 38\text{--}52\%$** for lightweight models like YOLO11n).

---

## 6. Fresher Interview Question

**Q: A client wants to detect cracks on airplane wings. Should you optimize your YOLO model for high Precision or high Recall?**
> **Answer:**
> **High Recall**. Missing a real crack (False Negative) could cause a catastrophic plane crash. Having a few false alarms (False Positives) that an engineer double-checks is acceptable, but missing a real defect is fatal.
