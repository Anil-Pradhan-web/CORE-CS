# Top 40 Computer Vision Interview Questions & Answers

A curated question bank covering the most common technical questions asked in entry-level Computer Vision and AI Engineer interviews.

---

## Section 1: Image & OpenCV Fundamentals

### Q1: Why does OpenCV read images in BGR instead of RGB?
> **A:** For historical reasons. In the late 1990s when OpenCV was created, camera manufacturers and Windows DIB formats stored pixel data in memory in BGR order.

### Q2: What is the difference between global thresholding and adaptive thresholding?
> **A:** Global thresholding applies a single fixed cutoff value across the entire image. Adaptive thresholding calculates a different threshold for every small pixel neighborhood, making it robust against non-uniform lighting and shadows.

### Q3: How does Otsu's thresholding work?
> **A:** It automatically computes an optimal global threshold by analyzing the image histogram and finding the value that maximizes the inter-class variance (spread) between foreground and background pixels.

### Q4: Explain the 4 steps of the Canny Edge Detector.
> **A:**
> 1. **Gaussian Blur:** Smooths out image noise.
> 2. **Gradient Calculation:** Uses Sobel kernels to find gradient magnitude and direction.
> 3. **Non-Maximum Suppression:** Thins edges down to 1-pixel width.
> 4. **Hysteresis Thresholding:** Uses two thresholds to preserve strong edges and connect weak edges while rejecting noise.

### Q5: What is the difference between morphological Opening and Closing?
> **A:**
> - **Opening = Erosion followed by Dilation:** Removes tiny white noise specks without altering overall object size.
> - **Closing = Dilation followed by Erosion:** Bridges small holes and gaps inside foreground shapes.

### Q6: What is a contour in OpenCV?
> **A:** A continuous curve joining all boundary points along an object that share the same color or intensity.

### Q7: Why convert images to grayscale before edge detection?
> **A:** Grayscale reduces 3 channels to 1, cutting compute and memory by 66%. Edges depend on light intensity contrast (luminance), not color hue.

### Q8: What is the difference between `cv2.INTER_LINEAR` and `cv2.INTER_AREA`?
> **A:** `INTER_LINEAR` (bilinear) is best for enlarging/upsampling images. `INTER_AREA` is best for downsampling/shrinking images because it avoids aliasing artifacts.

---

## Section 2: Deep Learning & CNNs

### Q9: Why do MLPs fail on images?
> **A:**
> 1. Parameter explosion (millions of dense weights).
> 2. Flattening destroys 2D spatial pixel relationships.
> 3. Lack of translation invariance.

### Q10: What is weight sharing in CNNs?
> **A:** The same small filter matrix (e.g., $3 \times 3$) slides across the entire image, applying identical weights everywhere. This keeps parameter count tiny and independent of image size.

### Q11: What is the formula for convolutional layer output dimensions?
> **A:** $O = \left\lfloor \frac{W - K + 2P}{S} \right\rfloor + 1$.

### Q12: Why is stacking two $3 \times 3$ filters better than one $5 \times 5$ filter?
> **A:** They cover the same $5 \times 5$ receptive field, but use $28\%$ fewer parameters ($18$ vs $25$) and include two non-linear activations (ReLUs) instead of one.

### Q13: What is the purpose of Max Pooling?
> **A:** It downsamples feature maps by $50\%$ in height and width, reducing compute and memory while providing translation invariance.

### Q14: What is Global Average Pooling (GAP) and why is it used?
> **A:** It takes the average of each entire feature map channel, converting a $(7, 7, 512)$ tensor into a $(1, 512)$ vector. It replaces heavy dense layers, preventing overfitting.

### Q15: What problem does ResNet solve and how?
> **A:** It solves the **vanishing gradient and degradation problem** in very deep networks using **skip/residual connections** ($F(x) + x$). The direct addition guarantees that gradients flow back to early layers without diminishing.

### Q16: How do Depthwise Separable Convolutions work in MobileNet?
> **A:** They split standard convolution into two cheap steps: Depthwise (spatial filtering per channel) and Pointwise ($1 \times 1$ conv for channel mixing). This reduces computations by $8\times\text{ to }9\times$.

### Q17: What is the difference between Feature Extraction and Fine-Tuning?
> **A:** In feature extraction, you freeze the backbone and train only the new classification head. In fine-tuning, you unfreeze top backbone layers and train them with a very small learning rate.

### Q18: What is Batch Normalization and why is it important?
> **A:** It normalizes layer activations across the mini-batch to zero mean and unit variance. It accelerates training, allows higher learning rates, and acts as a mild regularizer.

---

## Section 3: Object Detection & YOLO11n

### Q19: What is the difference between object localization and object detection?
> **A:** Localization finds and classifies a **single** object with one bounding box. Detection finds and classifies **multiple** objects with varying bounding boxes in the same image.

### Q20: What is IoU (Intersection over Union)?
> **A:** $\text{IoU} = \frac{\text{Area of Overlap}}{\text{Area of Union}}$. It measures how well a predicted bounding box matches ground truth.

### Q21: Explain Non-Maximum Suppression (NMS).
> **A:** It eliminates duplicate overlapping bounding boxes for the same object by iteratively selecting the highest-confidence box and suppressing any neighbor whose $\text{IoU} > \text{threshold}$.

### Q22: What does "mAP" stand for and how is it calculated?
> **A:** Mean Average Precision. It computes Average Precision (area under the Precision-Recall curve) for each class and averages them across all classes.

### Q23: What is the difference between `mAP@50` and `mAP@50-95`?
> **A:** `mAP@50` evaluates detections at a fixed IoU threshold of 0.50. `mAP@50-95` averages mAP across 10 IoU thresholds from 0.50 to 0.95 with step 0.05 (the strictest COCO standard).

### Q24: What is the primary difference between Faster R-CNN and YOLO?
> **A:** Faster R-CNN is a two-stage detector (RPN proposes candidate regions, second stage classifies). YOLO is a one-stage detector that predicts boxes and classes in a single forward pass, making it $5\times\text{ to }10\times$ faster.

### Q25: What does the "n" mean in YOLO11n?
> **A:** "Nano" — the most lightweight, edge-optimized model in the YOLO11 family ($\approx 2.6\text{M parameters}$, $\approx 5.6\text{ MB}$ weights).

### Q26: What is an Anchor-Free detector?
> **A:** Instead of predicting offsets from predefined template anchor boxes, modern YOLO models directly predict the distance from the cell center to the 4 bounding box edges.

### Q27: What is a Decoupled Head in modern YOLO?
> **A:** Splitting the final prediction into two independent branches: one dedicated to bounding box regression and one dedicated to class classification.

### Q28: What is the YOLO dataset annotation format?
> **A:** `<class_id> <x_center> <y_center> <width> <height>`, with all coordinate values normalized between $0.0$ and $1.0$.

---

## Section 4: Anomaly Detection & SSIM

### Q29: What is SSIM and what is its range?
> **A:** Structural Similarity Index Measure. It compares images based on luminance, contrast, and structure. Its score ranges from $-1.0$ to $+1.0$ (where $+1.0$ means identical).

### Q30: Why is SSIM better than MSE for defect detection?
> **A:** MSE measures raw pixel differences and fails when images shift by 1 pixel or when ambient lighting changes. SSIM evaluates perceived structural integrity.

### Q31: How do you localize a defect using SSIM?
> **A:** Generate the SSIM difference map, invert and threshold it with Otsu's method, apply morphological opening/closing, and extract contours to draw bounding boxes around defect blobs.

### Q32: What is a "Golden Template"?
> **A:** A known, calibrated, defect-free reference image against which manufactured test items are compared.

### Q33: How do you handle vibration/misalignment in SSIM inspection?
> **A:** Perform image registration (alignment) first using feature matching (ORB/SIFT) and perspective warping (`cv2.warpPerspective`).

### Q34: What threshold indicates a defect in SSIM?
> **A:** Typically, an SSIM score below $0.95$ or any localized defect contour area exceeding $40\text{--}50\text{ pixels}$.

---

## Section 5: OCR & Production Deployment

### Q35: What are the two internal stages of modern OCR?
> **A:** Text Detection (finding bounding boxes of words/lines using models like CRAFT or DBNet) and Text Recognition (decoding pixel sequences into character strings using CRNN or TrOCR).

### Q36: How do you fix a crooked/tilted scanned document before OCR?
> **A:** Deskewing: detect text pixel coordinates, find the angle using `cv2.minAreaRect()`, and rotate the image with `cv2.warpAffine()`.

### Q37: Tesseract vs EasyOCR: when to choose which?
> **A:** Choose Tesseract for clean scanned PDFs and high-throughput documents. Choose EasyOCR for scene text, photos taken at angles, and complex multi-lingual signs.

### Q38: Why export models to ONNX format?
> **A:** ONNX removes heavy PyTorch dependencies, optimizes computation graph nodes, and runs $2\times\text{ to }3\times$ faster on CPUs using ONNX Runtime.

### Q39: What causes video buffer lag in OpenCV and how do you fix it?
> **A:** OpenCV's `cv2.VideoCapture` blocks and queues frames if inference is slower than camera frame rate. The fix is **Threaded Video Capture**, which reads frames asynchronously on a background thread.

### Q40: How does Vision connect to GenAI and LangGraph?
> **A:** Vision models (YOLO11n, OCR, SSIM) act as the perception sensors extracting structured JSON events from images, while LangGraph agents act as the reasoning engine to query manuals via RAG and trigger automated actions.
