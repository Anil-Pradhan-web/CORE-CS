# Convolution, Kernels, and Feature Maps

## 1. What is Convolution in Deep Learning?

A **convolutional layer** slides a small matrix of learnable weights — called a **filter** or **kernel** (e.g., $3 \times 3$) — across the input image.

At each position:
1. It computes an **element-wise multiplication** between the filter weights and the local pixel patch.
2. It **sums up all products** into a single number.
3. It adds a bias term and passes the result through an activation function (like ReLU).

```
Image Patch (3x3)          Kernel Weights (3x3)            Feature Map Output
┌───┬───┬───┐              ┌───┬───┬───┐
│ 1 │ 0 │ 1 │              │ 1 │ 0 │ 1 │
├───┼───┼───┤              ├───┼───┼───┤
│ 0 │ 1 │ 0 │       *      │ 0 │ 1 │ 0 │       =       (1*1 + 0*0 + 1*1 +
├───┼───┼───┤              ├───┼───┼───┤                0*0 + 1*1 + 0*0 +
│ 1 │ 0 │ 1 │              │ 1 │ 0 │ 1 │                1*1 + 0*0 + 1*1) = [ 4 ]
└───┴───┴───┘              └───┴───┴───┘
```

The resulting 2D output grid is called a **Feature Map** (or Activation Map).

---

## 2. The Power of Weight Sharing

In an MLP, every connection has its own independent weight.

In a CNN, **the exact same filter slides across the entire image**:
- A $3 \times 3$ filter on a grayscale image has **only 9 weights (+ 1 bias = 10 parameters)**!
- It doesn't matter whether the image is $28 \times 28$ or $4000 \times 3000$ pixels — the parameter count is still **10**.
- **Translation Equivariance:** Because the same filter looks everywhere, if a cat's ear appears in the top-left or bottom-right, the ear-detecting filter will activate and fire in that location of the feature map.

---

## 3. Multiple Filters = Multi-Channel Feature Maps

A single filter can only look for one specific pattern (e.g., vertical edges).
To detect diverse patterns, a convolutional layer uses a bank of multiple filters (e.g., 64 filters):

```
Input Image (H x W x 3) ──► Conv Layer with 64 Filters (3 x 3 x 3) ──► Output Feature Map (H' x W' x 64)
```

Each of the 64 channels in the output feature map specializes in detecting a different visual primitive!

---

## 4. Hierarchical Feature Learning: How CNNs "See"

Unlike classical CV (where human engineers manually coded Sobel or Haar filters), a CNN **learns its filter weights automatically through backpropagation**:

```
Layer 1 (Shallow)            Layer 2 (Middle)             Layer 3 (Deep)
┌───────────────────────┐    ┌───────────────────────┐    ┌───────────────────────┐
│ Low-Level Features    │    │ Mid-Level Features    │    │ High-Level Features   │
│ - Diagonal Edges      │───►│ - Textures & Corners  │───►│ - Eyes, Noses, Faces  │
│ - Color Blends        │    │ - Circles & Grids     │    │ - Wheels, Car Doors   │
│ - Horizontal Lines    │    │ - Repetitive Patterns │    │ - Complete Objects    │
└───────────────────────┘    └───────────────────────┘    └───────────────────────┘
```

### The Intuition:
- The first layer sees tiny local pixel changes (edges).
- The second layer combines edges to form geometric shapes (circles, corners, textures).
- The deeper layers combine shapes into semantically rich objects (faces, dogs, vehicles).

---

## 5. Fresher Interview Question

**Q: In PyTorch, what are the parameters of `nn.Conv2d(in_channels=3, out_channels=32, kernel_size=3)`? How many trainable weights are there?**
> **Answer:**
> - Each filter must span across all `in_channels` ($3$).
> - So one filter has size: $3 \times 3 \times 3 = 27$ weights.
> - With `out_channels=32`, we have 32 such filters: $32 \times 27 = 864$ weights.
> - Plus 1 bias per filter ($32$ biases).
> - **Total trainable parameters:** $864 + 32 = \mathbf{896\text{ parameters}}$.
