# Soil Moisture Prediction from ERA5 Reanalysis using Machine Learning

Bias-correcting coarse-resolution ERA5-Land soil moisture against flux-tower observations, and validating the result against an independent satellite product.

**M.Tech project, CE 277 Remote Sensing in Ecohydrology, Indian Institute of Science (IISc) Bengaluru.**

---

## The problem

ERA5-Land gives you soil moisture everywhere, continuously, but at coarse resolution and with model assumptions baked in, so it drifts from what is actually happening at a given site. Flux towers give you accurate local measurements, but at a handful of points with gaps in the record.

Neither is enough on its own. This project trains a model to close that gap: use ERA5 meteorology as input, use tower observations as the target, and learn the correction.

The test that matters is whether it generalises. So the trained model is checked against **SMAP satellite retrievals**, which the model never saw during training.

---

## Site and data

| | |
|---|---|
| Site | US-Var, Vaira Ranch, California |
| Coordinates | 38.4133 N, 120.9508 W, 129 m elevation |
| Land cover | Grassland |
| Predictors | ERA5-Land hourly: 2 m air temperature (t2m), dew point (d2m), total precipitation (tp), surface solar radiation (ssrd), volumetric soil water layer 1 (swvl1), wind components (u10, v10) |
| Target | AmeriFlux in-situ soil water content |
| Independent validation | SMAP satellite soil moisture |
| Split | 70:30 train/test |

AmeriFlux half-hourly records were resampled to hourly, invalid values removed, and the two sources aligned on timestamp so only overlapping periods were retained.

---

## Results

### Random Forest on full ERA5 meteorology

| | R2 | RMSE |
|---|---|---|
| Training | 0.88 | 0.036 |
| **Testing** | **0.82** | **0.044** |

### Baseline: ERA5 soil moisture alone

| | R2 | RMSE |
|---|---|---|
| Training | 0.82 | 0.044 |
| **Testing** | **0.73** | **0.055** |

**The full meteorological feature set lifts test R2 from 0.73 to 0.82 and cuts RMSE by 20%.** That difference is the whole point of the project: ERA5 soil moisture on its own is a worse predictor of ground truth than a model that learns from the meteorology driving it.

### What it does not do well

The model tracks the seasonal signal closely but underestimates the peaks. High soil moisture events, which are exactly the conditions that matter for runoff generation and flood response, are where it loses accuracy. That is a real limitation, not a rounding error, and it is the first thing to fix.

---

## Notebooks

| Notebook | What it does |
|---|---|
| `01_random_forest_bias_correction.ipynb` | Main workflow. Loading, cleaning, timestamp alignment, Random Forest and linear baselines, evaluation. |
| `02_hourly_multifeature_xgboost.ipynb` | Hourly multi-feature variant, XGBoost comparison against the Random Forest. |
| `03_smap_independent_validation.ipynb` | Validation against SMAP satellite retrievals, including HDF5 reading and trend comparison across model, ERA5, tower and satellite. |

Built with pandas, NumPy, scikit-learn, XGBoost, h5py, SciPy and matplotlib.

---

## Data

The raw inputs are not committed. AmeriFlux BASE files for this site run to several hundred MB and the six ERA5-Land time series add roughly 85 MB, which does not belong in a git history.

`data/era5_land_soil_water_SAMPLE.csv` holds the first 400 rows of one ERA5 file so the schema is visible.

To reproduce:

| Source | Where |
|---|---|
| ERA5-Land hourly | [Copernicus Climate Data Store](https://cds.climate.copernicus.eu/) |
| AmeriFlux US-Var | [ameriflux.lbl.gov](https://ameriflux.lbl.gov/) (free registration) |
| SMAP L3 soil moisture | [NSIDC](https://nsidc.org/data/smap) |

---

## Why this project

Soil moisture controls how rainfall splits between infiltration, runoff and evapotranspiration, so getting it wrong propagates into every downstream hydrological estimate. The gap between global reanalysis and local truth is a practical obstacle for anyone doing site-specific work with continuous data, and machine learning is a reasonable way to bridge it.

The same pattern, correcting a coarse global product against sparse accurate observations, shows up across climate risk work well beyond soil moisture.

---

## Credits

Author: **Prathmesh Sonvane**, M.Tech Water Resources and Environmental Engineering, IISc Bengaluru
Course: CE 277 Remote Sensing in Ecohydrology, under Prof. Debsunder Dutta
Data: ERA5-Land (Copernicus), AmeriFlux (US-Var, Vaira Ranch), SMAP (NASA)
