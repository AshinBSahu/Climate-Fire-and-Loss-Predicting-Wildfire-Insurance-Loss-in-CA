# Climate, Fire, and Loss: Predicting Wildfire Insurance Loss in California

Does adding climate, drought, fire and structure-damage data improve predictions of
county-level wildfire insurance losses in California, compared with insurance data alone?

The project builds a county-by-year dataset (58 counties, 2018-2023), trains three models
(GLM, Random Forest, XGBoost) on two feature sets, and uses SHAP to explain the best
model. The full write-up is in [`CSML_Final_Project.pdf`](CSML_Final_Project.pdf).

## Results (test years 2022-2023, 116 county-years)

Mean absolute error in dollars of wildfire loss, lower is better.

| Model | Insurance-only MAE | Enhanced MAE | Improvement |
|---|---|---|---|
| GLM (Tweedie, log link) | $13.4M | $12.6M | 6.0% |
| Random Forest | $12.4M | $8.8M | 29.0% |
| XGBoost | $5.9M | $4.8M | 18.4% |

- Training years are 2018-2021 (232 county-years). Test years are 2022-2023.
- The enhanced feature set adds temperature, precipitation, drought severity (DSCI),
  burned acres and structures damaged or destroyed, plus one-year lagged versions of
  them.
- Full metrics (MAE, RMSE, R2) are in `Processed Data/enhanced_metrics_summary.csv`.

## Repository layout

```
Data Pre-propcessing.ipynb                   Build the county-year datasets from raw files
Models Training - Insurance Only Data.ipynb  Baseline models on insurance features only
Models Training - Enhanced Data.ipynb        Models on insurance + environmental features
Analysis.ipynb                               SHAP analysis (baseline vs enhanced XGBoost)
CSML_Final_Project.pdf                       Project paper
requirements.txt                             Python dependencies

Data/                                        Raw input files (see "Data" below)
Processed Data/                              Datasets, predictions and saved XGBoost models
Plots/                                       Figures produced by the notebooks
```

The preprocessing notebook's file name contains the spelling "Propcessing". It is left
unchanged so it matches the original project.

## Setup

Python 3.12 was used for development. From the project root:

```bash
python -m venv .venv

# Windows (PowerShell)
.venv\Scripts\Activate.ps1
# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
```

`geopandas` handles the shapefile reading and spatial joins. On some systems it is
easier to install through conda (`conda install -c conda-forge geopandas`).

## How to run

Start Jupyter from the project root so the relative `./Data` path resolves:

```bash
jupyter lab
```

Then run the notebooks in this order:

| Step | Notebook | What it does | Writes |
|---|---|---|---|
| 1 | `Data Pre-propcessing.ipynb` | Loads the raw files, aggregates to county-year, builds lag features | `Processed Data/master_*.csv`, `Plots/correlation_heatmap.png` |
| 2 | `Models Training - Insurance Only Data.ipynb` | Naive, GLM, Random Forest and XGBoost on insurance features | baseline predictions, `xgb_baseline_model.json`, baseline plots |
| 3 | `Models Training - Enhanced Data.ipynb` | Same models with environmental features; compares with Step 2 | enhanced predictions and metrics, `xgb_enhanced_model.json`, maps |
| 4 | `Analysis.ipynb` | SHAP bar and beeswarm plots for the XGBoost models | `Plots/shap_*.png` |

Each notebook sets `BASE = "./Data"` near the top. Change it if you keep the raw data in
another folder. The processed files and plots are written next to `Data/`, in
`Processed Data/` and `Plots/`.

### Rerunning preprocessing from the raw files

The repository already contains the processed datasets. If they exist, the notebooks
load them and skip the heavy steps (Step 0 in each notebook). To rebuild them from the
raw files, set `FORCE_REPROCESS = True` in the "STEP 0" cell of
`Data Pre-propcessing.ipynb`, or delete the `master_*.csv` files in `Processed Data/`.
The first run unzips the fire perimeter and county shapefiles into `Data/.../extracted/`
(ignored by git).

Model results can differ slightly between library versions. Random Forest and XGBoost use
`random_state=42`.

## Data

Included in `Data/` (public data, kept at the file names the notebooks expect):

| Folder | Source | Used for |
|---|---|---|
| `CA Insurance data/` | California Department of Insurance, residential property wildfire risk and losses (SB 824) | Exposure, premium and wildfire loss by county, policy type and year |
| `CA Climate Data/` | NOAA NCEI county temperature and precipitation, one file per year | Climate features |
| `CA Droght Data/` | U.S. Drought Monitor, county Drought Severity and Coverage Index (DSCI) | Drought feature |
| `CA Fire Perimeter Data/` | CAL FIRE historic fire perimeters | Burned acres per county-year |
| `CA Fire Structure Damage Data/` | CAL FIRE Damage Inspection (DINS) | Structures damaged and destroyed |
| `TIGER Shapefiles/` | U.S. Census Bureau TIGER/Line 2025 county boundaries | Assign fires to counties |

The folder name "CA Droght Data" is spelled this way in the code, so it is kept as is.

Three of these files are large (50-80 MB), so cloning takes a while.

## License

The code is released under the [MIT License](LICENSE). The data files keep the terms of
their original publishers, so check them before redistributing.
