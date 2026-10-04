# Lightweight Models & Depthwise Separable Convolutions (MobileNet)

## 1. Why Lightweight Models Matter

Most computer vision applications do not run in air-conditioned cloud data centers. They run on:
- Security CCTV cameras and Raspberry Pis
- Drones and autonomous robots
- Smartphones and low-power edge microcontrollers

Running a 138-million-parameter model like VGG on a battery-powered device drains the battery in minutes and drops framerates to $<2\text{ FPS}$.

To make models run in **real-time on edge devices**, researchers created lightweight architectures like **MobileNet** and **YOLO11n (Nano)**.

---

## 2. Standard Convolution vs Depthwise Separable Convolution

A standard convolutional layer performs two jobs simultaneously:
1. **Spatial Filtering:** Looking for patterns in $X$ and $Y$ space.
2. **Channel Mixing:** Combining information across input channels ($R, G, B$).

**MobileNet (Howard et al., 2017)** splits these two jobs into two independent, ultra-cheap steps:

```
Standard Convolution:
[Input (H x W x Cin)] ────(Single Heavy Conv: 3x3xCin)────► [Output (H' x W' x Cout)]

Depthwise Separable Convolution:
[Input (H x W x Cin)]
         │
         ▼  Step 1: Depthwise Conv (Spatial Only - 3x3 filter per channel, no channel mixing)
[Intermediate (H' x W' x Cin)]
         │
         ▼  Step 2: Pointwise Conv (Channel Mixing Only - 1x1 filter combining channels)
[Output (H' x W' x Cout)]
```

---

## 3. The Math: Why It Is 8x to 9x Faster

Let:
- Kernel size: $D_K \times D_K$ (e.g., $3 \times 3$)
- Input channels: $M$
- Output channels: $N$
- Output feature map size: $D_F \times D_F$

### Computational Cost of Standard Conv:
$$\text{Cost}_{\text{standard}} = D_K \cdot D_K \cdot M \cdot N \cdot D_F \cdot D_F$$

### Computational Cost of Depthwise Separable Conv:
$$\text{Cost}_{\text{separable}} = \underbrace{(D_K \cdot D_K \cdot M \cdot D_F \cdot D_F)}_{\text{Depthwise (Spatial)}} + \underbrace{(M \cdot N \cdot D_F \cdot D_F)}_{\text{Pointwise (1x1 Channel)}}$$

### The Reduction Ratio:
$$\frac{\text{Cost}_{\text{separable}}}{\text{Cost}_{\text{standard}}} = \frac{D_K^2 \cdot M + M \cdot N}{D_K^2 \cdot M \cdot N} = \frac{1}{N} + \frac{1}{D_K^2}$$

For a standard $3 \times 3$ kernel ($D_K = 3$) and large $N$ (e.g., 256 channels):
$$\text{Ratio} \approx \frac{1}{3^2} = \frac{1}{9} \approx \mathbf{11\% \text{ of original computations!}}$$

**That is an $8\times$ to $9\times$ reduction in arithmetic operations (FLOPs)** with only a $1\%$ drop in top-1 accuracy!

---

## 4. Connection to Your Resume: YOLO11n

> [!TIP]
> The same philosophy powers the **"n" (Nano)** variant of **YOLO11** on your resume!
> YOLO11n uses lightweight cross-stage partial blocks (C3k2) and depth-scaled bottleneck convolutions to deliver **$>100\text{ FPS}$ on standard laptops and edge hardware** while keeping model weights under $6\text{ MB}$.
