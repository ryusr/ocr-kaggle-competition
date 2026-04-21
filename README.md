
# Thai OCR — EasyOCR Pipeline

A OCR workflow for extracting Thai election result tables from scanned page images and producing a Kaggle-ready `submission.csv`.

All processing runs **locally** using EasyOCR. No API keys or external LLM services required.

---

## What It Does

- Reads election result page images from `data/images/`
- Detects table regions with OpenCV before OCR to reduce noise
- Runs page-by-page OCR with Thai + English language support
- Normalizes Thai digits and common OCR digit confusions
- Classifies pages to keep only useful result pages
- Merges multi-page documents into document-level vote rows
- Writes the final output as `data/submission.csv` in Kaggle format: `id,votes`

---

## Setup

```bash
pip install notebook jupyterlab easyocr opencv-python pillow numpy tqdm
```

> **Note:** EasyOCR will install PyTorch as a dependency. GPU acceleration requires a CUDA-compatible PyTorch build but is not required to run.

---

## Data Layout

Place competition files into this structure before running:

```
data/
  images/                   # page images (.png)
  sample_labels/            # sample JSON labels from the competition package
  submission_template.csv
```

The notebook will also generate:

```
data/
  image_cut/                # cropped page images (optional prep step)
  labels/                   # one JSON label per page
  ocr_cache_easyocr/        # cached OCR output for reproducibility
  submission.csv            # final Kaggle submission
  submission.partial.csv    # fault-tolerance checkpoint
```

---

## How To Run

Open `agentic_pipeline_easyOCR.ipynb` and adjust the config flags at the top:

| Flag | Purpose |
|------|---------|
| `RUN_PREPARE_IMAGE_CUT` | Crop/prepare images into `data/image_cut/` |
| `RUN_SCAN_TO_LABELS` | OCR pages and write `data/labels/*.json` |
| `RUN_BUILD_SUBMISSION` | Build `data/submission.csv` from labels |
| `LIMIT_DOCS` | Limit to N documents for quick testing |

Recommended flow:

1. Set `LIMIT_DOCS = 10` for a quick sanity check
2. Enable `RUN_SCAN_TO_LABELS` and inspect the generated labels
3. Enable `RUN_BUILD_SUBMISSION` to produce the CSV
4. Remove `LIMIT_DOCS` for the full run

---

## Output

- `data/labels/*.json` — per-page structured extraction results
- `data/ocr_cache_easyocr/` — cached OCR output for debugging and re-runs
- `data/submission.csv` — final Kaggle submission file
