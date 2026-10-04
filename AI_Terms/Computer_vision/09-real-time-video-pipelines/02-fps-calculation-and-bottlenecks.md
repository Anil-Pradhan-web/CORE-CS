# FPS Calculation, Bottlenecks, and Multi-Threading

## 1. How to Accurately Calculate FPS

Never rely on the camera's advertised FPS (`cap.get(cv2.CAP_PROP_FPS)`) — that only tells you the camera sensor's theoretical maximum.

To measure **actual pipeline processing speed**, calculate the time difference between consecutive frames using `time.perf_counter()`:

```python
import cv2
import time

prev_time = time.perf_counter()
fps_smooth = 0.0

cap = cv2.VideoCapture(0)

while True:
    ret, frame = cap.read()
    if not ret:
        break

    # --- Processing happens here ---

    # Measure elapsed time
    curr_time = time.perf_counter()
    time_diff = curr_time - prev_time
    prev_time = curr_time

    # Instantaneous FPS
    instant_fps = 1.0 / (time_diff + 1e-6)

    # Exponential Moving Average (EMA) prevents the on-screen number from flickering wildly
    fps_smooth = (0.9 * fps_smooth) + (0.1 * instant_fps) if fps_smooth > 0 else instant_fps

    # Render FPS overlay onto frame
    cv2.putText(
        frame, 
        f"FPS: {fps_smooth:.1f}", 
        (20, 40), 
        cv2.FONT_HERSHEY_SIMPLEX, 
        1.0, 
        (0, 255, 0), 
        2
    )
    cv2.imshow("Stream", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```

---

## 2. The Anatomy of a Vision Pipeline Bottleneck

If your pipeline is running at only $12\text{ FPS}$, you must profile where the milliseconds are being spent:

$$\text{Total Time per Frame} = T_{\text{capture}} + T_{\text{preprocess}} + T_{\text{inference}} + T_{\text{postprocess}} + T_{\text{render}}$$

| Pipeline Stage | Typical Time (YOLO11n CPU) | Potential Bottleneck & Fix |
|---|---|---|
| **$T_{\text{capture}}$ (Camera I/O)** | $15\text{--}30\text{ ms}$ | **MAJOR BOTTLENECK:** `cap.read()` blocks until camera sensor finishes exposure. Fix with **Threading**. |
| **$T_{\text{preprocess}}$** | $2\text{--}4\text{ ms}$ | High-res image resizing. Keep resizing on GPU if available. |
| **$T_{\text{inference}}$** | $15\text{--}20\text{ ms}$ | Model weight computation. Fix: use **YOLO11n + ONNX Runtime**. |
| **$T_{\text{postprocess}}$ (NMS)** | $1\text{--}3\text{ ms}$ | Filtering boxes. Fast in C++. |
| **$T_{\text{render}}$ (`cv2.imshow`)** | $4\text{--}8\text{ ms}$ | Window painting. Avoid unnecessary overlays. |

---

## 3. The Ultimate Fix: Threaded Video Capture

Standard `cap.read()` is **blocking** — Python pauses and waits idle while the camera captures light.

By running the camera capture on a **separate background thread**, the camera continuously updates the newest frame in memory while your main thread runs **YOLO11n** at maximum speed without waiting!

```python
import cv2
from threading import Thread

class ThreadedCamera:
    def __init__(self, src=0):
        self.cap = cv2.VideoCapture(src)
        self.ret, self.frame = self.cap.read()
        self.stopped = False

    def start(self):
        # Start reading frames on a background thread
        Thread(target=self.update, args=(), daemon=True).start()
        return self

    def update(self):
        while not self.stopped:
            if self.cap.isOpened():
                self.ret, self.frame = self.cap.read()

    def read(self):
        return self.ret, self.frame

    def stop(self):
        self.stopped = True
        self.cap.release()

# Usage:
# cam = ThreadedCamera(0).start()
# ret, frame = cam.read()  # Non-blocking, instant return!
```

> [!TIP]
> Mentioning **Threaded Video Capture** in an interview proves that you are a real-world software engineer who knows how to handle production hardware latency, not just someone who runs tutorial notebooks!
