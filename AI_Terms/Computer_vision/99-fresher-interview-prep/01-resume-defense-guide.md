# Resume Defense Guide — Answering Interview Questions with Authority

## 1. Defending Every CV Skill on Your Resume

When an interviewer points to your technical skills:
> **Languages:** Python, Java, JavaScript, SQL  
> **AI & Computer Vision:** LangGraph, Multi-Agent Systems, RAG, Prompt Engineering, LLM Integration, OpenCV, YOLO11n, OCR, Anomaly Detection (SSIM)

Here are the exact talking points and explanations to give:

---

## 2. Skill 1: OpenCV

### What the Interviewer Wants to Know:
*"Do you actually understand image processing, or do you just import black-box libraries?"*

### Your Talking Points:
- Explain that images are multi-dimensional NumPy arrays and explain the **BGR vs RGB** memory ordering.
- Highlight your preprocessing workflow: Grayscale conversion, Gaussian/Bilateral denoising, Otsu and Adaptive Gaussian thresholding, Canny edge detection, and morphological operations (Opening/Closing).
- Mention coordinate conventions: $(X, Y)$ math coordinates vs `image[row, col]` NumPy indexing.

---

## 3. Skill 2: YOLO11n

### What the Interviewer Wants to Know:
*"Why YOLO11n? What is the 'n'? How does single-stage detection work?"*

### Your Talking Points:
- **What 'n' means:** YOLO11 Nano — the smallest, ultra-lightweight variant ($\approx 2.6\text{M parameters}$, $\approx 5.6\text{ MB}$ weights).
- **Why you chose it:** Delivers real-time inference ($>50\text{ FPS}$) on standard CPU hardware without needing an expensive dedicated GPU.
- **Architectural features:** Anchor-free decoupled head, C3k2 building blocks, and multi-scale feature pyramids (strides 8, 16, 32).
- **Evaluation:** Tracked validation loss (box CIoU loss, classification BCE loss, Distribution Focal Loss) and evaluated with `mAP@50` and `mAP@50-95`.
- **Production Export:** Exported `.pt` weights to **ONNX format** to eliminate PyTorch runtime bloat and double inference speed.

---

## 4. Skill 3: Anomaly Detection (SSIM)

### What the Interviewer Wants to Know:
*"Why SSIM? Why not Mean Squared Error (MSE) or training an Autoencoder?"*

### Your Talking Points:
- **Limitation of Supervised DL:** Manufacturing defects are rare (severe class imbalance); you don't have thousands of labeled defective samples.
- **Flaws of MSE:** Fails on 1-pixel shifts and ambient illumination changes.
- **SSIM Advantages:** Models the Human Visual System by decomposing images into **Luminance, Contrast, and Structure**.
- **Pipeline:** Golden template comparison $\rightarrow$ SSIM difference map $\rightarrow$ inverted thresholding $\rightarrow$ morphological filtering $\rightarrow$ contour extraction $\rightarrow$ bounding boxes around defects.

---

## 5. Skill 4: OCR (Optical Character Recognition)

### What the Interviewer Wants to Know:
*"What engines did you use and how did you handle dirty, skewed, real-world images?"*

### Your Talking Points:
- Explain that OCR accuracy is **70% preprocessing**:
  - Rescaling to $\approx 300\text{ DPI}$.
  - Bilateral filtering to smooth paper grain while keeping character edges crisp.
  - Adaptive thresholding to remove harsh room shadows.
  - Deskewing using `minAreaRect` to rotate tilted text horizontally.
- Engine knowledge: Used **EasyOCR** (CRAFT text detector + CRNN recognizer) for complex scene text and **Tesseract** for clean documents.
- Post-processing: Extracted bounding boxes and confidence scores, filtering out tokens with confidence $<0.50$, and parsing text with LLMs.

---

## 6. Skill 5: The Full-Stack Bridge (CV + GenAI)

### Your Unique Differentiator:
*"Most candidates are either purely CV engineers or purely LLM prompt engineers. My strength is building end-to-end multimodal systems: using OpenCV and YOLO11n at the edge as the eyes, and connecting them to LangGraph multi-agent systems and RAG pipelines as the reasoning brain."*
