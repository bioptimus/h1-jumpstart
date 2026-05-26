# H-optimus-1 — AWS SageMaker Jumpstart

H-optimus-1 is a 1.1B parameter vision transformer foundation model for histology, developed by Bioptimus. Trained on over one million H&E stained whole slide images, it extracts powerful tile-level features for downstream tasks such as mutation prediction, survival analysis, and tissue classification.

This repository provides sample notebooks and example data for deploying and testing H-optimus-1 (and the bundled tissue segmentation model) via AWS Marketplace and Amazon SageMaker.

For more information: [bioptimus.com/h-optimus-1](https://www.bioptimus.com/h-optimus-1)

---

## Notebooks

### `H-optimus-1_v2.ipynb` — SageMaker deployment reference

A step-by-step reference notebook for deploying H-optimus-1 from AWS Marketplace and running inference on pre-built example payloads. Covers:

1. Subscribing to the model package and creating a SageMaker endpoint
2. Real-time embedding inference (single tile)
3. Real-time tissue segmentation inference
4. Batch transform inference (JSONL input)
5. Cleanup

> This is the recommended starting point for integrating H-optimus-1 into a pipeline.

### `H-optimus-1-tiling_v2.ipynb` — End-to-end tiling tutorial

An interactive tutorial designed to run on Google Colab or a SageMaker Notebook Instance. Covers the full workflow from a whole slide image to a PCA feature map:

1. Installing dependencies and configuring AWS credentials
2. Deploying the H-optimus-1 endpoint
3. Loading a sample lung carcinoma WSI and tiling with `lazyslide` at 0.5 MPP
4. Testing a single tile against the live endpoint
5. Processing all tiles in parallel (8 threads)
6. Visualising embeddings as a PCA RGB feature map
7. Cleanup

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/bioptimus/h1-jumpstart/blob/main/H-optimus-1-tiling.ipynb)

---

## Prerequisites

1. An AWS account with an IAM role that has `AmazonSageMakerFullAccess`
2. A subscription to [H-optimus-1 on AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-cuad7l27fobx4) — required to obtain the Model Package ARN
3. Sufficient SageMaker service quota for `ml.g5.xlarge` instances in your region

---

## Input data

Example payloads are provided under `data/input/`.

```
data/
  input/
    real-time/
      h1_embed_input.json        # Single tile, embedding mode
      tissue_seg_input.json      # Single 512×512 tile, tissue segmentation mode
    batch/
      h1_embed_inputs.jsonl      # 20 tiles, embedding mode (one JSON record per line)
```

Each embedding payload is a JSON object with the following fields:

| Field | Type | Description |
|---|---|---|
| `image_data` | string | Base64-encoded PNG tile |
| `slide_name` | string | Identifier for the source slide |
| `x`, `y` | int | Tile top-left coordinates in the slide (pixels) |
| `width`, `height` | int | Tile size in pixels (224×224 for H-optimus-1) |
| `tissue_ratio` | float \| null | Fraction of the tile covered by tissue |
| `patch_idx` | int | Index of this tile in the extraction plan |
| `resolution` | float \| null | Extraction resolution in microns per pixel (0.5 for H-optimus-1) |
| `model_name` | string | Server-side dispatch key: `"h1"` for embedding, `"tissue-seg"` for segmentation |
| `mode` | string | `"embedding"` for H-optimus-1; `"prediction"` for tissue segmentation |

The tissue segmentation payload uses the same schema with `width`/`height` = 512, `resolution` = 8.0, `model_name` = `"tissue-seg"`, and `mode` = `"prediction"`.

---

## Models

### H-optimus-1 — tile embedding

- **Input:** 224×224 tile at 0.5 µm/px, RGB
- **Output:** 1536-dimensional feature vector (`result["output"]`)
- **SageMaker dispatch:** `model_name: "h1"`, `mode: "embedding"`
- **Instance type:** `ml.g5.xlarge`

### Tissue segmentation

- **Input:** 512×512 tile at 8.0 µm/px, RGB
- **Output:** Flattened binary mask of length 262,144 (512×512), values 0.0 or 1.0 (`result["output"]`)
- **SageMaker dispatch:** `model_name: "tissue-seg"`, `mode: "prediction"`
- **Instance type:** `ml.g5.xlarge`

---

## Output format

All endpoints return a JSON object matching the `ModelResponse` schema:

```json
{
  "output": [/* float array */],
  "slide_name": "tcga_coad",
  "x": 1793,
  "y": 20173,
  "width": 224,
  "height": 224,
  "tissue_ratio": 0.77,
  "patch_idx": 0,
  "resolution": 0.5
}
```

Example outputs are provided under `data/output/` for both real-time and batch modes.

---

## Dependencies

```
sagemaker==2.254.1
boto3==1.42.2
```

The tiling notebook additionally requires `lazyslide` (for WSI handling) and standard scientific Python libraries (`numpy`, `scikit-learn`, `matplotlib`, `Pillow`).

---

## Legacy files

The original v1 notebooks (`H-optimus-1.ipynb`, `H-optimus-1-tiling.ipynb`) and their associated PNG example inputs (`data/input/real-time/example_input.png`, `data/input/batch/example_input_*.png`) and raw-array outputs (`data/output/*/example_output.json`) are retained for compatibility with the v1 model package API.
