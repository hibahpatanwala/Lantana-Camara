# Species Distribution Modelling of *Lantana camara* in India

Predicting habitat suitability of the invasive shrub *Lantana camara* across India using bioclimatic, hydrological and anthropogenic variables. The project compares three multicollinearity-handling strategies (correlation filtering, VIF, PCA) across six machine-learning classifiers.

The best pipeline, **VIF feature selection + tuned Random Forest**, reached **ROC AUC 0.908** on a held-out 30% test set and **0.825 (±0.021)** under spatial block cross-validation.

> Research Discourse II (Semester 5), B.Sc. Applied Mathematical Computing, NMIMS.
> Authors: Hibah Patanwala, Rishabh Tiwari. Supervisor: Jyoti Prajapati.

This work builds on Iyer, Bannerjee & Bhowmick, *Ecological Indicators* (lasso/ridge logistic regression for *Mikania micrantha* in India). We add anthropogenic and hydrological predictors and a wider set of ML classifiers.

---

## Data

| Source | What | Notes |
|---|---|---|
| [GBIF](https://www.gbif.org/) (via `pygbif`) | *L. camara* occurrence records | 3,464 records within India after cleaning |
| [WorldClim](https://worldclim.org/) | 19 bioclimatic variables (BIO1-BIO19) | GeoTIFFs, sampled with `rasterio` |
| [Quick Map Tools](https://www.quickmaptools.com/download-natural-earth) (Natural Earth) | India boundary and shapefiles for roads, railways, urban settlements, rivers | Distances computed with `geopandas.sjoin_nearest` in a metric CRS (EPSG:7755) |
| [HydroLAKES](https://www.hydrosheds.org/products/hydrolakes) | Distance to nearest lake/reservoir | Clipped to India |
| [Köppen-Geiger](https://www.gloh2o.org/koppen/) | Climate zones for background sampling | |

Total: **24 predictors** (19 climatic, 2 hydrological, 3 anthropogenic).
Raw data is **not** committed. See [`data/README.md`](data/README.md) for download instructions.

## Method

1. **Data cleaning**: keep records with valid coordinates and no geospatial issues, clip to India's boundary, keep one record per site to reduce spatial autocorrelation. See `notebooks/00_data_cleaning.ipynb`.
2. **Background points**: 3,464 pseudo-absences (matching the presence count) sampled in proportion to the Köppen-Geiger climate-zone frequencies of the presence points.
3. **Multicollinearity handling** (three pipelines):
   - **Correlation filter** (|r| ≥ 0.5) → 8 predictors
   - **VIF filter** (VIF ≥ 5, removed iteratively) → 13 predictors
   - **PCA** (components retained for ≥ 90% variance)
4. **Models**: Logistic Regression, KNN, Decision Tree, Random Forest, XGBoost, SVM (RBF).
5. **Model comparison**: Monte Carlo simulation (3 iterations, different random splits and seeds) over every pipeline × model combination.
6. **Final model**: VIF + Random Forest, tuned with 5-fold `GridSearchCV` (`n_estimators=200`, `min_samples_split=2`, `max_depth=None`).
7. **Validation**: 30% held-out test set plus spatial block CV on a 2.0° grid.

## Results

**Monte Carlo comparison (mean over iterations, ROC AUC):**

| Model | Correlation | VIF | PCA |
|---|---|---|---|
| Random Forest | 0.9057 | **0.9061** | 0.9015 |
| XGBoost | 0.8948 | 0.8962 | 0.8890 |
| KNN | 0.8937 | 0.8921 | 0.8963 |
| SVM | 0.8578 | 0.8642 | 0.8696 |
| Decision Tree | 0.8191 | 0.8255 | 0.8130 |
| Logistic Regression | 0.8113 | 0.8081 | 0.8183 |

**Final tuned model (held-out test set):**

| Accuracy | Precision | Recall | F1 | ROC AUC |
|---|---|---|---|---|
| 0.8345 | 0.8390 | 0.8277 | 0.8333 | 0.9082 |

**Spatial block CV:** AUC 0.825 (±0.021) versus 0.914 (±0.006) for random-split CV, which shows that random splits give optimistic estimates.

**Top predictors (Random Forest, Gini importance):** distance to urban areas, distance to railways, precipitation of the coldest quarter (BIO19), distance to roads, isothermality (BIO3). This suggests *L. camara* spreads opportunistically along human-disturbed landscapes and transport corridors.

<!-- Add figures: results/occurrences_india.png, results/test_predictions.png -->

## Repository structure

```
Lantana-Camara/
├── data/
│   └── README.md                       # where to get each dataset
├── notebooks/
│   ├── 00_data_cleaning.ipynb          # was: lantana_for_many
│   ├── 01_lantana.ipynb                # [one-line description of lantana.ipynb]
│   ├── 02_variable_selection.ipynb     # was: variable selection method (general overview)
│   ├── 03_corr.ipynb                   # correlation-based selection
│   ├── 04_vif.ipynb                    # VIF filtering
│   ├── 05_pca.ipynb                    # PCA
│   ├── 06_monte_carlo.ipynb            # model comparison across pipelines
│   └── 07_vif_final.ipynb              # was: vif final; tuned Random Forest on VIF features
├── results/                            # figures and metric tables
├── report/Research_Project.pdf
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

Run the notebooks in numeric order. `07_vif_final` is run after the Monte Carlo comparison (`06`) and holds the final tuned model.

## How to run

```bash
git clone https://github.com/hibahpatanwala/Lantana-Camara.git
cd Lantana-Camara
python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook notebooks/
```

Download the datasets listed in `data/README.md` first, and update the file paths at the top of each notebook (they currently point to local folders).

To download GBIF data you need a free GBIF account. Put your credentials in environment variables (`GBIF_USER`, `GBIF_PWD`, `GBIF_EMAIL`), never in the notebook.

## Limitations

- Presence-only data from GBIF is biased towards accessible and well-surveyed areas, and the anthropogenic predictors may partly reflect that sampling bias.
- Pseudo-absences are not true absences.
- The Monte Carlo comparison uses 3 iterations only.
- Static current-climate model; no future climate scenarios.

## References

1. Iyer, Bannerjee & Bhowmick. *Making choices that matter: use of statistical regularization in species distribution modelling for identification of climatic indicators, a case study with Mikania micrantha Kunth in India.* Ecological Indicators.
2. GBIF, WorldClim, Köppen-Geiger (Beck et al.), HydroLAKES.

## License

MIT. See [LICENSE](LICENSE).
