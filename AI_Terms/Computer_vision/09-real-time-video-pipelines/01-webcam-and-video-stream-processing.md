# Webcam & Video Stream Processing

## 1. Video as a Stream of Frames

In Computer Vision, a video is nothing more than a continuous sequence of still image frames played at high speed (typically $24\text{ to }60\text{ frames per second}$).

When building real-time vision pipelines with **OpenCV** and **YOLO11n**, your code must read, process, annotate, and render each frame before the camera produces the next one.

---

## 2. The Core Video Processing Loop

```python
import cv2
import time
from ultralytics import YOLO

# 1. Initialize detector
model = YOLO("yolo11n.pt")

# 2. Open camera (0 = built-in webcam; or pass RTSP stream URL: "rtsp://admin:pass@ip:554/...")
cap = cv2.VideoCapture(0)

# Set camera resolution (optional)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)

if not cap.isOpened():
    print("Error: Could not access video source.")
    exit()

print("Camera stream running. Press 'q' to quit.")

while True:
    ret, frame = cap.read()
    if not ret:
        print("Dropped frame or stream finished.")
        break

    # 3. Model inference on current frame
    results = model(frame, conf=0.35, verbose=False)

    # 4. Draw detections on frame using Ultralytics built-in plot or custom OpenCV
    annotated_frame = results[0].plot()

    # 5. Display the annotated frame
    cv2.imshow("Real-Time Vision Stream", annotated_frame)

    # Exit on 'q' press
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# Clean up resources
cap.release()
cv2.destroyAllWindows()
```

---

## 3. Best Practices for Video Processing

1. **`verbose=False`:** Always pass `verbose=False` into `model(frame)` inside a video loop. Printing lines to the terminal in Python is synchronous and drastically reduces your framerate.
2. **Buffer Bloat:** If your model takes $50\text{ ms}$ to process a frame but the camera produces frames every $33\text{ ms}$ ($30\text{ FPS}$), the OpenCV camera buffer fills up, creating a noticeable $2\text{--}5\text{ second}$ delay behind real life.
3. **Drop Frames if Lagging:** For security and robotics, always process the *most recent* frame rather than queued historical frames.
