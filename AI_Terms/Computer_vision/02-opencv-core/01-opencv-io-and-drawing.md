# OpenCV Basics — I/O, Video Capture, and Drawing

## 1. What is it?

OpenCV (**Open Source Computer Vision Library**) is the industry-standard C++/Python library for image processing and computer vision.

Every CV project — from drawing bounding boxes around **YOLO11n** detections to drawing defect boxes in **SSIM Anomaly Detection** — starts with reading, writing, displaying, and drawing on images.

---

## 2. Basic Image I/O

```python
import cv2

# 1. Read an image
# Flags: cv2.IMREAD_COLOR (default), cv2.IMREAD_GRAYSCALE, cv2.IMREAD_UNCHANGED
img = cv2.imread("input.jpg", cv2.IMREAD_COLOR)

if img is None:
    raise FileNotFoundError("Could not load image. Check the file path!")

# 2. Display the image in a window
cv2.imshow("Preview Window", img)

# 3. Wait for a key press (0 means wait indefinitely; 1 means wait 1ms)
cv2.waitKey(0)

# 4. Save/Write the image to disk
cv2.imwrite("output.png", img)

# 5. Clean up open windows
cv2.destroyAllWindows()
```

> [!IMPORTANT]
> Always pair `cv2.imshow()` with `cv2.waitKey()`. Without `cv2.waitKey()`, the operating system will not render the window and it will freeze or show a grey box.

---

## 3. Video Capture Loop (Webcam / Video File)

In computer vision, a video is simply a fast sequence of still images (frames).

```python
import cv2

# 0 opens the default webcam; or pass a video filename string: "video.mp4"
cap = cv2.VideoCapture(0)

if not cap.isOpened():
    print("Error: Could not open camera.")
    exit()

while True:
    ret, frame = cap.read()
    if not ret:
        print("End of stream or cannot receive frame.")
        break

    # Process frame here (e.g., pass frame to YOLO11n or OCR)
    
    # Display the processed frame
    cv2.imshow("Live Feed", frame)

    # Press 'q' on keyboard to exit the loop
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# Always release camera hardware and close windows
cap.release()
cv2.destroyAllWindows()
```

---

## 4. Drawing Overlays (Bounding Boxes & Labels)

This code is used in almost every object detection and anomaly detection pipeline:

```python
import cv2

# Draw a Green Bounding Box: (x1, y1) to (x2, y2)
# Color in BGR: (0, 255, 0) is Pure Green, thickness = 2
cv2.rectangle(img, (50, 50), (250, 300), (0, 255, 0), thickness=2)

# Draw a filled Blue Circle: center (x, y), radius = 10, thickness = -1 (filled)
cv2.circle(img, (150, 150), 10, (255, 0, 0), thickness=-1)

# Draw a Red Line: start point to end point
cv2.line(img, (50, 50), (250, 50), (0, 0, 255), thickness=2)

# Put Text Label above the bounding box
label = "Defect: 94%"
cv2.putText(
    img, 
    label, 
    (50, 40),                          # Coordinate (X, Y)
    cv2.FONT_HERSHEY_SIMPLEX,          # Font style
    0.6,                               # Font scale
    (0, 255, 0),                       # Color (Green in BGR)
    2                                  # Thickness
)
```

---

## 5. Fresher Interview Tip: Drawing In-Place

> [!NOTE]
> All OpenCV drawing functions (`cv2.rectangle`, `cv2.putText`, `cv2.circle`) **modify the input image array in-place**.
> If you want to keep the original unmodified image, always create an explicit copy first:
> `annotated_img = img.copy()`
