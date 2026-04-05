# Thai Election OCR Kaggle Pipeline

This repository contains a notebook-first OCR workflow for extracting Thai election result tables from scanned page images and converting them into Kaggle-ready `submission.csv` files.

The project uses EasyOCR as the main extraction engine, OpenCV-based table-region detection to avoid OCRing the whole page, and an optional Groq-powered repair pass for difficult pages.

## What This Project Does

- Reads election result page images from `data/images/`
- Detects table regions before OCR to reduce noise
- Runs page-by-page OCR with Thai + English language support
- Normalizes Thai digits and common OCR digit confusions
- Classifies pages to keep only useful result pages
- Merges multi-page documents back into document-level vote rows
- Writes final outputs in Kaggle submission format: `id,votes`

## Project Snapshot

The local working copy this repo was prepared from contains:

- 300 documents
- 847 page images
- 150 `constituency_*` documents
- 150 `party_list_*` documents

The raw dataset is intentionally excluded from GitHub.

## Main Notebooks

### `agentic_pipeline_easyOCR_update.ipynb`

Recommended baseline notebook. It contains the most complete EasyOCR workflow and utility steps, including:

- optional `image_cut` preparation
- page OCR
- JSON label generation
- submission building
- helper utilities for label lists and post-processing

### `agentic_pipeline_optimize.ipynb`

Hybrid notebook for harder cases. It keeps EasyOCR as the base extractor and optionally sends selected difficult pages to Groq (`meta-llama/llama-4-scout-17b-16e-instruct`) for targeted repair before building the final submission.

### `agentic_pipeline_easyOCR.ipynb`

Earlier baseline version focused on building `submission.csv` from page labels.

### `agentic_pipeline_easyOCR 2.ipynb`

Another EasyOCR pipeline variant with scan and build stages separated more explicitly.

## Expected Local Data Layout

This repo is published without the competition dataset. After downloading the competition files locally, place them into this structure:

```text
data/
  images/                     # page images (.png)
  sample_labels/              # sample JSON labels from the competition package
  submission_template.csv     # or "submission_template copy.csv"
  submission_template_v3.csv
```

During execution, the notebooks also create:

```text
data/
  image_cut/                  # cropped or prepared page images
  labels/                     # one JSON label per page
  ocr_cache_easyocr/          # page-level OCR cache
  repair_cache_groq/          # optional LLM repair payloads
  submission.csv              # final Kaggle submission
  submission.partial.csv      # temporary fault-tolerance file
```

## Setup

1. Create and activate a Python environment.
2. Install the notebook dependencies you plan to use.
3. Put the competition files into `data/`.
4. Open the notebook you want to run.
5. Toggle the execution flags in the config cell.

Example package install:

```bash
pip install notebook jupyterlab easyocr opencv-python pillow numpy tqdm openai
```

Notes:

- EasyOCR may install or require PyTorch depending on your environment.
- GPU acceleration depends on a CUDA-compatible PyTorch setup.
- The notebooks already include some lightweight install checks for missing packages.

## Optional Environment Variables

Create a local `.env` file only if you want to use the hybrid repair notebook:

```env
GROQ_API_KEY=your_groq_api_key
```

The baseline EasyOCR notebooks do not require an API key.

## How To Run

### Baseline EasyOCR Flow

Use `agentic_pipeline_easyOCR_update.ipynb` and adjust these flags in the config section:

- `RUN_PREPARE_IMAGE_CUT = True` if you want to generate `data/image_cut/`
- `RUN_SCAN_TO_LABELS = True` to OCR pages and write `data/labels/*.json`
- `RUN_BUILD_SUBMISSION = True` to build `data/submission.csv`
- `LIMIT_DOCS = 10` for quick tests on a subset

Recommended progression:

1. Start with `LIMIT_DOCS`
2. Run OCR and inspect generated labels
3. Build `submission.csv`
4. Remove `LIMIT_DOCS` for the full run

### Hybrid Repair Flow

Use `agentic_pipeline_optimize.ipynb` when the baseline misses too many rows:

- `RUN_SCAN_TO_LABELS = True` to generate base labels
- `RUN_REPAIR_HARD_CASES = True` to repair selected pages with Groq
- `RUN_BUILD_SUBMISSION = True` to merge repaired rows into the final output

Important flags in that notebook:

- `REPAIR_PAGE_LIMIT`
- `REPAIR_MIN_TABLE_SCORE`
- `REPAIR_DOC_COVERAGE_PARTY_LIST`
- `REPAIR_DOC_COVERAGE_CONSTITUENCY`
- `USE_REPAIR_OVERRIDES_IN_BUILD`

## Output Files

- `data/labels/*.json`: per-page structured extraction results
- `data/ocr_cache_easyocr/`: cached OCR output for reproducibility/debugging
- `data/repair_cache_groq/*.json`: repaired page payloads from the hybrid notebook
- `data/submission.csv`: final Kaggle submission file

## Repository Notes

- The dataset is not included in this GitHub version.
- `.env` and generated artifacts are ignored to prevent secret leakage and oversized commits.
- This repository is notebook-first, so most pipeline logic currently lives inside `.ipynb` files rather than Python modules.

## Next Improvements

- refactor shared logic from notebooks into reusable Python modules
- add a pinned `requirements.txt` or `environment.yml`
- add evaluation scripts for comparing submission variants
- add sample visual outputs and error analysis examples
