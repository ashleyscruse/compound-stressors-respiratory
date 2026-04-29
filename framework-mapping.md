# Framework Mapping

This project serves as a real-world validation of the [AI-Accelerated Research Framework](https://github.com/ashleyscruse/nairr-research) developed for the NAIRR Accelerator workshop curriculum at Morehouse College. As work progresses, observations on framework strengths, friction points, and refinements are recorded here to feed back into curriculum materials and case study presentations.

The project is owned by Ashley Scruse and produces a publishable paper. The framework mapping is a parallel artifact, not a project deliverable.

## The Five Stages

| Stage | Focus | Deliverable |
|---|---|---|
| 1. Ideation | Define question, validate against literature, identify data, select target venues | One-page research brief |
| 2. Design | Plan methodology, define data pipeline, select approach | Methodology document |
| 3. Compute | Stage data on HPC, run experiments, train models | Raw results + reproducible pipeline |
| 4. Analysis | Interpret results, build visualizations, connect to literature | Annotated results + limitations |
| 5. Publication | Write paper, format for venue, prepare submission | Submission-ready paper |

## Project Mapping

| Stage | Status | Artifact in this Repo | Notes |
|---|---|---|---|
| 1. Ideation | **Closed 2026-04-29** | `README.md`, `internal-spec.md`, `process-log.md` Stage 1 section | All 7 data sources smoke-tested; FIPS consistency verified; hard line passed |
| 2. Design | In progress | `internal-spec.md` (data joining, feature engineering, models) | Geographic scope and primary outcome variable open; 20K MPA vs county-FIPS decision is the pivotal one |
| 3. Compute | Not started | `scripts/`, `notebooks/` (empty) | Awaiting Stage 2 design decisions |
| 4. Analysis | Not started | `results/` (empty) | |
| 5. Publication | Not started | (TBD) | Target venues identified in `internal-spec.md` |

## Hard Lines

The framework specifies two stop-and-verify points. They apply here as written:

| Boundary | What to verify before crossing |
|---|---|
| Stage 1 → 2 | Confirmed gap, viable data sources, target venue, one-sentence research question |
| Stage 3 → 4 | Pipeline ran correctly, joins are clean, models trained without errors |

## Observations

Recorded as they emerge. Each entry: stage, observation, recommendation for the framework or curriculum.

### Stage 1 (Ideation)

**Observation 1.1 — "Viable data" must mean smoke-tested, not documented.** Of seven data sources documented in the spec, four had inaccuracies that only surfaced during smoke testing (CDC Tracking Network registration mechanism, PLACES temporal coverage, EPA EJScreen primary URL, CDC Tracking Network geography). None blocked the project, but all would have wasted Stage 3 time if discovered then.
**Recommendation for the framework:** strengthen the Stage 1 hard line language. "Confirmed viable data" should explicitly require successful smoke tests for each declared source, not just a written list of URLs and join keys.

**Observation 1.2 — Smoke-test cost is low compared to the cost of skipping it.** All seven sources verified in under two hours. Workshop participants should be told that skipping smoke tests to "save time" routinely costs days at Stage 3.
**Recommendation for the framework:** add a templated smoke-test checklist to the Ideation curriculum materials, mirroring the structure used in this project's `process-log.md` Stage 1 section.

**Observation 1.3 — "Process log" works as a case-study artifact.** Capturing decisions, friction points, and smoke-test outputs in real time produced a usable case-study record without retroactive reconstruction.
**Recommendation for the framework:** add `process-log.md` (or equivalent) to the Stage 1 deliverables list, alongside the existing one-page research brief.

**Observation 1.4 — Smoke tests must check geographic + temporal coverage, not just API surface.** Discovered at Stage 2 entry: the Stage 1 smoke test for CDC Tracking Network confirmed API access and asthma ED visit measures exist, but did not verify which states publish data. The actual coverage is only 8 grantee states, not national. This is a structural data limitation that drove a major design decision (two-pipeline approach instead of single national ED-visit study).
**Recommendation for the framework:** Stage 1 smoke test checklist should explicitly include "Verify the geographic and temporal scope of available data for each measure, not just that the API responds." A two-line query against the geographicItems endpoint would have surfaced this constraint at Stage 1 instead of Stage 2.

### Stage 2 (Design)

_(none yet)_

### Stage 3 (Compute)

_(none yet)_

### Stage 4 (Analysis)

_(none yet)_

### Stage 5 (Publication)

_(none yet)_

## Use as Case Study

Workshop participants will be pointed to the public repository ([github.com/ashleyscruse/compound-stressors-respiratory](https://github.com/ashleyscruse/compound-stressors-respiratory)) as a worked example of the framework applied end to end. Curriculum materials may reference specific commits, files, or observations recorded above. The project itself remains independent of the workshop schedule.
