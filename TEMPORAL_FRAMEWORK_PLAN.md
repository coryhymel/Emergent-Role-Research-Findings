# Temporal Leading-Indicator Framework — Execution Plan

**For:** Claude Code, working in the emergent-role detector repo.
**Goal:** Extend the existing snapshot detector (42,621 F500 postings, 27 candidate roles, vintage-scored) into a temporal framework that emits a quantified lead-time forecast for role settlement.
**Headline metric we are trying to produce:** "Median X months of lead time on roles that ultimately settle, with Y% recall, at Z% precision."

---

## Project context (do not skip — read first)

The existing detector is documented in `Emergent-Role-Research-Findings/index.html` (this folder). The substantive moving parts you should expect to find in the codebase:

- **Storage:** PostgreSQL 17. Postings table, extracted skills table, role taxonomy.
- **Models:** `mxbai-embed-large` (1024-d embeddings), `qwen2.5:7b-instruct` (labeling only), FAISS index over skill/role taxonomy.
- **Pipeline:** Python. Steps are: baselines → home-role-per-skill → bleed metric (`lift × cross-home × distance`) → bleed graph → Louvain communities (seed 42) → vintage score → LLM label.
- **Side inputs:** `pure_title_roles.yaml`, `tech_vintage.json`, skill blocklist, canonical skill map.
- **Output:** `emergent_v3_bleed.json` / `.md` → dashboard.

If any of the above doesn't match what's actually in the repo, **stop and report** before changing anything. The whole plan assumes that pipeline as the starting point.

---

## Hard constraints

1. **Keep inference local.** No external API calls for embeddings, labeling, or analysis. Same constraint as v3.
2. **Determinism.** Louvain seed stays at 42. All new randomness (matching, bootstrap, model fits) takes a seed parameter, default `42`.
3. **Reproducibility.** Every phase produces an artifact in `artifacts/<phase>/<window_id>/` so runs can be compared.
4. **Backwards compatibility.** The existing snapshot pipeline keeps working unchanged. Temporal pipeline is additive, not a rewrite.
5. **One workstation.** Memory and compute budgets must fit a single dev box. No distributed jobs.

---

## Phase 0 — Repo audit and baseline run (½ day)

**Goal:** Confirm the pipeline runs end-to-end as documented and produce a tagged baseline so later windows have a reference.

Tasks:
- Run the existing v3 pipeline against the current corpus. Capture the output to `artifacts/baseline_v3/`.
- Generate a report (`audit.md`) listing: actual file paths for each stage, actual column names in Postgres tables, model versions, library versions, runtime per stage.
- Diff the audit against the assumptions in this plan's "Project context" section. **If any drift, post the diff and pause.**

Acceptance: `artifacts/baseline_v3/emergent_v3_bleed.json` exists and matches the published 27-candidate set within ±1 candidate (Louvain non-determinism in fringe communities is acceptable).

---

## Phase 1 — Forward panel infrastructure (3–5 days)

**Goal:** Start capturing posting data on a fixed cadence so the time axis begins now.

### 1.1 Schema changes
Add to the postings table:
- `posted_date` (date, nullable — source-provided)
- `captured_date` (date, not null — when our scraper saw it)
- `source` (text, e.g. `greenhouse`, `lever`, `workday`, `direct`)
- `source_posting_id` (text — stable ID from the source)
- `expires_date` (date, nullable)
- `capture_run_id` (uuid, FK to a new `capture_runs` table)

Create `capture_runs(id, started_at, finished_at, source, n_postings, notes)`.

**Critical:** do *not* dedupe across capture runs. Each weekly capture is its own corpus. A posting that appears in 6 consecutive weeks should produce 6 rows linked by `(source, source_posting_id)`, not 1 row.

### 1.2 Scraper cadence
- Pick a weekly anchor (Sunday 02:00 local). Cron or systemd timer.
- Target the same F500 board set used in the snapshot. Document the list in `config/sources.yaml`.
- On failure, retry up to 3× then alert. Never silently skip a run — a missing window will bias every downstream analysis.

### 1.3 Capture audit dashboard
A small HTML page (`reports/capture_health.html`) showing per-week posting volume by source. This is how we'll catch scraper rot before it contaminates the panel.

Acceptance: 4 consecutive weekly captures have run, each with > 80% of expected source coverage, and the audit page renders.

---

## Phase 2 — Historical backfill (1–3 weeks, depends on path chosen)

**Goal:** Get enough pre-2026 data to backtest the framework against known settlement events (MLOps, AI Engineer, Analytics Engineer, etc.).

Three paths, in order of preference. Pick **one** based on budget. Surface the cost/timeline tradeoff to Cory before committing.

### Path A — License (fastest, ~$30–80k)
- Lightcast or Revelio Labs historical job posting feed, F500 filter, 2018-01 → present.
- Map their schema to ours via `etl/lightcast_to_panel.py`.

### Path B — Targeted reconstruction (cheapest, slowest)
- CommonCrawl + Wayback Machine over `boards.greenhouse.io/*`, `jobs.lever.co/*`, `*.myworkdayjobs.com/*`.
- Filter to F500 employer subdomains using a manually maintained `config/f500_subdomains.yaml`.
- 20–30% per-quarter coverage is acceptable; document the coverage curve.

### Path C — Partner data
- BLS OEWS, O*NET, ESCO for *labels and definitions only* (they don't have posting-level data).
- Use to validate settlement events, not to drive detection.

Schema: same as Phase 1.1. All historical rows get `capture_run_id` for a synthetic run per source-quarter, and `captured_date` set to ingest time.

Acceptance: ≥ 8 quarters of historical data with documented coverage per quarter, loaded into the same postings table as the forward panel.

---

## Phase 3 — Windowed detector (3–5 days)

**Goal:** Run the existing detector on rolling windows instead of one snapshot.

### 3.1 Window definition
- Default: quarterly windows with a 12-month look-back. (`Q1-2026` window = postings with `posted_date` in the 12 months ending 2026-03-31.)
- Parameterize: `--window-end`, `--lookback-months`, `--step-months`.

### 3.2 Freeze the taxonomy
- Pin the current taxonomy as `taxonomy/v1.frozen.json`. **The longitudinal series uses only this version.**
- Any taxonomy edits go into `v2`, `v3`, etc., and run as a separate parallel series. This is the single most important methodological discipline in the whole plan — if the baseline drifts mid-study, all bleed signal is confounded.

### 3.3 Drop vintage as the primary emergence proxy
- Keep `vintage_score` as a *feature*, but the primary emergence signal for the temporal series is **bleed strength + posting-volume growth** between consecutive windows.
- Vintage now provides robustness check, not the main rank.

### 3.4 Per-window outputs
- `artifacts/windows/<window_id>/bleed_graph.parquet`
- `artifacts/windows/<window_id>/candidates.json` (same schema as `emergent_v3_bleed.json`)
- `artifacts/windows/<window_id>/title_divergence.json` (per-candidate)
- `artifacts/windows/<window_id>/run_metadata.json` (taxonomy version, code git sha, runtime)

Acceptance: 4 quarters of windows produced without manual intervention; per-window candidate count is stable to within ±15%.

---

## Phase 4 — Bundle alignment across windows (3–5 days, **hardest engineering**)

**Goal:** Give every emergent bundle a stable identity across time so you can track its trajectory.

### 4.1 The matching problem
Louvain in `Q1` and Louvain in `Q2` produce unordered community sets. Without alignment, "candidate #7" means nothing across windows.

### 4.2 Algorithm
- Between consecutive windows compute a similarity matrix `S[i,j] = Jaccard(skills_in_Qn_bundle_i, skills_in_Qn+1_bundle_j)`.
- Solve the assignment with `scipy.optimize.linear_sum_assignment` on `-S` (Hungarian / Kuhn-Munkres).
- Accept matches with `S >= 0.30`. Unmatched Qn+1 bundles get **new** stable IDs; unmatched Qn bundles are marked **retired**.
- For multi-window stability, prefer transitive matching: a bundle's ID propagates as long as Jaccard to the previous instance stays above threshold.

### 4.3 Output
- `artifacts/bundles/registry.parquet` — one row per `(bundle_stable_id, window_id)` with metrics: emergence score, coverage, title divergence, bleed-edge count, dominant titles, top skills.
- `artifacts/bundles/trajectories.json` — per-bundle time series, ready for plotting.

### 4.4 Validation
- Manually inspect 10 trajectories. Names should be plausible across time (you should not see "MLOps" in Q1 mapped to "Frontend Designer" in Q2).
- Compute and log the trajectory-length distribution. A flat-looking distribution (almost no bundle survives 2+ windows) means the threshold is too tight or the windows are too narrow.

Acceptance: ≥ 50% of bundles in any window match to a bundle in the previous window; the 7 known emergent roles all show ≥ 4-quarter continuous trajectories on backfilled data.

---

## Phase 5 — Settlement event definition (2 days, mostly design + tuning)

**Goal:** Define the dependent variable. Without a precise "settlement" definition, none of the downstream forecasting makes sense.

### 5.1 Working definition (tune against ground truth)
A bundle is **settled** in window `Qn` if all of the following hold:
1. Title divergence < 15% (titles match the bundle's hiring intent).
2. A single dominant title accounts for > 60% of postings carrying the bundle.
3. Conditions (1) and (2) hold for 2 consecutive windows.

### 5.2 Ground truth set
- Use the 7 known recoveries (DevSecOps, MLOps, SRE, Analytics Engineer, Data Engineer, Platform Engineer, ML Engineer — confirm exact list against the current `recall_list` in the dashboard).
- For each, identify the approximate settlement window from historical data + literature.

### 5.3 Threshold tuning
- Grid-search thresholds on the 7 known roles. Choose the combination that produces settlement events within ±2 quarters of the known-true settlement.
- Document the chosen thresholds in `config/settlement.yaml`. Lock them.

Acceptance: thresholds yield correct settlement detection for ≥ 6 of 7 known roles within ±2 quarters.

---

## Phase 6 — Backtest harness (3–5 days)

**Goal:** Quantify the lead time. This is the headline number for the framework paper.

### 6.1 Harness
- For each backfill quarter `Qb` from `2019-Q1` onward:
  - Run the windowed detector with data only up to `Qb`.
  - Record which bundles cross the "candidate" threshold (bleed score + coverage minimum) for the first time.
- For each settlement event `Se` in the ground-truth set:
  - Find the earliest `Qb` at which that bundle was first flagged.
  - Lead time = `Se - Qb` (in quarters).

### 6.2 Outputs
- `reports/backtest/lead_time_distribution.png` — histogram of lead times across known roles.
- `reports/backtest/recall_precision.json` — at each quarterly threshold, what fraction of eventually-settled roles were flagged, and what fraction of flagged roles eventually settled.
- `reports/backtest/false_positives.md` — bundles flagged but never settled. Hand-audit each; many are real candidates that simply haven't settled yet, but some will be artifacts.

### 6.3 Headline numbers
At minimum produce:
- Median lead time on settled roles.
- Recall at 12 / 18 / 24-month lead horizons.
- Precision at the operating threshold (% flagged that ultimately settle within 36 months).

Acceptance: a single-page `reports/backtest/HEADLINE.md` with the three numbers above, defensible methodology section, and links to the underlying artifacts.

---

## Phase 7 — Hazard / survival model (3–5 days)

**Goal:** Move from rank to probability. "This candidate has a 70% probability of settling within 18 months" is what makes this a leading *indicator*.

### 7.1 Setup
- Use `lifelines` (Python). Cox proportional hazards is the default; consider AFT models for sensitivity.
- Event = settlement (Phase 5 definition).
- Subject = bundle stable ID (Phase 4).
- Observation window per subject = quarters from first flag to settlement (event) or to end-of-data (right-censored).

### 7.2 Covariates
- Bleed strength (per-window mean).
- Posting-volume growth rate (per-window log-delta).
- Vintage score (median).
- Cross-home count (number of canonical roles the bundle bridges).
- Employer concentration (Herfindahl across hiring employers).
- Top-skill stability across windows (Jaccard of top-20 skills between consecutive windows).

### 7.3 Outputs
- `artifacts/hazard/model.pkl`
- `reports/hazard/coefficients.md` — which covariates predict faster settlement, with CIs.
- `reports/hazard/active_forecasts.json` — for every currently-active candidate, P(settle within 12 / 18 / 24 months).

### 7.4 Validation
- Time-based cross-validation: train on subjects whose events occurred before `Qx`, test on subjects whose events occurred after. Report concordance index per fold.

Acceptance: out-of-sample concordance ≥ 0.65. Below that, the framework is not yet a useful indicator and we should iterate on features before publishing.

---

## Phase 8 — Index publication (2–3 days)

**Goal:** Make this a recurring, useful artifact rather than a paper.

### 8.1 Quarterly index page
Update `Emergent-Role-Research-Findings/index.html` (or a sibling `index_quarterly.html`) with:
- Active candidates list with current emergence score, lead-time forecast, and trajectory sparkline.
- Settlement events this quarter (bundles that crossed the threshold).
- Retired candidates (bundles that failed to settle and faded).
- A `data/quarterly_index.json` payload so the page can be updated without code changes.

### 8.2 Automation
- A make target / Justfile recipe: `just publish-quarter Q3-2026` runs Phases 3 → 7 against the freshest data and regenerates the index page.

Acceptance: the page renders against generated data for the current quarter end-to-end, with no manual editing.

---

## Cross-cutting requirements (apply throughout)

- **Versioning:** every artifact carries the git sha that produced it and the taxonomy version in `run_metadata.json`.
- **Testing:** unit tests for the matching algorithm (Phase 4), the settlement detector (Phase 5), and the backtest harness (Phase 6). Property-based tests on the matcher using `hypothesis` are a good idea — bundle alignment is the highest-risk component.
- **Logging:** structured JSON logs to `logs/<phase>/<run_id>.jsonl`. The audit report at Phase 0 should also be regenerated at the end as a "what actually ran" delta.
- **Don't let posting volume contaminate the bleed metric.** Volume feeds the hazard model in Phase 7, not the detector. Keep detection structural; introduce temporal signal only at the settlement-prediction layer. This is the single most important methodological note in the plan.

---

## Suggested execution order and rough size

| Phase | Effort | Blocking | Notes |
|------|--------|----------|-------|
| 0 | ½ day | — | Audit first; do not skip |
| 1 | 3–5 d | 0 | Start capturing immediately, in parallel with everything else |
| 2 | 1–3 wk | 0 | Path decision needs Cory; start Path B opportunistically |
| 3 | 3–5 d | 1 or 2 | Can use partial backfill |
| 4 | 3–5 d | 3 | Hardest; budget extra |
| 5 | 2 d | 3 | Mostly design + tuning |
| 6 | 3–5 d | 2, 4, 5 | Headline-number-producing phase |
| 7 | 3–5 d | 4, 5, 6 | Requires real settlement events to fit against |
| 8 | 2–3 d | 7 | Mostly engineering |

**Minimum viable end-to-end:** Phases 0, 1, 3 (forward-only, no backfill), 4, 5, plus a thin Phase 6 against the synthetic forward panel. ~3 weeks of focused work. Real headline numbers require Phase 2 (backfill).

---

## What success looks like

A single `reports/backtest/HEADLINE.md` that says, defensibly:

> Across N known emergent SDLC roles between 2019 and 2025, the temporal detector flagged each as a candidate a median of K months before its settlement event, with recall R at a 12-month horizon and precision P at the operating threshold. The active-candidate forecasts for the current quarter are at `data/quarterly_index.json`.

Everything else in this plan exists to make that paragraph true.
