# Python Implementation: SSIM Defect Detection

## 1. Prerequisites

Make sure you have `opencv-python` and `scikit-image` installed:
```bash
pip install opencv-python scikit-image numpy
```

---

## 2. Complete Runnable Python Script

You can run this self-contained script immediately — it synthesizes a golden reference and a defective sample image to demonstrate the pipeline end-to-end.

```python
import cv2
import numpy as np
from skimage.metrics import structural_similarity as ssim

def detect_anomalies(golden_img_path, test_img_path, min_defect_area=50):
    # 1. Load images
    img_golden = cv2.imread(golden_img_path)
    img_test = cv2.imread(test_img_path)

    if img_golden is None or img_test is None:
        raise ValueError("Could not load one or both images!")

    # Ensure identical dimensions
    if img_golden.shape != img_test.shape:
        img_test = cv2.resize(img_test, (img_golden.shape[1], img_golden.shape[0]))

    # 2. Convert to grayscale
    gray_golden = cv2.cvtColor(img_golden, cv2.COLOR_BGR2GRAY)
    gray_test = cv2.cvtColor(img_test, cv2.COLOR_BGR2GRAY)

    # 3. Compute SSIM score and pixel-wise difference map
    # full=True returns the per-pixel difference matrix
    score, diff = ssim(gray_golden, gray_test, full=True)
    print(f"[INFO] Global SSIM Similarity Score: {score * 100:.2f}%")

    # 4. Convert difference map to 8-bit image [0, 255]
    diff = (diff * 255).astype("uint8")

    # 5. Threshold the difference map using Otsu's thresholding
    # THRESH_BINARY_INV flips defect areas to pure white (255)
    _, thresh = cv2.threshold(diff, 0, 255, cv2.THRESH_BINARY_INV | cv2.THRESH_OTSU)

    # 6. Morphological cleanup: remove small noise dots, close gaps
    kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))
    clean_mask = cv2.morphologyEx(thresh, cv2.MORPH_OPEN, kernel)
    clean_mask = cv2.morphologyEx(clean_mask, cv2.MORPH_CLOSE, kernel)

    # 7. Find contours of the defect regions
    contours, _ = cv2.findContours(clean_mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

    annotated_test = img_test.copy()
    defect_count = 0

    for cnt in contours:
        area = cv2.contourArea(cnt)
        if area > min_defect_area:
            defect_count += 1
            x, y, w, h = cv2.boundingRect(cnt)
            # Draw red bounding box around the defect
            cv2.rectangle(annotated_test, (x, y), (x + w, y + h), (0, 0, 255), 2)
            cv2.putText(
                annotated_test, 
                f"Defect #{defect_count}", 
                (x, y - 5), 
                cv2.FONT_HERSHEY_SIMPLEX, 
                0.5, 
                (0, 0, 255), 
                1
            )

    # Decision logic
    status = "PASS" if defect_count == 0 and score > 0.95 else f"FAIL ({defect_count} Defects)"
    print(f"[RESULT] Quality Check: {status}")

    return score, defect_count, clean_mask, annotated_test

# --- Self-Contained Test Demonstration ---
if __name__ == "__main__":
    # Create synthetic Golden reference image (Clean grey board with blue circle)
    golden = np.full((300, 300, 3), 200, dtype=np.uint8)
    cv2.circle(golden, (150, 150), 60, (255, 0, 0), -1)
    cv2.imwrite("golden_sample.png", golden)

    # Create synthetic Defective image (has a scratch/black crack)
    defective = golden.copy()
    cv2.line(defective, (120, 140), (180, 160), (0, 0, 0), 4)  # Scratch defect
    cv2.imwrite("defective_sample.png", defective)

    # Run detection
    score, defects, mask, result = detect_anomalies("golden_sample.png", "defective_sample.png")

    # Save results
    cv2.imwrite("inspection_result.png", result)
    print("[SUCCESS] Output saved as inspection_result.png")
```

---

## 3. How to Interpret the Output

- If `score >= 0.95` and `defect_count == 0`: **Product Passes QC**.
- If a scratch or defect is present:
  - `clean_mask` will show a solid white patch where the crack is.
  - `inspection_result.png` will feature a crisp red bounding box around the exact defect.
