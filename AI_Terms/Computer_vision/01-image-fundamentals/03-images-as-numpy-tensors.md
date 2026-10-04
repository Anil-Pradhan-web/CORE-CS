# Images as NumPy Arrays & Deep Learning Tensors

## 1. What is an Image Tensor?

In Python, an image is simply a multi-dimensional array:
- In **OpenCV / NumPy**: an image is an `ndarray` of shape `(Height, Width, Channels)` or `(H, W, C)`.
- In **PyTorch / Deep Learning**: models expect tensors of shape `(Batch_Size, Channels, Height, Width)` or `(B, C, H, W)`.

Understanding this dimensional transition is the #1 prerequisite to feeding images into **YOLO11n** or any CNN.

---

## 2. Shape Conversion: OpenCV ➔ PyTorch / YOLO

```
OpenCV Image:
  Shape: (480, 640, 3)  ──► [Height, Width, Channels]
          │
          ▼  Step 1: Convert BGR to RGB
  Shape: (480, 640, 3)
          │
          ▼  Step 2: Transpose dimensions (H, W, C) ➔ (C, H, W)
  Shape: (3, 480, 640)
          │
          ▼  Step 3: Normalize pixel values (0-255 ➔ 0.0-1.0)
  Values: float32 in [0.0, 1.0]
          │
          ▼  Step 4: Add Batch dimension (unsqueeze)
  Shape: (1, 3, 480, 640)  ──► [Batch, Channels, Height, Width]
```

### PyTorch Transformation Code:

```python
import cv2
import numpy as np
import torch

# 1. Load image (OpenCV loads as BGR, shape: H x W x C)
img_bgr = cv2.imread("sample.jpg")

# 2. Convert BGR to RGB
img_rgb = cv2.cvtColor(img_bgr, cv2.COLOR_BGR2RGB)

# 3. Normalize to [0.0, 1.0] and convert dtype to float32
img_normalized = img_rgb.astype(np.float32) / 255.0

# 4. Transpose from (H, W, C) to (C, H, W)
img_transposed = np.transpose(img_normalized, (2, 0, 1))

# 5. Convert to PyTorch Tensor and add Batch dimension: (1, C, H, W)
tensor_input = torch.from_numpy(img_transposed).unsqueeze(0)

print("Final Tensor Shape for Neural Network:", tensor_input.shape)
# Output: torch.Size([1, 3, 480, 640])
```

---

## 3. Practical NumPy Image Slicing (Cropping / ROI)

Because images are NumPy arrays, cropping is as fast as array slicing without memory duplication:

```python
# Crop a Region of Interest (ROI)
# Remember: image[y_min : y_max, x_min : x_max]
roi = image[100:300, 200:450]

# Replace that region with black
image[100:300, 200:450] = 0
```

---

## 4. Channel Splitting and Merging

```python
# Splitting channels into individual 2D arrays
b, g, r = cv2.split(image_bgr)
# Each channel has shape (H, W)

# Re-merging channels
merged = cv2.merge([b, g, r])
```

> [!TIP]
> `cv2.split()` is relatively slow because it allocates new memory. In performance-critical video loops, use NumPy indexing:
> `blue_channel = image_bgr[:, :, 0]`

---

## 5. Why do Neural Networks require Normalization?

When pixel values are between $0$ and $255$, gradients during backpropagation can fluctuate wildly because input values are large.

By dividing by $255.0$ (scaling to $[0.0, 1.0]$) or standardizing with ImageNet mean and std:
$$\text{normalized} = \frac{\text{pixel} - \text{mean}}{\text{std}}$$
1. Optimization converges significantly faster.
2. Activation functions like Sigmoid or ReLU do not saturate immediately.
3. Neural network weights remain numerically stable.
