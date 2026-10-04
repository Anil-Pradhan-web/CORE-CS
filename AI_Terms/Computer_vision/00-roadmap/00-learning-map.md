# Computer Vision Learning Map — The Complete Story (Fresher Edition)

This folder is **ONE continuous story**, designed to take you from raw pixels all the way to modern Deep Learning, Real-time Object Detection (**YOLO11n**), **OCR**, **Anomaly Detection (SSIM)**, and **Multimodal AI**.

Read the folders in number order. Every file explains why the previous topic matters and what comes next.

---

## 1. The Big Picture Chain

```
[Digital Image] -> Raw pixels (H x W x C NumPy array)
       │
       ▼
[Classical OpenCV] -> Filters, Thresholding, Contours, Edge Detection
       │
       ├─────────────────────────────────┐
       ▼                                 ▼
[SSIM Anomaly Detection]       [OCR Text Extraction]
(Defect finding via structure)   (Document to machine text)
       │                                 │
       └────────────────┬────────────────┘
                        ▼
           [Why ANN / MLP Fails on Images]
                        │
                        ▼
           [Convolutional Neural Networks (CNN)]
           (Kernels, Stride, Padding, Pooling)
                        │
                        ▼
           [CNN Architectures & Transfer Learning]
           (ResNet, MobileNet, Feature Extraction)
                        │
                        ▼
           [Object Detection Fundamentals]
           (Bounding Boxes, IoU, NMS, mAP)
                        │
                        ▼
           [YOLO11n Deep-Dive]
           (Nano Architecture, Custom Training, Edge Deployment)
                        │
                        ▼
           [Real-Time Video Pipelines]
           (Webcam streams, FPS optimization, Threading)
                        │
                        ▼
           [Multimodal Vision & GenAI Bridge]
           (VLM, Document AI, Vision RAG, Multi-Agent CV)
                        │
                        ▼
           [Fresher Interview Prep & Resume Defense]
```

---

## 2. The Story in One Paragraph

To a computer, an image is not a picture; it is a 2D grid of numbers (pixels). In **Classical OpenCV**, we manipulate these numbers using math filters to find edges, shapes, and contours. Using these foundations, we can compare images to detect defects (**SSIM Anomaly Detection**) and read text from scanned documents (**OCR**). But classical algorithms fail when objects have complex variations, angles, or lighting. Traditional Multi-Layer Perceptrons (MLPs) explode in parameter size and destroy 2D spatial arrangement, which is why **Convolutional Neural Networks (CNNs)** were invented. CNNs learn spatial patterns automatically. Rather than training huge models from scratch, we use **Transfer Learning** with architectures like **ResNet** or lightweight **MobileNet**. When we need to pinpoint *where* multiple objects are in real-time, we use **Object Detection**, specifically modern single-stage detectors like **YOLO11n (Nano)**. Finally, we optimize these models for real-time video streams, connect vision with **Generative AI** (Vision RAG and Multi-Agent workflows), and prepare to defend our skills in technical interviews.

---

## 3. Resume Skills Alignment Map

This roadmap directly covers the skills on your resume:

| Resume Skill | Targeted Folder | What Interviewers Look For |
|---|---|---|
| **OpenCV** | `01-image-fundamentals`<br>`02-opencv-core`<br>`09-real-time-video-pipelines` | Pixel manipulation, color space conversion (BGR vs RGB), thresholding, contour detection, video streaming loops. |
| **Anomaly Detection (SSIM)** | `03-ssim-and-anomaly-detection` | Difference between pixel-level loss (MSE) and structural similarity (SSIM), reference-based inspection, difference masking. |
| **OCR** | `04-ocr-text-extraction` | Image preprocessing for OCR (deskew, binarization, denoising), OCR engines (Tesseract/EasyOCR), text bounding boxes. |
| **YOLO11n** | `07-object-detection-fundamentals`<br>`08-yolo11n-deep-dive` | Why YOLO is single-stage, what "n" (nano) means, IoU, NMS, mAP@0.5 calculation, custom dataset training & ONNX export. |
| **AI / GenAI Integration** | `10-multimodal-cv-and-genai` | Bridging CV with LangGraph, RAG, and LLMs (Multimodal Document AI, Vision Agents). |
| **Interview Readiness** | `99-fresher-interview-prep` | Explaining projects in STAR format, answering counter-questions, code cheatsheets. |

---

## 4. Complete Folder Map

| Folder | What you learn | Key question it answers |
|---|---|---|
| **`00-roadmap`** | Complete learning map & resume alignment | "What is the logical order and why does it matter?" |
| **`01-image-fundamentals`** | Pixels, RGB vs BGR, HSV, NumPy tensors | "What actually IS an image to a computer?" |
| **`02-opencv-core`** | Transformations, Blurring, Thresholding, Canny, Contours | "How do I manipulate and inspect images using OpenCV?" |
| **`03-ssim-and-anomaly-detection`** | Structural Similarity, MSE vs SSIM, Defect detection | "How do I find visual defects or anomalies automatically?" |
| **`04-ocr-text-extraction`** | Tesseract, EasyOCR, document preprocessing | "How do I extract readable text from real-world images?" |
| **`05-deep-learning-for-vision`** | Why ANN fails, CNN kernels, stride, padding, pooling | "Why do we need CNNs instead of standard Neural Networks?" |
| **`06-cnn-architectures`** | VGG, ResNet skip connections, MobileNet, Transfer learning | "How do standard vision architectures work and how do I reuse them?" |
| **`07-object-detection-fundamentals`** | Bounding boxes, IoU, NMS, Precision/Recall, mAP | "How do models locate and classify multiple objects at once?" |
| **`08-yolo11n-deep-dive`** | YOLO evolution, YOLO11n Nano architecture, training, ONNX | "How does YOLO11n work and how do I train it for edge devices?" |
| **`09-real-time-video-pipelines`** | Webcam loops, FPS measurement, latency bottlenecks | "How do I run vision models on live video feeds smoothly?" |
| **`10-multimodal-cv-and-genai`** | VLMs, Document AI, Vision RAG, Multi-Agent integration | "How does Computer Vision connect with modern GenAI?" |
| **`99-fresher-interview-prep`** | Resume defense guide, STAR project templates, Top 40 Q&A | "How do I answer tricky interview questions and prove my skills?" |
