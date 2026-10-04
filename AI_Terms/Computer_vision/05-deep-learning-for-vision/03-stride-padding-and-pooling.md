# Stride, Padding, and Pooling — Controlling Dimensions

## 1. The Output Dimension Formula

Every fresher must know this formula by heart for technical interviews:

Given:
- Input dimension: $W$ (Width or Height)
- Kernel size: $K$
- Padding: $P$
- Stride: $S$

The output spatial dimension $O$ is:
$$\mathbf{O = \left\lfloor \frac{W - K + 2P}{S} \right\rfloor + 1}$$

---

## 2. Padding: Preserving Borders & Dimensions

When a $3 \times 3$ kernel slides across an image without padding:
- The edge pixels are only visited once or twice.
- The output feature map shrinks by $2$ pixels in each dimension ($32 \times 32 \rightarrow 30 \times 30$).
- After 10 layers, the image would shrink to zero!

```
No Padding (Valid)                    Same Padding (Zero-Padding)
┌───┬───┬───┐                         ░░░░░░░░░░░░░░░
│   │   │   │                         ░ ┌───┬───┬───┐ ░
├───┼───┼───┤                         ░ │   │   │   │ ░  <-- Zero padding (P=1)
│   │   │   │                         ░ ├───┼───┼───┤ ░      preserves the
├───┼───┼───┤                         ░ │   │   │   │ ░      original spatial size
│   │   │   │                         ░ └───┴───┴───┘ ░
└───┴───┴───┘                         ░░░░░░░░░░░░░░░
```

### Types of Padding:
1. **Valid Padding ($P=0$):** No padding added. The output is smaller than the input.
2. **Same Padding:** Padding is chosen such that output spatial dimensions equal input dimensions when $S=1$.
   For kernel size $K$:
   $$P = \frac{K - 1}{2} \quad (\text{For } K=3, P=1; \text{ for } K=5, P=2)$$

---

## 3. Stride: The Step Size

**Stride ($S$)** is the number of pixels the filter shifts horizontally and vertically between convolutions.
- **$S=1$:** Filter shifts 1 pixel at a time (dense sampling).
- **$S=2$:** Filter skips every second pixel. This halves the spatial dimensions ($H/2, W/2$), performing both feature extraction and downsampling at the same time.

---

## 4. Pooling: Downsampling & Spatial Invariance

A **Pooling layer** has no learnable weights! It applies a fixed mathematical function over non-overlapping windows (typically $2 \times 2$ with stride $2$).

### 1. Max Pooling
Selects the maximum value in each $2 \times 2$ patch:
```
Input 4x4 Patch                  Max Pooling (2x2, Stride 2)
┌────┬────┬────┬────┐
│ 12 │ 20 │ 30 │  0 │            ┌────┬────┐
├────┼────┼────┼────┤            │ 20 │ 30 │
│  8 │ 14 │  2 │ 15 │     ──►    ├────┼────┤
├────┼────┼────┼────┤            │ 95 │ 40 │
│ 95 │ 45 │ 10 │ 40 │            └────┴────┘
├────┼────┼────┼────┤
│ 32 │ 18 │  5 │ 22 │
└────┴────┴────┴────┘
```

#### Why Max Pooling is so powerful:
1. **Reduces Computation:** Shrinks spatial dimensions by 75% ($H/2 \times W/2$), drastically reducing GPU memory for subsequent layers.
2. **Translation Invariance:** If a feature shifts slightly by 1 pixel, the maximum value in the neighborhood remains the same!

### 2. Global Average Pooling (GAP)
Instead of flattening large feature maps into massive dense layers (which causes overfitting), modern architectures (ResNet, YOLO) take the **average of each entire feature map**:
```
Feature Map: 7 x 7 x 512 channels ──► Global Average Pooling ──► 1 x 1 x 512 vector
```
*This eliminates millions of dense weights and prevents overfitting.*

---

## 5. Quick Test: Can you calculate this?

**Interview Problem:**
Input is $224 \times 224 \times 3$.
Apply `Conv2d` with kernel $K=7$, padding $P=3$, stride $S=2$.
What is the output spatial size?

$$O = \left\lfloor \frac{224 - 7 + 2(3)}{2} \right\rfloor + 1 = \left\lfloor \frac{224 - 7 + 6}{2} \right\rfloor + 1 = \left\lfloor \frac{223}{2} \right\rfloor + 1 = 111 + 1 = \mathbf{112 \times 112}$$
*(Notice: $S=2$ cleanly halved the resolution from $224$ to $112$!)*
