# Transfer Learning & Fine-Tuning in Vision

## 1. What is Transfer Learning?

**Transfer Learning** is the practice of taking a neural network that has already been trained on a massive, diverse dataset (such as **ImageNet** with 1.4 million images across 1,000 categories) and repurposing its learned visual representations for your custom task.

---

## 2. Why Transfer Learning Works

Remember that CNNs learn hierarchically:
- Early layers learn general visual primitives: edges, curves, color gradients, textures.
- An edge detector trained on ImageNet is **identical** to the edge detector you need for medical X-rays or industrial defect detection!

```
Pretrained on ImageNet (1.4M images)
┌───────────────────────────────────────┐
│ Early Conv Layers (Edges, Textures)   │ ──► [FREEZE - Universal Features]
├───────────────────────────────────────┤
│ Mid Conv Layers (Shapes, Parts)       │ ──► [FREEZE or Fine-Tune]
├───────────────────────────────────────┤
│ Top Fully-Connected Classifier (1000) │ ──► [REPLACE with your Custom Classes: e.g., 2]
└───────────────────────────────────────┘
```

---

## 3. Feature Extraction vs Fine-Tuning

| Strategy | What You Do | When to Use | Learning Rate |
|---|---|---|---|
| **Feature Extraction** | Freeze the entire backbone (`requires_grad = False`). Train **only** the newly attached classification head. | Small custom dataset ($<1,000$ images) similar to ImageNet. | Standard ($10^{-3}$) |
| **Fine-Tuning** | Unfreeze the top convolutional layers of the backbone along with the head. | Medium-to-large dataset or specialized domain (e.g., satellite / medical scans). | Very Small ($10^{-5}$ to avoid destroying pretrained weights) |

---

## 4. PyTorch Implementation Walkthrough

```python
import torch
import torch.nn as nn
from torchvision import models

# 1. Load pretrained ResNet-50
model = models.resnet50(weights=models.ResNet50_Weights.DEFAULT)

# 2. Freeze all convolutional backbone parameters
for param in model.parameters():
    param.requires_grad = False

# 3. Inspect the original classification head
# ResNet's final layer is named 'fc' (Fully Connected), input features = 2048
num_features = model.fc.in_features

# 4. Replace the final layer with our custom classifier (e.g., 2 classes: Defect / No-Defect)
# By default, newly created layers have requires_grad = True
model.fc = nn.Sequential(
    nn.Linear(num_features, 256),
    nn.ReLU(),
    nn.Dropout(0.3),
    nn.Linear(256, 2)  # Output: 2 classes
)

# 5. Pass only the trainable parameters to the optimizer
optimizer = torch.optim.Adam(model.fc.parameters(), lr=1e-3)
criterion = nn.CrossEntropyLoss()
```

---

## 5. Fresher Interview Question: The "Catastrophic Forgetting" Trap

**Q: Why shouldn't you fine-tune a pretrained model with a large learning rate (e.g., $\text{lr} = 0.1$)?**
> **Answer:**
> If the learning rate is too high, the massive initial gradients from the randomly initialized head will propagate back and overwrite the finely tuned feature extractors in the backbone. This causes **Catastrophic Forgetting**, where the model loses its general knowledge of edges and textures. We always train the head first, or use a learning rate $10\times$ to $100\times$ smaller (e.g., $10^{-5}$) when fine-tuning the backbone.
