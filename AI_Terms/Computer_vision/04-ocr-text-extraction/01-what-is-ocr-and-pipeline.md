# What is OCR & The Complete Extraction Pipeline

## 1. What is OCR?

**OCR (Optical Character Recognition)** is the technology that converts images of text (scanned documents, photo receipts, license plates, road signs) into machine-encoded, editable digital text strings.

Without OCR, a scanned invoice is just a flat 2D grid of colored pixels. With OCR, your software can extract the invoice number, date, vendor, and total amount.

---

## 2. The Two Internal Stages of Modern OCR

A common fresher misconception is that OCR is a single neural network. In reality, modern high-accuracy OCR consists of **two distinct machine learning stages**:

```
                       ┌───────────────────────────────┐
                       │          Input Image          │
                       └──────────────┬────────────────┘
                                      │
                                      ▼
                       ┌───────────────────────────────┐
                       │ Stage 1: Text Detection       │
                       │ (Where is the text?)          │
                       │ Models: CRAFT, DBNet, EAST    │
                       └──────────────┬────────────────┘
                                      │ Crops text bounding boxes
                                      ▼
                       ┌───────────────────────────────┐
                       │ Stage 2: Text Recognition     │
                       │ (What characters are these?)  │
                       │ Models: CRNN, CTC, TrOCR      │
                       └──────────────┬────────────────┘
                                      │ Decoded string + Confidence
                                      ▼
                       ┌───────────────────────────────┐
                       │ Output Text: "Total: $450.00" │
                       └───────────────────────────────┘
```

1. **Text Detection (Localization):**
   - Finds the bounding polygons surrounding words or lines.
   - Algorithms: **CRAFT** (Character Region Awareness for Text Detection), **DBNet** (Real-time scene text detector).
2. **Text Recognition (Transcription):**
   - Takes each cropped text strip and decodes the pixel sequence into ASCII/Unicode text.
   - Architectures: **CRNN** (CNN to extract feature maps + RNN to model character sequences + CTC loss).

---

## 3. The End-to-End OCR Engineering Pipeline

In real-world applications, feeding a raw photo straight into an OCR engine yields poor accuracy ($\le 60\%$). A production pipeline looks like this:

```
[Raw Camera Photo / Scan]
         │
         ▼
[1. Document Geometry] ──► Perspective warp to flatten folded page
         │
         ▼
[2. Image Preprocessing] ──► Grayscale, Denoise, Adaptive Binarize, Deskew
         │
         ▼
[3. OCR Engine] ──► Detect text regions & recognize characters (Tesseract / EasyOCR)
         │
         ▼
[4. Post-Processing] ──► Regex cleanup, spellcheck, formatting
         │
         ▼
[5. Structured Extraction] ──► Feed into LLM (RAG / LangGraph) for JSON parsing
```

---

## 4. Types of OCR Tasks

| Category | Typical Example | Difficulty | Best Tool |
|---|---|---|---|
| **Document OCR** | Scanned PDFs, legal contracts, receipts | Moderate (clean background, standard fonts) | Tesseract, PaddleOCR |
| **Scene Text OCR** | Billboards, store signs, T-shirt text | High (varying perspectives, shadows, wild fonts) | EasyOCR, CRAFT |
| **Handwritten OCR (HTR)** | Doctor prescriptions, exam sheets | Very High (cursive variations) | TrOCR (Vision Transformer) |
