# What Is a Digital Image? — From Pixels to Numbers

## 1. What is it?

A digital image is **not a photo to a computer** — it is a **two-dimensional grid (matrix) of numbers**.

- Every single point in the grid is called a **pixel** (short for *picture element*).
- Each pixel contains a numeric value representing brightness or color intensity.
- In standard 8-bit images, pixel values range from **0 to 255** ($2^8 = 256$ possible values):
  - **0** = Pure Black (no light)
  - **255** = Pure White (maximum light)

---

## 2. Why does this matter?

All computer vision algorithms — whether a simple blur filter, an OCR engine, or **YOLO11n** — are just **matrix mathematics performed on these numbers**.

If you change the numbers, you change the image:
- Adding 50 to all pixels makes the image brighter.
- Multiplying by 1.5 increases the contrast.
- Setting pixels below 128 to 0 and above 128 to 255 creates a high-contrast binary mask.

---

## 3. Grayscale vs Color Images

```
Grayscale Image (2D Matrix: Height x Width)
┌─────┬─────┬─────┐
│  12 │ 240 │ 180 │
├─────┼─────┼─────┤
│  50 │  95 │ 210 │   Shape: (H, W) -> e.g., (480, 640)
├─────┼─────┼─────┤
│ 255 │   0 │ 128 │
└─────┴─────┴─────┘

Color Image (3D Tensor: Height x Width x 3 Channels)
      Red Channel       Green Channel       Blue Channel
     ┌─────┬─────┐     ┌─────┬─────┐     ┌─────┬─────┐
    / 255 /  40 /     /   0 / 180 /     / 120 /  50 /
   ┌─────┬─────┐     ┌─────┬─────┐     ┌─────┬─────┐
  │ 180 │  90 │     │ 210 │  30 │     │   0 │ 255 │
  └─────┴─────┘     └─────┴─────┘     └─────┴─────┘
```

- **Grayscale (1 Channel):** Each pixel is a single number ($0\text{–}255$).
- **Color (3 Channels):** Every pixel is a triplet of three numbers representing color intensities. By mixing different amounts of Red, Green, and Blue, we can represent $256 \times 256 \times 256 \approx 16.7\text{ million}$ colors.

---

## 4. The Coordinate System Trap (Row/Col vs X/Y)

This is one of the most common mistakes freshers make in OpenCV and CV interviews!

```
        X (Width / Columns) ───►  0 to W-1
      ┌─────────────────────────────┐
  Y   │ (0,0)                       │
(H /  │                             │
Rows) │                             │
  │   │                             │
  ▼   │                      (W, H) │
      └─────────────────────────────┘
```

- **OpenCV / Math coordinates:** Point $(X, Y)$ = `(column, row)` = `(width_offset, height_offset)`.
- **NumPy array indexing:** `image[row, col]` = `image[Y, X]`.

> [!WARNING]
> If you want to access a pixel at coordinate $(x=100, y=50)$, in NumPy you must write:
> `pixel = image[50, 100]` (**row first, then column**).
> If you write `image[100, 50]`, you have swapped $X$ and $Y$!

---

## 5. Python & NumPy Verification

```python
import numpy as np

# Create a blank black grayscale image of size 3x3
gray = np.zeros((3, 3), dtype=np.uint8)

# Set the center pixel to pure white
gray[1, 1] = 255

print("Grayscale image matrix:\n", gray)
print("Shape:", gray.shape)  # Output: (3, 3)

# Create a red pixel in a 3-channel image (Height=1, Width=1, Channels=3)
# In RGB: Red = [255, 0, 0]
color_pixel = np.array([[[255, 0, 0]]], dtype=np.uint8)
print("Color pixel shape:", color_pixel.shape)  # Output: (1, 1, 3)
```

---

## 6. Fresher Interview Questions

**Q1: What does `uint8` mean in Computer Vision?**
> **Answer:** `uint8` stands for Unsigned 8-bit Integer. It stores numbers from $0$ to $255$ ($2^8 - 1$). Since human eyes perceive color gradations well in 256 steps and it uses exactly 1 byte per channel, `uint8` is the standard memory format for digital images.

**Q2: What happens if you add 10 to a pixel with value 250 in `uint8`?**
> **Answer:** In standard NumPy, integer overflow occurs: `250 + 10 = 260 % 256 = 4` (it wraps around to dark!). However, OpenCV's `cv2.add()` uses saturation arithmetic: `cv2.add(250, 10) = 255` (it caps at maximum white).
