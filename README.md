# Compound Environmental Stressors and Respiratory Health

Do simultaneous exposures to air pollution, extreme heat, and humidity predict respiratory health outcomes better than any single stressor alone?

## Overview

Most epidemiological studies of air quality and respiratory disease use a single pollutant, typically PM2.5, as the primary exposure variable. This project examines compound environmental stressors: days on which high PM2.5, high temperature, and high humidity occur simultaneously, and whether these compound exposure events drive respiratory health outcomes more strongly than any individual factor.

The working hypothesis is that the interaction between stressors carries explanatory power that single-pollutant models miss. Detecting it requires nonlinear methods, which is where machine learning is well suited: tree-based and recurrent models can identify the threshold and interaction effects that conventional regression cannot.

## Research Question

Do compound environmental stressors (air pollution + extreme heat + humidity occurring simultaneously) predict respiratory health outcomes better than any single stressor alone?

## Contribution

| | |
|---|---|
| Multi-stressor framing | Most published work isolates one pollutant; this project models pollution, temperature, and humidity jointly |
| Compound exposure variables | High PM2.5 AND high heat occurring on the same day, rather than averaged separately |
| Nonlinear interaction modeling | Tree ensembles and SHAP attribution to surface threshold and interaction effects |
| Public health translation | SHAP-based interpretability connects model findings to actionable signals for clinicians and public health agencies |

## Approach

| Stage | Method |
|---|---|
| Exposure assembly | County-level daily air quality (EPA AQS), weather (NOAA GSOD), and environmental justice indicators (EPA EJScreen) joined to a county-year panel |
| Outcome data | County-level respiratory mortality (CDC WONDER) and asthma prevalence (CDC PLACES); asthma ED visit rates if API access is granted |
| Feature engineering | Compound exposure days, intensity-weighted compound exposures, lagged variables, cross-stressor interaction terms |
| Modeling | Random Forest baseline, XGBoost primary, optional LSTM for temporal resolution, stacked ensemble where time permits |
| Interpretation | SHAP values to compare feature importance of compound vs single-stressor variables |

## Data

All sources are publicly available and require no IRB review. Detailed data acquisition notes live in [internal-spec.md](internal-spec.md).

| Source | Variable Type | Resolution |
|---|---|---|
| EPA AQS | Air quality (PM2.5, ozone, NO2, SO2) | Daily, monitor-level |
| NOAA GSOD | Temperature, humidity, dew point | Daily, station-level |
| CDC WONDER | Respiratory mortality (ICD-10 J00-J99) | Annual, county-level |
| CDC PLACES | Asthma and COPD prevalence | Annual, county-level |
| Census ACS | Demographics and socioeconomic indicators | 5-year estimates, county-level |
| EPA EJScreen | Environmental justice indicators | Block group, aggregated to county |

## Compute

Analysis runs on Texas Advanced Computing Center (TACC) systems through the National AI Research Resource (NAIRR) Pilot. Tree ensemble training, hyperparameter sweeps, and SHAP attribution use CPU partitions; deep temporal models, when used, run on GPU partitions.

## Status

Active. Currently in the design phase: defining geographic scope and finalizing the primary outcome variable.

## Repository Structure

```
compound-stressors-respiratory/
├── data/         # Raw and processed data (gitignored; retrieval handled by scripts/)
├── scripts/      # Data download and processing scripts
├── notebooks/    # Exploratory and analysis notebooks
├── src/          # Reusable Python modules
├── results/      # Figures, metrics, reports
└── docs/         # Project documentation and Pages site
```

## Author

**Ashley Scruse, Ph.D.**
Deputy Director, Morehouse Supercomputing Facility
Morehouse College
[ashley.scruse@morehouse.edu](mailto:ashley.scruse@morehouse.edu)
