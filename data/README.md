# Data Directory

This repository is published without the competition dataset.

Place the downloaded competition files in this folder before running the notebooks:

```text
data/
  images/
  sample_labels/
  submission_template.csv
  submission_template_v3.csv
```

Depending on the notebook and flags you run, the pipeline will also create:

```text
data/
  image_cut/
  labels/
  ocr_cache_easyocr/
  repair_cache_groq/
  submission.csv
  submission.partial.csv
```

Do not commit raw images, generated crops, caches, or local secrets.
