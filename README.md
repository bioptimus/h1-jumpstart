# H-Optimus-1 — General-Purpose Histology Embeddings

**Extract powerful tile-level features directly from H&E slides for downstream machine learning.**

[H-Optimus-1](https://docs.bioptimus.com/documentation/models/h-optimus) is a 1.1B-parameter vision transformer (ViT-g/14) foundation model for histology developed by [Bioptimus](https://www.bioptimus.com). Trained with self-supervised learning on billions of tiles from over one million H&E whole-slide images, it converts each 224×224 tile (0.5 µm/px) into a compact **1536-dimensional feature vector** — a general-purpose fingerprint of tissue morphology. The package also includes a dedicated tissue segmentation model.

| Capability | Output |
|---|---|
| **Tile embeddings** | 1536-dimensional feature vector per 224×224 tile |
| **Tissue segmentation** | Binary tissue mask (bundled model, no extra cost) |
| **Downstream ML** | Features for mutation prediction, survival analysis, tissue classification, and slide retrieval |

## Getting started

The tutorial notebook **[`H-Optimus-1.ipynb`](H-Optimus-1.ipynb)** is an end-to-end, runnable walkthrough: it downloads a demo TCGA-LUAD slide, generates tissue masks, extracts H-Optimus-1 embeddings, and visualizes the results with PCA and nearest-neighbor tiles.

1. Install the SDK: `pip install "bioptimus-sdk[torch]" scikit-learn matplotlib` (the `[torch]` extra is only needed for the `local` backend).
2. Open `H-Optimus-1.ipynb`.
3. In **Section 2 (Configuration)**, choose your deployment backend (see below).
4. Run the cells top to bottom.

## Choosing a deployment option

The notebook runs the same pipeline against any of three backends, selected via the `Backend` enum. Pick **one** and configure it in both the **Configuration** cell (Section 2) and the **`Inference(...)`** call (Section 4) — only one backend block may be active at a time.

| Backend | When to use | Key parameters |
|---|---|---|
| **`remote`** (`Backend.REMOTE`) | You have a running Bioptimus FastAPI server (Docker) | `api_url` |
| **`aws`** (`Backend.AWS`) | You have a deployed SageMaker endpoint | `endpoint_name`, `region_name` |
| **`local`** (`Backend.LOCAL`) | You have a CUDA GPU plus the `.pt2` checkpoints (no server) | `checkpoints`, `device` |

> **Not sure which to pick?** Given a **server URL** → `remote`. Deployed on **AWS SageMaker** → `aws`. Have a **GPU machine plus the model files** → `local` (the notebook's default).

### Backend-specific setup

- **`remote`** — Set `API_URL` (e.g. `http://0.0.0.0:8080`); `utils.check_server(API_URL)` verifies connectivity.
- **`aws`** — Set `ENDPOINT_NAME` and `REGION_NAME`. Requires an AWS account with an active [Marketplace subscription to H-Optimus-1](https://aws.amazon.com/marketplace/pp/prodview-cuad7l27fobx4), an IAM role with **AmazonSageMakerFullAccess**, and quota for `ml.g5.xlarge`. Tissue segmentation is bundled on the same endpoint at no extra cost.
- **`local`** — Requires a CUDA GPU compatible with the exported `.pt2` models (default `sm_86` — A10G, RTX 3090) and two paths, both provided by Bioptimus:
  - `H1_CHECKPOINT` — the H-Optimus-1 `.pt2` checkpoint
  - `TISSUE_SEG_CHECKPOINT` — the tissue-segmentation `.pt2` checkpoint

  The `checkpoints` dict keys (`"h1"`, `"tissue-seg"`) are fixed identifiers — do not rename them.

## Prerequisites

| Requirement | Details |
|---|---|
| **Python** | 3.12+ |
| **SDK** | `bioptimus-sdk` (`[torch]` extra for the `local` backend) |
| **Backend** | One of `remote`, `aws`, or `local` (see above) |
| **Disk space** | ~2 GB slide + ~200 MB outputs for the demo |

The notebook downloads its demo data automatically. To use your own slides, drop whole-slide images (`.svs`, `.tiff`, `.ndpi`, and others) into the WSI directory. Outputs are written as [Zarr](https://zarr.readthedocs.io/) stores under `<output_path>/<experiment>/run_<run>/<variant>/`.

## Documentation

| Resource | URL |
|---|---|
| H-Optimus model page | https://docs.bioptimus.com/documentation/models/h-optimus |
| Tile embeddings & PCA guide | https://docs.bioptimus.com/guides/workflows/embeddings-pca |
| SDK overview | https://docs.bioptimus.com/guides/get-started/sdk |
| Inference facade | https://docs.bioptimus.com/guides/get-started/inference-facade |
| Cohorts guide | https://docs.bioptimus.com/guides/workflows/cohort |
| Visualizing results | https://docs.bioptimus.com/guides/get-started/visualizing-results |
| Choosing a model | https://docs.bioptimus.com/documentation/models/choosing-a-model |
