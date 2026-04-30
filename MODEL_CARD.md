# Model Card

## Model Name

Temporal Transformer U-Net for monthly water-body gap filling.

## Intended Use

The model is designed to fill residual missing pixels in monthly water-body maps by using a 121-month temporal sequence centered on the target month. The full model additionally uses static auxiliary predictors, including DEM and river-network layers.

## Model Variants

| Variant | File | Static predictors |
| --- | --- | --- |
| Full model | `src/nmwbd/model_full.py` | DEM and river-network layers |
| No-static baseline | `src/nmwbd/model_nostatic.py` | none |

## Input

Full model:

- monthly sequence tensor: `[B, 121, 128, 128]`
- static tensor: `[B, 2, 128, 128]`

No-static baseline:

- monthly sequence tensor: `[B, 121, 128, 128]`

## Output

The model returns one logit map:

- output tensor: `[B, 1, 128, 128]`

After sigmoid activation, pixels above the selected threshold are assigned to water and pixels below the threshold are assigned to non-water.

## Label Convention

| Value | Meaning |
| --- | --- |
| 0 | no observation / padded value |
| 1 | missing pixel to be filled |
| 2 | non-water |
| 3 | water |

## Training Objective

The training script uses a masked weighted binary cross-entropy loss. Training masks hide a proportion of observed water-extent pixels in the target month and optimize the model to recover the water/non-water class.

## Evaluation Metrics

The training scripts log:

- true positive
- false positive
- false negative
- true negative
- F1 score for water
- F1 score for non-water
- loss

## Limitations

- The default implementation assumes input rasters are spatially aligned and share the same grid.
- The temporal window is fixed to 121 months unless the model and preprocessing settings are changed consistently.
- Full training samples, trained weights, and full-resolution outputs may be restricted by data-volume, source-data, or intellectual-property constraints.
