# Connecting OCR to GenAI & LLMs — Document AI

## 1. Why Raw OCR is Not Enough

Raw OCR outputs a jumble of disconnected text strings:
```
INVOICE #9481 DATE 12/04/2026 ACME CORP TOTAL DUE $1,450.00 DUE DATE NET 30
```

Real software cannot easily query this string with SQL or trigger an automated bank transfer. You need **structured, typed data**:
```json
{
  "invoice_number": "9481",
  "date": "2026-04-12",
  "vendor": "ACME CORP",
  "total_amount": 1450.00,
  "currency": "USD"
}
```

By connecting your **OCR pipeline** to your **GenAI / LLM Integration skills**, you create a production-grade **Document AI system**.

---

## 2. The Architecture: OCR + LLM Pipeline

```
[Document Image / PDF]
          │
          ▼
[OpenCV Preprocessing] ──► Denoise, Deskew, Binarize
          │
          ▼
[EasyOCR / Tesseract] ──► Extract text tokens + bounding boxes
          │
          ▼
[Reading Order Reconstruction] ──► Sort tokens by (Y-coordinate, X-coordinate)
          │
          ▼
[LLM Prompt with Structured Output] ──► System Prompt + JSON Schema
          │
          ▼
[Pydantic Validation / LangGraph Agent] ──► Validates totals, flags missing fields
          │
          ▼
[Database / Vector DB (RAG)] ──► Ready for semantic search or SQL storage
```

---

## 3. Python Implementation: OCR to Structured JSON

```python
import easyocr
import json

# 1. Run OCR
reader = easyocr.Reader(['en'], gpu=False)
ocr_results = reader.readtext('invoice_sample.jpg')

# 2. Reconstruct raw document text preserving reading order
extracted_lines = [item[1] for item in ocr_results if item[2] > 0.40]
raw_ocr_text = "\n".join(extracted_lines)

# 3. Construct prompt for LLM (OpenAI, Anthropic, or local Ollama)
prompt = f"""
You are an expert document parser. Extract key fields from the following OCR text into valid JSON.

Schema:
- invoice_id (string)
- vendor_name (string)
- invoice_date (YYYY-MM-DD string)
- total_amount (float)
- line_items (list of objects with description, quantity, price)

Raw OCR Text:
\"\"\"
{raw_ocr_text}
\"\"\"

Output ONLY valid JSON. Do not include markdown codeblocks or explanations.
"""

# Example LLM Call (Pseudocode)
# response = llm_client.chat.completions.create(
#     model="gpt-4o-mini",
#     messages=[{"role": "user", "content": prompt}],
#     response_format={"type": "json_object"}
# )
```

---

## 4. Handling OCR Errors with LLM Post-Processing

One of the greatest benefits of pairing OCR with LLMs is **error resilience**:
- If OCR misreads `"$1,45O.OO"` (with capital letters `'O'` instead of zero `'0'`), a regex fails.
- An LLM easily understands the context and corrects it to `1450.00`.
- If an address is broken across multiple bounding boxes, the LLM fuses them naturally.

---

## 5. Fresher Resume & Interview Link

> [!TIP]
> This connects your **Computer Vision (OpenCV, OCR)** skills with your **GenAI (LLM Integration, LangGraph, RAG)** skills!
> In interviews, describe this as an **"Intelligent Document Processing (IDP) Pipeline"**:
> *"I designed an IDP pipeline where OpenCV preprocesses high-resolution scans, EasyOCR extracts character tokens, and an LLM agent validates business logic and populates structured JSON into a database."*
