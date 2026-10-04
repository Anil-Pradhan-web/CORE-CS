# Classification vs Localization vs Object Detection

## 1. The Vision Task Hierarchy

Computer Vision problems progress in complexity from understanding the whole image to pinpointing individual pixels:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 1. Image Classification: "What is in this image?"                           │
│    -> Single label output: "Dog" (No location coordinates)                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ 2. Classification + Localization: "What is it and WHERE is it?"             │
│    -> Single object: Label "Dog" + 1 Bounding Box [x, y, w, h]              │
├─────────────────────────────────────────────────────────────────────────────┤
│ 3. Object Detection: "What and where are ALL the objects?"                  │
│    -> Multiple objects: [("Dog", box1), ("Cat", box2), ("Dog", box3)]       │
├─────────────────────────────────────────────────────────────────────────────┤
│ 4. Instance Segmentation: "Which exact PIXELS belong to each object?"       │
│    -> Pixel-level binary masks for every detected instance                  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Bounding Box Coordinate Formats

Different libraries use different coordinate conventions. Knowing how to convert between them is essential when preparing datasets for **YOLO11n**:

### Format 1: Pascal VOC `[xmin, ymin, xmax, ymax]`
- Used by standard OpenCV drawing (`cv2.rectangle`) and torchvision.
- Defines the **top-left corner** $(x_{\min}, y_{\min})$ and **bottom-right corner** $(x_{\max}, y_{\max})$.

### Format 2: COCO `[xmin, ymin, width, height]`
- Used by the Microsoft COCO dataset.
- Defines the **top-left corner** $(x_{\min}, y_{\min})$ plus the box width and height.

### Format 3: YOLO Format `[class_id, x_center, y_center, width, height]` (Normalized $0.0\text{--}1.0$)
- Used by **YOLO11n** annotation `.txt` files.
- All values are divided by image width and height so coordinates are scale-independent!

```python
def voc_to_yolo(xmin, ymin, xmax, ymax, img_w, img_h):
    # Calculate box center and dimensions
    x_center = ((xmin + xmax) / 2.0) / img_w
    y_center = ((ymin + ymax) / 2.0) / img_h
    w = (xmax - xmin) / img_w
    h = (ymax - ymin) / img_h
    return x_center, y_center, w, h
```

---

## 3. What Does a Detection Model Actually Output?

When you pass an image into an object detector like YOLO, for each detected candidate box it outputs a vector of numbers:

$$\mathbf{Output} = [x_c, y_c, w, h, \text{objectness\_score}, P(\text{class}_1), P(\text{class}_2), \dots, P(\text{class}_C)]$$

- **$x_c, y_c, w, h$:** The bounding box center and size.
- **$\text{objectness}$:** The probability that any object actually exists in that candidate box.
- **$P(\text{class}_i)$:** The conditional probability distribution across all known classes.
