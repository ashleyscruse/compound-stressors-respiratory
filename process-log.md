# Process Log

A chronological record of the concrete steps taken to execute this project, organized by framework stage. Each item is checked off when done, with a short note about what worked, what failed, or what surprised me. Together with [framework-mapping.md](framework-mapping.md), this serves as the case-study record of the project.

---

## Stage 1: Ideation

**Goal:** confirm gap, viable data, target venue, and a one-sentence research question. Cross the hard line into Stage 2 only when all four are verified.

### On-paper deliverables

- [x] Research question stated in one sentence (`README.md`)
- [x] Gap identified and articulated (`README.md` Contribution section, `internal-spec.md` "Why This Is Publishable")
- [x] Data sources documented with URLs, formats, and join keys (`internal-spec.md`)
- [x] Target venues identified (`internal-spec.md`)

### API key registrations (background, takes 1-2 days to issue)

- [x] EPA AQS API key — register by visiting `https://aqs.epa.gov/data/api/signup?email=YOUR_EMAIL` (the email must be a URL parameter, not a form field; key arrives by email within ~1 minute). **Done 2026-04-29; key stored in local `.env` (not committed).**
- [x] CDC Tracking Network API token — **token is OPTIONAL, not required.** No web registration form. Email `nephtrackingsupport@cdc.gov` to request a token if expanded access is needed later. Spec doc had this wrong.
- [x] Census API key — register at https://api.census.gov/data/key_signup.html. **Done 2026-04-29; key stored in local `.env` (not committed). Issued within minutes via email.**

### Smoke tests for keyless sources

- [x] **NOAA GSOD**: pulled Atlanta Hartsfield station, full year 2022, via HTTPS direct download (96 KB CSV, 365 daily rows). All required columns present: TEMP, DEWP, MAX, MIN, PRCP, plus station lat/lon. **No FIPS in file** — spatial join (station to county centroid) required as spec anticipated.
- [x] **CDC PLACES**: downloaded county-level CSV (51 MB). Asthma + COPD measures present ("Current asthma among adults", "Chronic obstructive pulmonary disease among adults"). LocationID is 5-digit county FIPS as spec said. **Major caveat:** only 2022 and 2023 are in the dataset, not the full 2018-2022 window the spec assumed. PLACES is a model-based estimate with smoothed multi-year inputs; fine for cross-sectional analysis, thin for a year-by-year temporal panel. **Stage 2 decision: keep PLACES as a covariate or upgrade primary outcome strategy.**
- [x] **CDC WONDER**: query for Georgia, 2022, J00-J98 (Diseases of the respiratory system), grouped by County. Returned 137 of 159 GA counties (86%); 22 small counties suppressed by the 9-or-fewer rule. Format: tab-separated text with `.xls` extension. Has County Code (5-digit FIPS), Deaths, Population, Crude Rate, 95% CI. Suppression manageable for J00-J98 at one year; aggregating across multiple years closes most gaps.
- [x] **EPA EJScreen**: primary EPA URL (`gaftp.epa.gov/EJScreen/`) returned 404 — confirmed dead, as spec warned. **Zenodo backup at https://zenodo.org/records/14767363 is the live source.** Years 2018-2022 all available (1.6-3.5 GB each). Tech doc (788 KB) downloaded to `references/` for column reference. Format: CSV/GDB at block group level (242,336 block groups for 50 states + DC + PR), built from 2022 Census TIGER/Line. Block group FIPS (12-digit) needs to be truncated to 5-digit county FIPS and aggregated for our analysis.

### Smoke tests for keyed sources (run after keys arrive)

- [x] **EPA AQS API**: PM2.5 (parameter 88101) for Georgia (state 13) on 2022-07-15. Status: Success. 289 records (one row per monitor-day across all GA monitoring sites). Each record has `state_code` + `county_code` (3-digit zero-padded), daily `arithmetic_mean`, `first_max_value`, lat/lon, sometimes `aqi`. **Subtlety:** multiple monitors and POCs (Parameter Occurrence Codes) per county; Stage 3 must aggregate to county-day. AQI field is null for some daily-mean records.
- [x] **CDC Tracking Network**: one query for asthma ED visits, one state; verify response and check whether county-level data is exposed. **Done 2026-04-29.** Findings: API reachable without token; asthma ED visits available (indicator 90, measure 902); **only at "20K Minimum Population Area" geography, not standard county FIPS** (CDC privacy aggregation). Decision to make in Stage 2: keep ED visits as primary outcome and aggregate exposure data to 20K MPA, or switch primary outcome to CDC PLACES asthma prevalence (county FIPS).
- [x] **Census ACS API**: 2022 5-year estimates, all GA counties, variables NAME + B01001_001E (population) + B17001_001E/_002E (poverty universe + below) + B19013_001E (median income). 159 of 159 counties returned (100%, no suppression at county level). State + county fields concatenate cleanly to 5-digit FIPS (state=13, county=001 → 13001). Sanity check: Appling County population 18,441 here vs 18,428 in WONDER — same order of magnitude (5-yr ACS estimate vs single-year WONDER estimate; slight differences are expected).

### Cross-source verification

- [x] **Confirm county FIPS is consistently 5-digit string (or convertible) across every source pulled.** Verified using Appling County, GA (FIPS 13001) as anchor: PLACES `LocationID = 13001`, WONDER `County Code = 13001`, Census ACS `state=13 + county=001 → 13001`, EPA AQS `state_code=13 + county_code=NNN → 13NNN` (all 3-digit zero-padded county codes). EPA EJScreen requires 12-to-5-digit truncation. NOAA GSOD requires spatial join. CDC Tracking Network requires 20K MPA crosswalk. **All FIPS-keyed sources align cleanly; non-FIPS sources have documented join strategies.**
- [x] **Confirm column names in each pulled file match what's documented in `internal-spec.md`.** Confirmed for all sources. CDC Tracking Network needed re-mapping from "indicator/measure" terminology to specific measure IDs (902 for asthma ED visits at 20K MPA, annual, age-adjusted) — this should be added to the spec.
- [ ] **Update `internal-spec.md`** with corrections: (1) CDC Tracking Network registration is by email to `nephtrackingsupport@cdc.gov`, not via the URL listed; (2) PLACES coverage is 2022-2023 only, not 2018-2022; (3) EPA EJScreen primary URL is dead, use Zenodo backup; (4) CDC Tracking Network ED visits available only at 20K MPA geography, not direct county FIPS.

### Hard line check

- [x] **All four Stage 1 deliverables confirmed:** (1) Gap — multi-stressor compound exposure modeling is rare in published literature; (2) Data viability — all 7 sources reachable, all FIPS encodings consistent, all required variables present; (3) Target venue — list of journals and conferences in `internal-spec.md`; (4) Research question — single-sentence formulation in `README.md`. **Stage 1 is closed; Stage 2 (Design) is open.**

**Notes:**

**2026-04-29 (full Stage 1 closeout in one session):**

Process observations worth flagging for the case study:

- **Spec accuracy mattered.** Of 7 sources documented in the spec, 4 had inaccuracies that only surfaced during smoke testing: CDC Tracking Network registration URL was actually the API endpoint (not a form), PLACES temporal coverage was overstated, EPA EJScreen primary URL was dead, and CDC Tracking Network's geography wasn't standard county FIPS. None blocked the project, but all of them would have wasted time at Stage 3 if discovered then. **Lesson for the framework: the Stage 1 hard line on "viable data" must mean "smoke-tested," not "documented."**
- **Smoke tests are cheap.** Total time for all 7 sources was under 2 hours. The cost of NOT doing them is days of Stage 3 backtracking.
- **CDC's 9-or-fewer suppression** affects WONDER (lost 22 of 159 GA counties for one year), making multi-year aggregation likely necessary for small-county coverage.
- **CDC Tracking Network's 20K MPA geography** is the most consequential finding. It forces a Stage 2 decision: keep the strongest outcome (ED visits) and aggregate exposure data up, or switch primary outcome to keep the cleaner county-FIPS pipeline.
- **EPA AQS authentication** echoes the API key back in the response URL — don't paste raw responses into anything public. The local file is in `data/raw/_smoke_tests/` which is gitignored.
- **CDC API user guide PDF** (downloaded to `references/`, gitignored) was essential for understanding the Content Area → Indicator → Measure hierarchy. Worth surfacing this dependency in workshop materials.

---

## Stage 2: Design

**Goal:** finalize methodology, data pipeline, and approach. Produce a methodology document.

### Open decisions

- [ ] Geographic scope: regional (e.g., Southeast US, ~4 states) vs. national
- [ ] **Primary outcome variable + matching geography:** options are (a) CDC Tracking Network asthma ED visits at "20K Minimum Population Area" (annual, age-adjusted, requires aggregating exposure data up from county FIPS to MPA via crosswalk), (b) CDC PLACES asthma prevalence at county FIPS (cleaner pipeline but a stock measure, less responsive to short-term compound exposure events), or (c) CDC WONDER respiratory mortality at county FIPS (long-term signal but rare events with suppression). This decision drives the geographic resolution of the entire pipeline.
- [ ] Time window: confirm 2018-2022 (5 years) is the right period given data coverage and the working from home / pandemic-era caveats

### Pipeline plan

- [ ] Document the full data join pipeline (which datasets join on what, in what order)
- [ ] Document the spatial join method for NOAA GSOD (nearest-station vs. inverse-distance weighting)
- [ ] Document the feature engineering plan, including compound exposure thresholds

### Compute plan

- [ ] Decide which HPC system the project will use
- [ ] Map each pipeline step to a Tapis or sbatch job
- [ ] Estimate compute and storage requirements

### Hard line check

- [ ] Methodology document complete and saved in repo before any Stage 3 work begins

**Notes:**

_(populated as decisions are made)_

---

## Stage 3: Compute

**Goal:** stage data on HPC, run experiments, train models, produce raw results.

_(populated when Stage 2 is complete)_

**Notes:**

---

## Stage 4: Analysis

**Goal:** interpret results, build visualizations, connect findings to literature.

_(populated when Stage 3 is complete)_

**Notes:**

---

## Stage 5: Publication

**Goal:** write the paper, format for venue, prepare submission.

_(populated when Stage 4 is complete)_

**Notes:**
