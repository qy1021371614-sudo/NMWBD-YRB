# Code and Data Availability

## Code Availability Statement

The deep learning code used in this study is available in this repository. It includes scripts for sample construction, probabilistic sample grouping, model training, ablation training without static predictors, and monthly gap-filling inference.

The repository does not include large raster datasets or trained model weights by default. Users can reproduce the workflow by preparing monthly water maps and static auxiliary rasters following the file naming and label conventions described in `README.md`. A synthetic demo generator and model smoke test are included to verify the code structure without distributing restricted research data.

## Data Availability Statement Draft

The source remote-sensing datasets used in this study are publicly available from their original providers. The JRC Global Surface Water Monthly Water History product is available from the European Commission Joint Research Centre. Sentinel imagery is available from the Copernicus data services. Static auxiliary layers and basin boundary data should be cited according to their original sources.

The processed monthly water-body maps, training samples, and trained model weights are large derivative files. If no intellectual-property or third-party data restrictions apply, they can be deposited in an open repository such as Zenodo, Figshare, or an institutional repository after manuscript revision. If restrictions apply, access can be provided through a controlled-access repository or upon reasonable request to the corresponding author after intellectual-property review. The final manuscript should clearly state the access mechanism.

## Files That Should Be Uploaded Separately

The following files are not stored in GitHub because of size limits:

- monthly preliminary water maps (`YYYY_MM.tif`)
- DEM raster aligned to the study grid
- river-network raster aligned to the study grid
- maximum water extent mask
- permanent-water masks, if used during inference
- generated `.npy` training samples
- trained model weights (`.pt`), unless public release is approved
- final monthly filled water maps

## Suggested Manuscript Text

Code used for sample construction, model training, ablation experiments, and monthly gap-filling inference is available at: `[GitHub URL to be added]`. A synthetic demonstration dataset generator and model smoke test are included for code verification. Large raster inputs, generated training samples, trained model weights, and final monthly water-body products are available at: `[Data repository DOI/URL to be added]` or from the corresponding author upon reasonable request after intellectual-property review.
