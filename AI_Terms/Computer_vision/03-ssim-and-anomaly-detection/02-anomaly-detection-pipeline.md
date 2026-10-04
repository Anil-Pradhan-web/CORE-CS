# The End-to-End SSIM Anomaly Detection Pipeline

## 1. The High-Level Architecture

In an industrial quality inspection setting (e.g., PCB inspection, pharmaceutical packaging, metal sheet manufacturing), you rarely have thousands of defect images to train a deep neural network.

Instead, we use a **reference-based SSIM pipeline**:

```
┌────────────────────────┐      ┌────────────────────────┐
│ Baseline "Golden" Image│      │ Incoming Test Sample   │
└───────────┬────────────┘      └───────────┬────────────┘
            │                               │
            └───────────────┬───────────────┘
                            ▼
               [Step 1: Grayscale & Denoise]
                            │
                            ▼
               [Step 2: Alignment / Registration]
                            │
                            ▼
               [Step 3: SSIM Difference Map]
                            │
                            ▼
               [Step 4: Thresholding Difference]
                            │
                            ▼
               [Step 5: Morphological Cleanup]
                            │
                            ▼
               [Step 6: Contour Extraction]
                            │
                            ▼
               [Step 7: Defect Filtering & Bounding Boxes]
                            │
                            ▼
               [Decision: PASS or FAIL]
```

---

## 2. Step-by-Step Breakdown

### Step 1: Grayscale & Denoising
Convert both the golden reference and test image to grayscale. Apply a slight Gaussian blur ($3 \times 3$) to eliminate sensor pixel noise.

### Step 2: Alignment / Registration (Crucial in Real Factories!)
If the physical conveyor belt vibrates or places the component 2 millimeters off-center, you must align the test image to the golden image using **Homography** (ORB feature matching).
*(Without alignment, the entire outer edge will be flagged as a defect).*

### Step 3: Compute SSIM & Difference Map
The `structural_similarity` function returns:
1. `score`: A single float (e.g., $0.942$) representing global image similarity.
2. `diff`: A 2D floating-point array of shape `(H, W)` with values in $[-1.0, 1.0]$.
   - Converting to 8-bit image: `diff = (diff * 255).astype("uint8")`.

### Step 4: Binary Thresholding
Regions with high structural difference will appear dark in the difference map.
Applying an inverted binary threshold (`cv2.THRESH_BINARY_INV | cv2.THRESH_OTSU`) flips the defect regions into **bright white ($255$)**, while flawless regions become **black ($0$)**.

### Step 5: Morphological Operations
Apply morphological **Opening** (removes dust specks) followed by **Closing** (bridges cracked segments together into a single continuous defect shape).

### Step 6 & 7: Contour Detection & Area Filtering
Extract contours. Any contour with an area larger than a predefined threshold (e.g., $\text{area} > 40\text{ pixels}$) is marked with a red bounding box and tagged as an **Anomaly**.

### Step 8: Pass/Fail Logic
```python
if total_defects == 0 and ssim_score > 0.95:
    status = "PASS"
else:
    status = "FAIL (Rejected)"
```

---

## 3. Why This Project Shines on a Fresher's Resume

- Demonstrates practical understanding of **unsupervised anomaly detection** (no manual labeling needed).
- Shows mastery of combining **mathematical algorithms (SSIM)** with **OpenCV morphological pipelines**.
- Shows awareness of real-world constraints (conveyor vibration, lighting variations, false positive thresholds).
