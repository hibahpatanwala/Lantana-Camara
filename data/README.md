# Data

Raw data is not stored in this repository. Download and place files under `data/raw/`.

| Dataset | Source | Used for |
|---|---|---|
| Lantana camara occurrences | GBIF (https://www.gbif.org/), via `pygbif` | Presence points |
| Bioclimatic variables BIO1-BIO19 | WorldClim (https://worldclim.org/) | Climate predictors |
| Köppen-Geiger climate zones | https://www.gloh2o.org/koppen/ | Stratified background sampling |
| India boundary, roads, railways, rivers, cities and towns (shapefiles) | Quick Map Tools, Natural Earth downloads: https://www.quickmaptools.com/download-natural-earth | Clipping to India; distance predictors |
| HydroLAKES | https://www.hydrosheds.org/products/hydrolakes | Distance to lakes |

Processed files (e.g. the final modelling CSV) can go in `data/processed/` if small enough to commit.
