# SGTU-Net-YRB: Static-Guided Temporal U-Net for Monthly Water-Body Gap Filling

This repository contains the deep learning component used for monthly water-body gap filling in the manuscript:

**Near-real-time monthly water-body mapping for the Yellow River Basin by cascading JRC Monthly Water History and Sentinel imagery**

The model is referred to in the manuscript as **Static-Guided Temporal U-Net (SGTU-Net)**. It combines a U-Net spatial encoder-decoder with a temporal Transformer bottleneck. The full model uses monthly water observations together with static auxiliary layers, including DEM and river-network information. A no-static baseline is also provided for ablation experiments.

## Repository Structure

```text
SGTU-Net-YRB/
  configs/
    example_config.yaml
  scripts/
    build_samples.py
    classify_samples.py
    create_demo_data.py
    infer_gapfill.py
    train_full.py
    train_nostatic.py
  src/
    nmwbd/
      config.py
      dataset.py
      metrics.py
      model_full.py
      model_nostatic.py
      sampling.py
  CODE_AND_DATA_AVAILABILITY.md
  DATA_INVENTORY_TEMPLATE.md
  LICENSE
  MODEL_CARD.md
  README.md
  requirements.txt
  tests/
    test_model_smoke.py
```

## Main Workflow

1. Prepare monthly preliminary water maps as GeoTIFF files named `YYYY_MM.tif`.
2. Prepare static raster layers aligned with the monthly maps:
   - DEM raster
   - river-network raster
   - maximum water extent mask
3. Build training samples as `.npy` tensors.
4. Classify samples into month, maximum-water-extent, and occurrence-frequency groups for balanced probabilistic sampling.
5. Train SGTU-Net or the no-static ablation model.
6. Apply the trained model to fill residual missing pixels in monthly water maps.

## Label Convention

The current code follows the label convention used in the manuscript workflow:

| Value | Meaning |
| --- | --- |
| 0 | no observation / padded value |
| 1 | missing pixel to be filled |
| 2 | non-water |
| 3 | water |

If your exported rasters use different labels, update the conversion rules before training or inference.

## Installation

Create a Python environment and install dependencies:

```bash
pip install -r requirements.txt
```

PyTorch should be installed following the official instructions for your CUDA version when GPU training is required.

## Quick Code Check

The model definitions can be checked without any research data:

```bash
python tests/test_model_smoke.py
```

This smoke test only verifies that the full model and the no-static baseline can complete a forward pass and return the expected output shape.

## Demo Data

For reviewers who want to verify the file structure and command-line workflow without accessing the full research dataset, a tiny synthetic demo dataset can be generated:

```bash
python scripts/create_demo_data.py --output-dir demo_data
```

The synthetic demo is not used for scientific evaluation. It only demonstrates the expected GeoTIFF naming convention, label convention, static raster inputs, and sample-point text format.

## Configuration

Copy `configs/example_config.yaml` and edit paths for your local data:

```bash
cp configs/example_config.yaml configs/local_config.yaml
```

All scripts accept command-line arguments, so the repository does not depend on private local paths.

## Build Training Samples

```bash
python scripts/build_samples.py \
  --sample-txt data/sample_points.txt \
  --monthly-dir data/preliminary_maps \
  --dem data/static/DEM_YRB.tif \
  --river data/static/GRWL_YRB.tif \
  --output-dir data/samples/npy
```

Each sample is saved as a tensor with shape `[123, 128, 128]`:

- first 121 bands: monthly water sequence centered on the target month
- band 122: DEM patch
- band 123: river-network patch

## Classify Samples for Probabilistic Sampling

```bash
python scripts/classify_samples.py \
  --source-dir data/samples/npy \
  --target-dir data/samples/classified \
  --weights-out data/samples/class_weights.tsv
```

The classifier groups samples into 1200 categories: 12 months x 10 maximum-water-extent bins x 10 occurrence-frequency bins. The output weight table is used by the training dataset.

## Train Full Model

```bash
python scripts/train_full.py \
  --sample-root data/samples/classified \
  --prob-table data/samples/class_weights.tsv \
  --water-mask data/static/MaxWaterExtent_YRB.tif \
  --log-xlsx outputs/loss_full.xlsx \
  --model-out outputs/best_model_full.pt
```

## Train No-Static Baseline

```bash
python scripts/train_nostatic.py \
  --sample-root data/samples/classified \
  --prob-table data/samples/class_weights.tsv \
  --water-mask data/static/MaxWaterExtent_YRB.tif \
  --log-xlsx outputs/loss_nostatic.xlsx \
  --model-out outputs/best_model_nostatic.pt
```

## Gap-Filling Inference

```bash
python scripts/infer_gapfill.py \
  --monthly-dir data/preliminary_maps \
  --output-dir outputs/filled_maps \
  --dem data/static/DEM_YRB.tif \
  --river data/static/GRWL_YRB.tif \
  --permanent-water-dir data/static/permanent_water \
  --model outputs/best_model_full.pt \
  --start 2000-01 \
  --months 264 \
  --overlap 5
```

## Notes for Reproducibility

- The training samples are generated from aligned monthly GeoTIFF rasters.
- The default temporal window is 121 months, using 60 months before and after the target month.
- The default patch size is 128 x 128 pixels.
- The default inference overlap is five pixels to reduce boundary artifacts.
- The full model uses DEM and river-network static predictors.
- The no-static model is included as an ablation baseline.
- Random sampling is used during training; set `--seed` to improve reproducibility.

## Intellectual Property and Restricted Artifacts

The source code, model architecture, preprocessing workflow, and command-line interfaces are provided for transparency and reproducibility assessment. Full training samples, full-resolution monthly products, and trained model weights may be restricted because of file size, third-party data terms, or intellectual-property review. When these artifacts cannot be released publicly, they should be deposited in a controlled-access repository or made available from the corresponding author upon reasonable request and after intellectual-property review.

Additional documentation is provided in `MODEL_CARD.md` and `DATA_INVENTORY_TEMPLATE.md`.

## Citation

If this repository is made public after manuscript revision, add the final manuscript citation or preprint DOI here.
