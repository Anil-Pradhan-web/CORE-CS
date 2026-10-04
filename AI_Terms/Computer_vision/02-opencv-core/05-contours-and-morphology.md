# Contours & Morphological Operations

## 1. What are Morphological Operations?

Morphological operations are mathematical operations applied to **binary images** based on shapes (structuring elements / kernels).

They are used to clean up masks after thresholding — such as joining broken text characters for **OCR** or eliminating tiny false-positive specks in **SSIM Anomaly Detection**.

---

## 2. Core Morphological Operations

```python
import cv2
import numpy as np

# Create a 5x5 rectangular structuring element
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))
```

### 1. Erosion (`cv2.erode`)
- **What it does:** Erodes away the boundaries of foreground (white) objects.
- **Effect:** Shrinks objects; completely wipes out tiny white noise dots.

### 2. Dilation (`cv2.dilate`)
- **What it does:** Expands the boundaries of foreground objects.
- **Effect:** Enlarges objects; closes small black holes and bridges broken letters.

```
Original Mask              After Erosion (Shrunk)       After Dilation (Expanded)
  ██████                     ████                         ████████
  ██████                     ████                         ████████
  ██████  . (noise dot)                                   ████████
```

### 3. Opening (`cv2.MORPH_OPEN`)
- **Formula:** **Erosion followed by Dilation**.
- **Best for:** Eliminating background noise without shrinking real objects.

### 4. Closing (`cv2.MORPH_CLOSE`)
- **Formula:** **Dilation followed by Erosion**.
- **Best for:** Closing small holes or gaps inside detected objects and joining broken text strokes.

```python
# Clean up noise specks
clean_mask = cv2.morphologyEx(binary_mask, cv2.MORPH_OPEN, kernel)

# Bridge broken lines or fill holes
solid_mask = cv2.morphologyEx(clean_mask, cv2.MORPH_CLOSE, kernel)
```

---

## 3. Contours — Finding and Measuring Objects

A **contour** is a continuous line joining all the boundary points along a shape that have the same color or intensity.

```python
# Always run findContours on a binary image (thresholded or Canny edge output)
# cv2.RETR_EXTERNAL: retrieves only extreme outer contours (ignores nested holes)
# cv2.CHAIN_APPROX_SIMPLE: compresses horizontal, vertical, and diagonal segments (saves memory)
contours, hierarchy = cv2.findContours(
    binary_mask, 
    cv2.RETR_EXTERNAL, 
    cv2.CHAIN_APPROX_SIMPLE
)

print(f"Total objects detected: {len(contours)}")
```

---

## 4. Extracting Shape Properties (The Defect Detection Pipeline)

In anomaly detection and object counting, we filter contours by geometric properties:

```python
for cnt in contours:
    # 1. Calculate contour area in pixels
    area = cv2.contourArea(cnt)
    
    # Ignore tiny noise artifacts below 100 pixels
    if area < 100:
        continue
        
    # 2. Get straight bounding rectangle: [x, y, width, height]
    x, y, w, h = cv2.boundingRect(cnt)
    
    # 3. Draw bounding box around detected object/defect
    cv2.rectangle(img, (x, y), (x + w, y + h), (0, 0, 255), 2)
    
    # 4. Optional: Minimum area rotated bounding box (for tilted objects)
    rect = cv2.minAreaRect(cnt)
    box = cv2.boxPoints(rect)
    box = np.int0(box)
    cv2.drawContours(img, [box], 0, (255, 0, 0), 2)
```

---

## 5. Fresher Interview Tip: `CHAIN_APPROX_NONE` vs `CHAIN_APPROX_SIMPLE`

> [!NOTE]
> - `CHAIN_APPROX_NONE`: Stores **all** contour boundary points. For a rectangle with a 400-pixel perimeter, it stores 400 $(X, Y)$ points.
> - `CHAIN_APPROX_SIMPLE`: Compresses redundant points. For the same rectangle, it stores **only the 4 corner points**, reducing memory usage by 99%! Always use `CHAIN_APPROX_SIMPLE` unless you need every sub-pixel curve point.
