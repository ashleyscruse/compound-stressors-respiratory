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

- [ ] EPA AQS API key — register at https://aqs.epa.gov/data/api/signup
- [ ] CDC Tracking Network API token — register at https://ephtracking.cdc.gov/apigateway/api/v1/register
- [ ] Census API key — register at https://api.census.gov/data/key_signup.html

### Smoke tests for keyless sources

- [ ] NOAA GSOD: pull one station for one year from `s3://noaa-global-summary-of-the-day-pds/`; confirm columns match spec
- [ ] CDC PLACES: download county-level CSV from https://data.cdc.gov/api/views/swc5-untb/rows.csv?accessType=DOWNLOAD; verify LocationID is 5-digit FIPS
- [ ] CDC WONDER: run one test query at https://wonder.cdc.gov for respiratory mortality (J00-J99), one state, one year; save the CSV
- [ ] EPA EJScreen: download a sample from https://gaftp.epa.gov/EJScreen/ (or Zenodo backup at https://zenodo.org/records/14767363); confirm block group FIPS is present

### Smoke tests for keyed sources (run after keys arrive)

- [ ] EPA AQS API: one daily call for PM2.5, one state, one date; verify response shape
- [ ] CDC Tracking Network: one query for asthma ED visits, one state; verify response and check whether county-level data is exposed
- [ ] Census ACS API: one call for population and poverty rate, one state; verify FIPS structure

### Cross-source verification

- [ ] Confirm county FIPS is consistently 5-digit string (or convertible) across every source pulled
- [ ] Confirm column names in each pulled file match what's documented in `internal-spec.md`
- [ ] Update `internal-spec.md` if any source has changed format since the spec was written

### Hard line check

- [ ] All four Stage 1 deliverables confirmed: gap, data viability, target venue, research question

**Notes:**

_(record what worked, what failed, what surprised you as you check items off)_

---

## Stage 2: Design

**Goal:** finalize methodology, data pipeline, and approach. Produce a methodology document.

### Open decisions

- [ ] Geographic scope: regional (e.g., Southeast US, ~4 states) vs. national
- [ ] Primary outcome variable: respiratory mortality (CDC WONDER) vs. asthma prevalence (CDC PLACES) vs. asthma ED visits (CDC Tracking Network, if API access granted)
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
