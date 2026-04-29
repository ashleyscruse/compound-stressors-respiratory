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
- [ ] Census API key — register at https://api.census.gov/data/key_signup.html

### Smoke tests for keyless sources

- [ ] NOAA GSOD: pull one station for one year from `s3://noaa-global-summary-of-the-day-pds/`; confirm columns match spec
- [ ] CDC PLACES: download county-level CSV from https://data.cdc.gov/api/views/swc5-untb/rows.csv?accessType=DOWNLOAD; verify LocationID is 5-digit FIPS
- [ ] CDC WONDER: run one test query at https://wonder.cdc.gov for respiratory mortality (J00-J99), one state, one year; save the CSV
- [ ] EPA EJScreen: download a sample from https://gaftp.epa.gov/EJScreen/ (or Zenodo backup at https://zenodo.org/records/14767363); confirm block group FIPS is present

### Smoke tests for keyed sources (run after keys arrive)

- [ ] EPA AQS API: one daily call for PM2.5, one state, one date; verify response shape
- [x] CDC Tracking Network: one query for asthma ED visits, one state; verify response and check whether county-level data is exposed. **Done 2026-04-29.** Findings: API reachable without token; asthma ED visits available (indicator 90, measure 902); **only at "20K Minimum Population Area" geography, not standard county FIPS** (CDC privacy aggregation). Decision to make in Stage 2: keep ED visits as primary outcome and aggregate exposure data to 20K MPA, or switch primary outcome to CDC PLACES asthma prevalence (county FIPS).
- [ ] Census ACS API: one call for population and poverty rate, one state; verify FIPS structure

### Cross-source verification

- [ ] Confirm county FIPS is consistently 5-digit string (or convertible) across every source pulled
- [ ] Confirm column names in each pulled file match what's documented in `internal-spec.md`
- [ ] Update `internal-spec.md` if any source has changed format since the spec was written

### Hard line check

- [ ] All four Stage 1 deliverables confirmed: gap, data viability, target venue, research question

**Notes:**

**2026-04-29:**
- EPA AQS registration: signup endpoint requires email as URL query parameter, not a form field. Spec doc (`?email=YOUR_EMAIL`) was correct format but I almost missed it. Updated process-log to be explicit.
- CDC Tracking Network registration: the URL listed in the spec (`https://ephtracking.cdc.gov/apigateway/api/v1/register`) is actually an API endpoint, not a registration form. Hitting it returned a 400. Real path: token is OPTIONAL (most endpoints work without it), and to request one you email `nephtrackingsupport@cdc.gov`. Spec needs updating.
- CDC Tracking Network smoke test: API reachable, asthma ED visits exposed, but only at "20K Minimum Population Area" — not standard county FIPS. CDC privacy aggregation. This is a Stage 2 decision point captured in Stage 2 Open Decisions.
- Found CDC API user guide PDF (downloaded to `references/`, gitignored). Indispensable for understanding the Content Area → Indicator → Measure hierarchy.

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
