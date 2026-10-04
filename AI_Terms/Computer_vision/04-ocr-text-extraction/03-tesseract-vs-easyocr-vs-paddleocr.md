# Tesseract vs EasyOCR vs PaddleOCR — Comparison & Code

## 1. Engine Comparison: Which One to Choose?

When building an OCR pipeline, picking the right engine depends on your hardware and image source:

| Feature | Tesseract (Google) | EasyOCR (JaidedAI) | PaddleOCR (Baidu) |
|---|---|---|---|
| **Underlying Tech** | C++ Engine + LSTM RNN | PyTorch + CRAFT + ResNet | PaddlePaddle + PP-OCRv4 |
| **Best Used For** | Clean scanned PDFs, books | Wild scene text, street signs, rotated labels | Complex tables, multilingual receipts |
| **Speed (CPU)** | Very Fast ($<50\text{ ms}$) | Moderate ($200\text{--}500\text{ ms}$) | Blazing Fast ($30\text{--}80\text{ ms}$) |
| **GPU Acceleration** | Poor / Complex | Native PyTorch (`gpu=True`) | Native support |
| **Ease of Setup** | Requires separate OS binary (`apt`/`.exe`) | `pip install easyocr` (Self-contained) | `pip install paddleocr` |
| **Bounding Box Format** | Raw character/word boxes | 4 corner polygons `[[x1,y1], ...]` | 4 corner polygons |

---

## 2. Practical Implementation: EasyOCR

EasyOCR is the most popular choice for modern Python developers because it installs easily via `pip` and includes deep-learning text detection (CRAFT) out of the box.

```bash
pip install easyocr
```

```python
import easyocr
import cv2

# Initialize reader with desired languages (e.g., English 'en')
# Set gpu=True if CUDA is available, else False
reader = easyocr.Reader(['en'], gpu=False)

# Read text directly from image file or NumPy array
results = reader.readtext('receipt.jpg')

# Output structure for each detection:
# [ (bounding_box_coordinates, recognized_text, confidence_score) ]
for bbox, text, confidence in results:
    # Filter out low-confidence noise
    if confidence > 0.50:
        print(f"Detected: '{text}' | Confidence: {confidence * 100:.1f}%")
        
        # Bounding box points: top-left, top-right, bottom-right, bottom-left
        pt1 = tuple(map(int, bbox[0]))
        pt2 = tuple(map(int, bbox[2]))
        
        # Draw green box and text label
        cv2.rectangle(img, pt1, pt2, (0, 255, 0), 2)
        cv2.putText(img, text, (pt1[0], pt1[1] - 5), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 1)
```

---

## 3. Practical Implementation: PyTesseract

```bash
pip install pytesseract
# (Also requires installing Tesseract binary on Windows/Linux)
```

```python
import pytesseract
from PIL import Image

# In Windows, specify the tesseract executable path if not in system PATH:
# pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files\Tesseract-OCR\tesseract.exe'

# 1. Simple text extraction
text = pytesseract.image_to_string(Image.open('document.png'))
print("Extracted Text:\n", text)

# 2. Detailed extraction with bounding boxes and confidence scores
data = pytesseract.image_to_data(Image.open('document.png'), output_type=pytesseract.Output.DICT)

for i in range(len(data['text'])):
    conf = int(data['conf'][i])
    word = data['text'][i].strip()
    
    # Filter valid words with confidence > 60%
    if conf > 60 and len(word) > 0:
        x, y, w, h = data['left'][i], data['top'][i], data['width'][i], data['height'][i]
        print(f"Word: '{word}' at ({x}, {y}, {w}, {h}) with {conf}% confidence")
```

---

## 4. Fresher Interview Questions

**Q1: How do you handle text in multiple languages?**
> **Answer:** In EasyOCR, we pass language codes during initialization: `easyocr.Reader(['en', 'hi'])` for English and Hindi. In Tesseract, we pass the language flag: `lang='eng+hin'` and ensure the corresponding `.traineddata` files are present.

**Q2: What is the primary cause of hallucinated characters in OCR?**
> **Answer:** Low contrast and background noise (such as watermark lines or coffee stains). The recognition network mistakes noise patterns for character strokes (e.g., misreading a dot as a period or a comma, or mistaking `'O'` for `'0'`). Applying adaptive thresholding and morphological opening before OCR resolves most of these errors.
