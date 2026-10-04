# Thresholding & Edge Detection — Finding Boundaries

## 1. What is Thresholding?

**Thresholding** converts a grayscale image into a high-contrast binary image (pure black $0$ and pure white $255$).

It is used to separate the foreground (text in **OCR**, defect regions in **SSIM**) from the background.

---

## 2. The 3 Types of Thresholding

### 1. Simple Global Thresholding
Applies a single fixed cutoff value across every pixel in the entire image.
```python
# Pixels > 127 become 255; others become 0
ret, thresh = cv2.threshold(gray, 127, 255, cv2.THRESH_BINARY)
```
*Limitation: Fails completely if one side of the image has shadows or uneven lighting.*

### 2. Otsu's Thresholding (Bimodal Specialist)
Instead of guessing the threshold number, **Otsu's algorithm automatically calculates the optimal threshold** by analyzing the image histogram and maximizing the variance between two pixel clusters (foreground vs background).
```python
# Pass 0 as the threshold value; add cv2.THRESH_OTSU flag
ret, otsu_thresh = cv2.threshold(gray, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)
print("Otsu calculated optimal threshold:", ret)
```

### 3. Adaptive Thresholding (The OCR Hero)
Calculates a different threshold for small local neighborhoods ($11 \times 11$ blocks) across the image.
```python
adaptive = cv2.adaptiveThreshold(
    gray,
    maxValue=255,
    adaptiveMethod=cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
    thresholdType=cv2.THRESH_BINARY,
    blockSize=11,  # Size of pixel neighborhood (must be odd)
    C=2            # Constant subtracted from the mean
)
```
> [!TIP]
> **Why Adaptive Thresholding is essential for OCR:**
> When a user photographs a document, there are almost always shadow gradients (e.g., hand shadow or lamp on one side). Global thresholding turns the shaded half into pure black. Adaptive thresholding calculates localized contrast, making text crisp even inside deep shadows.

---

## 3. What is an Edge? (Image Gradients)

An **edge** is a sudden, sharp change in pixel intensity (brightness).

In mathematics, a sharp rate of change is a **derivative (gradient)**:
$$\nabla I = \left[ \frac{\partial I}{\partial x}, \frac{\partial I}{\partial y} \right]$$

- **Sobel Operator:** Computes directional gradients using $3 \times 3$ kernels:
  - $Sobel_X$: Detects vertical edges (horizontal changes).
  - $Sobel_Y$: Detects horizontal edges (vertical changes).

```python
sobelx = cv2.Sobel(gray, cv2.CV_64F, 1, 0, ksize=3)
sobely = cv2.Sobel(gray, cv2.CV_64F, 0, 1, ksize=3)
gradient_magnitude = cv2.magnitude(sobelx, sobely)
```

---

## 4. Canny Edge Detection — The 4-Step Algorithm

Developed by John Canny in 1986, the **Canny Edge Detector** is the most widely used edge detection algorithm because it produces clean, single-pixel-wide edges.

```
Input Image ──► [1. Gaussian Blur] ──► [2. Sobel Gradients] ──► [3. Non-Max Suppression] ──► [4. Hysteresis] ──► Edges
```

### The 4 Internal Steps (Guaranteed Interview Question):
1. **Gaussian Filtering:** Smooths image to eliminate noise.
2. **Gradient Calculation:** Finds intensity gradients ($G_x, G_y$) and edge directions ($\theta = \arctan(G_y / G_x)$).
3. **Non-Maximum Suppression (NMS):** Thinning step. Looks along the gradient direction; if the current pixel is not the local maximum, it is suppressed to $0$. This reduces thick blurry edges to crisp 1-pixel lines.
4. **Hysteresis Thresholding:** Uses two thresholds ($T_{\text{lower}}$ and $T_{\text{upper}}$):
   - Pixel gradient $> T_{\text{upper}}$ $\rightarrow$ **Strong Edge** (definitely kept).
   - Pixel gradient $< T_{\text{lower}}$ $\rightarrow$ **Suppressed to 0** (discarded as noise).
   - Pixel between $T_{\text{lower}}$ and $T_{\text{upper}}$ $\rightarrow$ **Weak Edge** (kept *only* if connected to a strong edge!).

### Code Implementation:

```python
import cv2

# Threshold1 = 50 (lower), Threshold2 = 150 (upper)
# Standard rule of thumb: ratio is usually 1:2 or 1:3
edges = cv2.Canny(gray, threshold1=50, threshold2=150)
```
