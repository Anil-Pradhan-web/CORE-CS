# Key Terminology (Quick Glossary)

Short, simple definitions designed for fast recall and fresher interviews. Full technical breakdowns are provided in each module.

---

## 1. Image & OpenCV Basics

| Term | Simple Meaning |
|---|---|
| **Pixel** | The smallest addressable unit of an image, representing color/brightness value (0–255). |
| **Channels** | Number of color planes in an image (e.g., Grayscale = 1 channel, RGB/BGR = 3 channels). |
| **Resolution** | Total width and height in pixels (e.g., $1920 \times 1080$). |
| **BGR vs RGB** | OpenCV loads color images in **Blue-Green-Red** order; standard libraries use **Red-Green-Blue**. |
| **Grayscale** | Single-channel image representing light intensity from 0 (pure black) to 255 (pure white). |
| **Thresholding** | Converting an image into binary (black & white) based on pixel intensity cutoffs. |
| **Kernel / Filter** | A small matrix (e.g., $3 \times 3$) passed over an image to apply blur, sharpen, or edge detection. |
| **Contour** | A continuous curve joining all boundary points having the same color or intensity. |
| **ROI (Region of Interest)** | A cropped sub-portion of an image where your target object resides. |

---

## 2. Anomaly Detection & SSIM

| Term | Simple Meaning |
|---|---|
| **Anomaly Detection** | Identifying unusual patterns, defects, or deviations that do not match normal data. |
| **MSE (Mean Squared Error)** | Pixel-by-pixel mathematical difference; fails when lighting shifts or images shift by 1 pixel. |
| **SSIM** | **Structural Similarity Index Measure**: compares luminance, contrast, and structure (-1 to +1). |
| **Golden / Reference Image** | The flawless, defect-free sample image used as the baseline for comparison. |
| **Difference Map** | The visual subtraction matrix highlighting exact areas of structural degradation. |

---

## 3. OCR (Optical Character Recognition)

| Term | Simple Meaning |
|---|---|
| **OCR** | Technology that converts typed, handwritten, or printed text in images into editable digital text. |
| **Text Localization** | Finding *where* text exists inside an image (detecting text bounding boxes). |
| **Text Recognition** | Decoding the characters inside localized text boxes into machine-readable strings. |
| **Deskewing** | Rotating a tilted or crooked scanned document back to horizontal alignment. |
| **Binarization** | Converting text images into high-contrast black text on pure white background. |
| **Tesseract** | Popular open-source OCR engine maintained by Google. |
| **EasyOCR** | Modern PyTorch-based OCR library supporting 80+ languages with deep learning models. |

---

## 4. Deep Learning & CNNs

| Term | Simple Meaning |
|---|---|
| **Convolution** | Element-wise multiplication and sum of a sliding kernel across an image to extract features. |
| **Feature Map** | The output matrix produced by passing a convolutional filter over an image. |
| **Stride** | The step size (number of pixels) the filter shifts across the image during convolution. |
| **Padding** | Adding border pixels (usually zeros) to preserve image dimensions after convolution. |
| **Pooling (Max/Avg)** | Downsampling operation to reduce spatial dimensions and compute costs while retaining key features. |
| **Receptive Field** | The specific region of the input image that a particular neuron "sees" and responds to. |
| **Skip Connection** | A shortcut in ResNet that lets gradients flow directly, preventing vanishing gradients in deep nets. |
| **Transfer Learning** | Taking a model pretrained on a massive dataset (ImageNet) and adapting it to your custom task. |

---

## 5. Object Detection & YOLO11n

| Term | Simple Meaning |
|---|---|
| **Bounding Box** | Coordinates defining an object's location: $[x_{\text{min}}, y_{\text{min}}, x_{\text{max}}, y_{\text{max}}]$ or $[x_{\text{center}}, y_{\text{center}}, w, h]$. |
| **IoU (Intersection over Union)** | Overlap ratio between predicted box and ground truth box (0 = no overlap, 1 = perfect match). |
| **NMS (Non-Maximum Suppression)** | Algorithm that removes duplicate, overlapping bounding boxes for the same object. |
| **Confidence Score** | The model's probability that an object exists in a box AND belongs to a specific class. |
| **mAP (mean Average Precision)** | The gold standard metric for object detection accuracy (e.g., mAP@50 or mAP@50-95). |
| **One-Stage Detector** | Model that predicts bounding boxes and classes in a single forward pass (e.g., YOLO, SSD). |
| **Two-Stage Detector** | Model that first generates region proposals, then classifies them (e.g., Faster R-CNN). |
| **YOLO11n (Nano)** | The most lightweight version of YOLO11, designed for maximum FPS and edge deployment. |

---

## 6. Real-Time Pipelines & Deployment

| Term | Simple Meaning |
|---|---|
| **FPS (Frames Per Second)** | Number of video frames processed in one second (real-time typically requires $\ge 25\text{–}30\text{ FPS}$). |
| **Inference Latency** | Time taken by the model to process a single frame (e.g., 8 ms). |
| **ONNX** | Open Neural Network Exchange: a cross-platform format to run models fast on any hardware. |
| **TensorRT** | NVIDIA's high-performance inference optimizer for GPUs. |

---

## 7. Multimodal AI & Vision-Language

| Term | Simple Meaning |
|---|---|
| **VLM (Vision-Language Model)** | A model that accepts both images and text prompts (e.g., GPT-4o, LLaVA, Gemini Vision). |
| **CLIP** | OpenAI model trained to connect images and text in a shared vector embedding space. |
| **SAM (Segment Anything)** | Meta's foundational zero-shot segmentation model that segments any object on prompt. |
| **Vision RAG** | Retrieving relevant images or document screenshots and injecting them into a multimodal LLM prompt. |
