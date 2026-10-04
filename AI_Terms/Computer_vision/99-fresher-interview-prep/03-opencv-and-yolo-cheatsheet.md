# OpenCV & YOLO Python Cheatsheet (Quick Reference)

Keep this cheatsheet handy for technical coding tests, take-home assignments, and live pair-programming rounds.

---

## 1. Image I/O & Color Space Conversion

```python
import cv2

# Read image (loads as BGR)
img = cv2.imread("image.jpg")

# Convert BGR to RGB (for Matplotlib or PyTorch)
img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

# Convert BGR to Grayscale
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Convert BGR to HSV
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

# Save image
cv2.imwrite("output.png", img)
```

---

## 2. Drawing Overlays (Bounding Boxes & Text)

```python
# Draw Rectangle: (x1, y1) to (x2, y2), Color (BGR), Thickness
cv2.rectangle(img, (50, 50), (200, 250), (0, 255, 0), 2)

# Draw Filled Circle: center (x, y), radius, Color, filled (-1)
cv2.circle(img, (125, 150), 8, (0, 0, 255), -1)

# Put Text: (x, y) is bottom-left corner of text string
cv2.putText(img, "Defect: 95%", (50, 40), cv2.FONT_HERSHEY_SIMPLEX, 0.6, (0, 255, 0), 2)
```

---

## 3. Resizing & Cropping (ROI)

```python
# Crop ROI: img[y_min : y_max, x_min : x_max]
roi = img[100:300, 150:400]

# Resize to exact dimensions (Width, Height)
resized = cv2.resize(img, (640, 640), interpolation=cv2.INTER_LINEAR)

# Downsample (Shrink) using INTER_AREA
shrunk = cv2.resize(img, (0, 0), fx=0.5, fy=0.5, interpolation=cv2.INTER_AREA)
```

---

## 4. Denoising, Thresholding & Edges

```python
# Gaussian Blur
blurred = cv2.GaussianBlur(gray, (5, 5), 0)

# Otsu's Automatic Thresholding
_, otsu_bin = cv2.threshold(blurred, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)

# Adaptive Gaussian Thresholding (for shadows in OCR)
adaptive = cv2.adaptiveThreshold(blurred, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C, cv2.THRESH_BINARY, 11, 2)

# Canny Edge Detection (lower, upper)
edges = cv2.Canny(blurred, 50, 150)
```

---

## 5. Morphology & Contours

```python
# Structuring kernel
kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (5, 5))

# Opening (removes noise) and Closing (fills holes)
clean = cv2.morphologyEx(otsu_bin, cv2.MORPH_OPEN, kernel)
solid = cv2.morphologyEx(clean, cv2.MORPH_CLOSE, kernel)

# Find Contours
contours, _ = cv2.findContours(solid, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

for cnt in contours:
    if cv2.contourArea(cnt) > 50:  # Area filter
        x, y, w, h = cv2.boundingRect(cnt)
        cv2.rectangle(img, (x, y), (x + w, y + h), (0, 255, 0), 2)
```

---

## 6. SSIM Anomaly Detection (Resume Skill)

```python
from skimage.metrics import structural_similarity as ssim

# Compute SSIM score and difference map
score, diff = ssim(golden_gray, test_gray, full=True)
diff = (diff * 255).astype("uint8")

# Invert and threshold to extract defect blobs
_, defect_mask = cv2.threshold(diff, 0, 255, cv2.THRESH_BINARY_INV | cv2.THRESH_OTSU)
```

---

## 7. EasyOCR Text Extraction (Resume Skill)

```python
import easyocr

reader = easyocr.Reader(['en'], gpu=False)
results = reader.readtext('receipt.jpg')

for bbox, text, conf in results:
    if conf > 0.50:
        print(f"Text: '{text}' ({conf * 100:.1f}%)")
```

---

## 8. YOLO11n Inference & Custom Training (Resume Skill)

```python
from ultralytics import YOLO

# 1. Custom Training
model = YOLO("yolo11n.pt")
model.train(data="data.yaml", epochs=50, imgsz=640, batch=16, device=0)

# 2. Prediction on Image/Video
results = model.predict(source="test.jpg", conf=0.35, iou=0.45)
for r in results:
    for box in r.boxes:
        coords = box.xyxy[0].cpu().numpy()  # [x1, y1, x2, y2]
        cls_name = model.names[int(box.cls[0])]
        conf = float(box.conf[0])

# 3. Export to ONNX
model.export(format="onnx", imgsz=640, simplify=True)
```

---

## 9. Real-Time Video Loop with FPS Calculation

```python
import cv2
import time

cap = cv2.VideoCapture(0)
prev_time = time.perf_counter()

while True:
    ret, frame = cap.read()
    if not ret:
        break

    curr_time = time.perf_counter()
    fps = 1.0 / (curr_time - prev_time + 1e-6)
    prev_time = curr_time

    cv2.putText(frame, f"FPS: {fps:.1f}", (20, 40), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)
    cv2.imshow("Video", frame)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```
