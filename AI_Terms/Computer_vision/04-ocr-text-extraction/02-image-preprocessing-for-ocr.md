# Image Preprocessing for OCR — The Accuracy Multiplier

## 1. Why Preprocessing is 70% of OCR Success

If you pass a tilted, low-resolution, shadowy photo of an invoice directly into Tesseract or EasyOCR, the engine outputs garbage characters (e.g., `|NV01CE # ?82-X`).

OCR models are trained on clean, high-contrast, horizontally aligned text. **OpenCV preprocessing bridges the gap between chaotic real-world photos and pristine OCR input.**

---

## 2. Key Preprocessing Stages

```
   Raw Input Photo (Shadows, Tilted)
                  │
                  ▼
   1. Grayscale Conversion (Drops color noise)
                  │
                  ▼
   2. Bilateral / Median Denoising (Removes paper grain, preserves character strokes)
                  │
                  ▼
   3. Adaptive Gaussian Binarization (Handles uneven lighting / shadows)
                  │
                  ▼
   4. Deskewing (Rotates text lines horizontally)
                  │
                  ▼
   Clean High-Contrast Input for OCR Engine
```

---

## 3. Deskewing: How to Fix Tilted Documents

When someone scans or photos a document, it is frequently rotated by $2^\circ$ to $15^\circ$. OCR word detectors struggle with diagonal lines.

### Deskewing Algorithm:
1. Find all foreground text pixel coordinates using `np.where(thresh > 0)`.
2. Compute the minimum area bounding box enclosing the text using `cv2.minAreaRect()`.
3. Extract the rotation angle $\theta$.
4. Rotate the image using `cv2.getRotationMatrix2D` and `cv2.warpAffine`.

---

## 4. Complete Production Preprocessing Function

```python
import cv2
import numpy as np

def preprocess_image_for_ocr(image_path):
    # 1. Load image
    image = cv2.imread(image_path)
    if image is None:
        raise FileNotFoundError("Image not found!")

    # 2. Resize if image is too low-resolution (OCR needs ~300 DPI / clear font heights)
    h, w = image.shape[:2]
    if w < 1000:
        scale = 1000 / w
        image = cv2.resize(image, (0, 0), fx=scale, fy=scale, interpolation=cv2.INTER_CUBIC)

    # 3. Convert to Grayscale
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

    # 4. Denoise: Bilateral filter preserves sharp text boundaries while removing paper texture
    denoised = cv2.bilateralFilter(gray, d=9, sigmaColor=75, sigmaSpace=75)

    # 5. Adaptive Thresholding to handle shadows and uneven room lighting
    binary = cv2.adaptiveThreshold(
        denoised, 
        255, 
        cv2.ADAPTIVE_THRESH_GAUSSIAN_C, 
        cv2.THRESH_BINARY, 
        15, 
        4
    )

    # 6. Deskewing: Determine angle of text
    coords = np.column_stack(np.where(binary == 0))  # Black text pixel positions
    if len(coords) > 0:
        angle = cv2.minAreaRect(coords)[-1]
        
        # Adjust angle logic for OpenCV minAreaRect conventions
        if angle < -45:
            angle = -(90 + angle)
        elif angle > 45:
            angle = 90 - angle
        else:
            angle = -angle

        # If significant tilt detected, rotate back
        if abs(angle) > 0.5:
            (h, w) = binary.shape[:2]
            center = (w // 2, h // 2)
            M = cv2.getRotationMatrix2D(center, angle, 1.0)
            binary = cv2.warpAffine(
                binary, 
                M, 
                (w, h), 
                flags=cv2.INTER_CUBIC, 
                borderMode=cv2.BORDER_REPLICATE
            )
            print(f"[INFO] Deskewed document by {angle:.2f} degrees")

    return binary
```

---

## 5. Summary: Preprocessing Impact

| Problem in Raw Image | OpenCV Preprocessing Fix | Resulting Benefit |
|---|---|---|
| Deep shadow across document | `cv2.adaptiveThreshold` | Background remains pure white, letters remain crisp black. |
| Grainy paper texture | `cv2.bilateralFilter` | Noise is averaged out without blurring sharp ink boundaries. |
| Crooked / Tilted document | `cv2.minAreaRect` + `cv2.warpAffine` | Text lines are aligned horizontally, boosting accuracy. |
| Tiny pixelated text | `cv2.resize` with `cv2.INTER_CUBIC` | Interpolated letters provide clear character outlines. |
