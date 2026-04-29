# Example Research Project: Compound Environmental Stressors and Respiratory Health Outcomes

## Research Question

Do compound environmental stressors (air pollution + extreme heat + humidity occurring simultaneously) predict respiratory health outcomes better than any single stressor alone?

## Why This Is Publishable

Most published work uses a single pollutant (usually PM2.5) as the predictor. Modeling multiple simultaneous stressors with ML is the gap. Key novelty points:

- Multi-stressor approach (most papers use pollution alone)
- Compound exposure variable (high PM2.5 AND high heat at the same time) is rarely modeled
- ML captures nonlinear interactions between stressors that traditional models miss
- SHAP-based interpretability connects ML outputs back to actionable public health insight

## Target Venues

| Type | Venue |
|---|---|
| Journal | Environmental Research |
| Journal | GeoHealth (AGU) |
| Journal | Science of the Total Environment |
| Journal | Int. J. Environmental Research and Public Health |
| Conference | ISEE (Int'l Society for Environmental Epidemiology) |
| Conference | AGU Fall Meeting (GeoHealth sessions) |
| Conference | APHA (American Public Health Association) |

---

## Data Sources

All datasets are publicly available. No IRB required.

### 1. EPA AQS (Air Quality)

Daily criteria pollutant measurements from ~4,000 monitors nationwide.

| Pollutant | Parameter Code | File Pattern |
|---|---|---|
| PM2.5 (FRM/FEM) | 88101 | `daily_88101_{YEAR}.zip` |
| Ozone (O3) | 44201 | `daily_44201_{YEAR}.zip` |
| NO2 | 42602 | `daily_42602_{YEAR}.zip` |
| SO2 | 42401 | `daily_42401_{YEAR}.zip` |

**Download:** `https://aqs.epa.gov/aqsweb/airdata/daily_{PARAM}_{YEAR}.zip`

**Format:** Zipped CSV, one file per pollutant per year. ~400K-800K rows per file.

**Key columns:** State Code, County Code, Site Num, Date Local, Arithmetic Mean, 1st Max Value, AQI, Latitude, Longitude

**Join key:** State Code (2-digit) + County Code (3-digit) = 5-digit county FIPS

**API (alternative):** `https://aqs.epa.gov/data/api/dailyData/byState?email=X&key=X&param=88101&bdate=20230101&edate=20231231&state=13`

Sign up: `https://aqs.epa.gov/data/api/signup?email=YOUR_EMAIL`

### 2. NOAA GSOD (Weather)

Pre-computed daily weather summaries from global stations.

**Download:** `https://www.ncei.noaa.gov/data/global-summary-of-the-day/access/{YEAR}/{STATION_ID}.csv`

**AWS bulk access (no auth):** `s3://noaa-global-summary-of-the-day-pds/`

**Station list:** `https://www.ncei.noaa.gov/pub/data/noaa/isd-history.csv`

**Key columns:** STATION, DATE, LATITUDE, LONGITUDE, TEMP (mean daily F), DEWP (dew point F), SLP (sea level pressure), WDSP (wind speed knots), MAX, MIN, PRCP

**Derived variables:**
- Relative humidity: computed from TEMP and DEWP
- Heat index: computed from temperature and relative humidity

**Join method:** Spatial join. Assign each county its nearest GSOD station using county centroids (from Census TIGER/Line) and station lat/lon coordinates.

### 3. CDC WONDER (Health Outcomes: Mortality)

County-level mortality counts by ICD-10 code.

**Access:** `https://wonder.cdc.gov`

**Relevant ICD-10 codes:**
- J00-J99: All respiratory diseases
- J45: Asthma
- J40-J44: COPD and bronchitis

**Resolution:** Annual or monthly, county-level, age/race stratified

**Suppression rule:** Counts of 9 or fewer are suppressed. Use broad categories (all J00-J99) and annual totals to minimize suppression.

**Limitation:** This is deaths only, not ER visits or hospitalizations.

### 4. CDC Environmental Public Health Tracking Network (Health Outcomes: ER Visits)

This is the stronger outcome variable if accessible. Tracks asthma ED visits and hospitalizations at state and county level.

**API base:** `https://ephtracking.cdc.gov/apigateway/api/v1/`

**Registration:** `https://ephtracking.cdc.gov/apigateway/api/v1/register`

**Status:** Requires API token registration. Check availability before the workshop. If accessible, use this instead of WONDER for the outcome variable.

### 5. CDC PLACES (Chronic Disease Prevalence)

Model-based prevalence estimates at county and tract level.

**County-level download:** `https://data.cdc.gov/api/views/swc5-untb/rows.csv?accessType=DOWNLOAD`

**Tract-level download:** `https://data.cdc.gov/api/views/cwsq-ngmh/rows.csv?accessType=DOWNLOAD`

**Relevant measures:**
- Current asthma among adults (prevalence)
- COPD among adults (prevalence)
- Current cigarette smoking among adults
- Obesity among adults
- Lack of health insurance among adults 18-64
- No leisure-time physical activity among adults

**Join key:** LocationID = 5-digit county FIPS

### 6. Census ACS (Demographics)

**API:** `https://api.census.gov/data/{YEAR}/acs/acs5`

**Key signup (for heavy use):** `https://api.census.gov/data/key_signup.html`

**Recommended variables (county level, 5-year estimates):**

| Variable | Code | Description |
|---|---|---|
| Total Population | B01001_001E | For denominators |
| Poverty Rate | B17001_002E / B17001_001E | Persons below poverty |
| Median Household Income | B19013_001E | |
| Uninsured Rate | B27001 group | Health insurance coverage |
| Housing Pre-1960 | B25034 group | Year structure built (lead paint proxy) |
| % Non-White | B02001 group | Race breakdown |
| % Age 65+ | B01001_020E-025E + 044E-049E | Elderly population |
| % Under 18 | B09001_001E | Children |

**Example API call:**
```
https://api.census.gov/data/2022/acs/acs5?get=NAME,B17001_001E,B17001_002E,B19013_001E&for=county:*&in=state:13
```

**Join key:** state + county fields = 5-digit FIPS

### 7. EPA EJScreen (Environmental Justice)

**Download:** `https://gaftp.epa.gov/EJScreen/` (check availability, EPA restructures URLs periodically)

**Alternative archive:** `https://zenodo.org/records/14767363`

**Format:** CSV and Geodatabase at Census block group level

**Environmental indicators (11 total):**
1. PM2.5 (annual average)
2. Ozone (summer seasonal average)
3. NATA air toxics cancer risk
4. NATA respiratory hazard index
5. NATA diesel PM
6. Traffic proximity and volume
7. Lead paint indicator
8. Proximity to RMP sites
9. Proximity to TSDFs (hazardous waste)
10. Proximity to NPL (Superfund) sites
11. Proximity to major water dischargers

**Demographic index:** Combines % low-income and % minority at block group level

**Join method:** Block group FIPS (12-digit). Truncate to 5 digits and average within county.

---

## Data Joining Strategy

All datasets join on 5-digit county FIPS code, except NOAA GSOD which requires a spatial join.

| Dataset | Join Key | Method |
|---|---|---|
| EPA AQS | State Code + County Code = 5-digit FIPS | Direct concatenation |
| CDC WONDER | County FIPS in query results | Direct |
| CDC PLACES | LocationID = 5-digit FIPS | Direct |
| Census ACS | state + county API fields = 5-digit FIPS | Direct concatenation |
| EPA EJScreen | Block group FIPS (12-digit) | Truncate to 5 digits, average within county |
| NOAA GSOD | Lat/Lon only | Nearest-station to county centroid or inverse-distance weighting |

For the GSOD spatial join, county centroid coordinates are available from Census TIGER/Line shapefiles or a simple lookup table.

---

## Recommended Scope

| Dimension | Recommendation |
|---|---|
| Geography | 4 states, ~372 counties (pick a region or go national) |
| Time | 2018-2022 (5 years, complete coverage across all sources) |
| Primary outcome | Annual county-level respiratory mortality (CDC WONDER, ICD J00-J99) |
| Alternative outcome | County-level asthma prevalence (CDC PLACES, cross-sectional) |
| Best outcome (if accessible) | Asthma ED visit rates (CDC Tracking Network API) |
| Total data size | Under 1GB |

---

## Feature Engineering

These are the features that make this project novel. Standard approaches use raw pollutant levels. This project engineers compound exposure variables.

**Standard features (per county per year):**
- Mean PM2.5, ozone, NO2, SO2
- Mean temperature, humidity, heat index
- Demographics (poverty rate, insurance, age distribution)
- Baseline disease prevalence (PLACES asthma/COPD rates)
- EJScreen environmental justice indices

**Novel compound features:**
- Days above PM2.5 threshold (35 ug/m3) per county per year
- Days above ozone threshold (70 ppb)
- Heat wave days (consecutive days above 95F)
- **Compound exposure days:** high PM2.5 AND high heat simultaneously
- **Compound exposure intensity:** sum of (PM2.5 x heat index) on compound days
- Lagged exposure variables (prior month, prior season averages)
- Interaction terms across stressors (pollution x heat, pollution x humidity)

---

## Models

| Model | Role |
|---|---|
| Random Forest | Baseline, interpretable |
| XGBoost | Primary model |
| LSTM | If using monthly/daily temporal resolution |
| Ensemble (stacking) | Combine RF + XGBoost + LSTM if time allows |

**Interpretability:** SHAP values for feature importance. The key result is showing that compound exposure features rank higher in importance than any single stressor, which is the publishable finding.

---

## HPC Workflow on Vista via Tapis

### Why this needs HPC

- Joining and processing 5 years of daily data across hundreds of counties and multiple sources
- Spatial joins between weather stations and county centroids
- Training multiple model configurations with hyperparameter sweeps
- Running SHAP analysis across all features and all observations
- GPU acceleration for LSTM/neural net models (gh queue, H200)

### Tapis job flow

1. **Data staging job:** Download all datasets to $SCRATCH on Vista
2. **Preprocessing job:** Clean, join, and build the county-year panel dataset
3. **Feature engineering job:** Compute compound exposure variables from daily data
4. **Model training job:** Train RF, XGBoost, LSTM with hyperparameter grid search
5. **Evaluation job:** Generate SHAP plots, geographic visualizations, performance metrics

Each step is a Tapis job submitted via tapipy or the Morehouse tenant UI.

---

## Project TODOs

- [ ] Register for CDC Tracking Network API token (best outcome variable)
- [ ] Register for EPA AQS API key
- [ ] Register for Census API key (optional, bulk download works too)
- [ ] Decide geographic scope (specific region vs. national)
- [ ] Download all datasets and test the join pipeline
- [ ] Build and test the full notebook on the chosen HPC system
- [ ] Run the models and confirm results support the hypothesis
