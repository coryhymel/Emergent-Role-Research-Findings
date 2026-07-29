# Emergent Role Classification from Skill Mapping

**Andela Research: findings summary**
*Run generated 16 June 2026 (deterministic, Louvain seed 42). This summary regenerated 29 July 2026, transcribed programmatically from the report's embedded source data (`CANDS`, `VAL`). It supersedes the 5 June version, which described an earlier run.*

> This Markdown file is a standalone distillation of `index.html` so the findings can be referenced without loading the ~360 KB report (most of which is the embedded bleed-graph visualization data). The interactive bleed-graph node/edge data lives only in `index.html`.

---

## The question / thesis

The central claim: **job titles lag the work.** A company hires a "data scientist" and then asks them to ship LLM agents; it posts for a "security engineer" who turns out to live inside the deployment pipeline. The skills move first, and the title catches up years later. Andela's interest is in identifying these shifts early. This study is a lens into catching that movement while it is happening.

This is how the **ML Engineer** formed in the gap between software engineering and statistics and how **DevSecOps** formed between development, security, and operations. A role emerges in the seam between two older ones as their skill sets *bleed* together.

**Wider thesis (talent debt):** skill half-lives are shrinking from roughly a decade to a couple of years or less. Half-life cannot be measured on a five-week window (so it is not a focus here), but which skill bundles are fusing into new roles *can* be seen, and that is the leading edge of the same phenomenon.

## Definition

> **An emergent role is a stabilized pattern of skill "bleed" across historically distinct roles.** More formally, a recurring bundle of skills whose canonical homes belong to different jobs.

### Detecting emergence without a clock

The corpus is a single ~five-week snapshot (30 March to 2 May 2026) of job postings from current, publicly facing Fortune 500 job boards. "Historical" is therefore reframed as **structure rather than date**: every established role requires a certain set of skills (drawn from Andela's internal skill taxonomy). The method measures how far postings have drifted from that baseline, and dates the technologies involved. **Canonical drift plus technology vintage stand in for the missing time axis.**

---

## Key findings

1. **The strongest signal is a role still settling into its name.** The **MLOps Pipeline Engineer** (automated infrastructure to deploy, version, and monitor ML models in production) scores highest on emergence (0.244; median vintage 2019). It bridges ML Engineering, DevOps, and Data Engineering. Its bundle appears in **1,582 postings** (postings whose skills contain the bundle, not postings titled for it), and validation matches it to **MLOps Engineer**, the youngest of the known emergent roles: the detector's top candidate is a role caught while its title is still consolidating, not a brand-new invention.

2. **The AI frontier is narrower than the hype.** Exactly one AI bundle ranks among the most-emergent candidates: the **LLM Application Engineer** (#3 by recency, 6,758 postings). Once every bundle's technologies are dated, no candidate's stack has a median birth year later than **2019**. Fortune 500 hiring trails the technology frontier by several years, and post-2023 tools (agent frameworks and the like) have not yet stabilized into any recurring cross-role bundle in this corpus.

3. **Employers name the old role, hire the new one.** Among postings still titled for a familiar role like "AI Engineer" or "ML Engineer," **53%** (972 of 1,832 pure-title postings for the LLM Application Engineer bundle) already demand the newer skill set: large language models and conversational AI wired through the OpenAI API and Vertex AI, evaluated with structured harnesses like OpenAI Evals.


---

## Method

Seven steps turn a pile of job postings into a ranked list of forming roles. The algorithm isolates each bundle; a language model only *names* it afterward, and never decides what counts as emergent.

1. **Baselines.** Build each role's "before" two independent ways: taxonomy skill centroids and the profiles of title-pure postings.
2. **Home roles.** Assign every skill the established role it historically belongs to, keeping only confident placements.
3. **Bleed metric.** Keep skill pairs that co-occur far more than chance, span different home roles, and are not synonyms.
4. **Communities.** Cluster the bleed graph alone, so every community bridges two or more historical roles by construction.
5. **Vintage.** Date each bundle's technologies; recent stacks rank as live emergence, old stacks as settled hybrids.
6. **Title divergence.** Measure how often a posting's title names one role while its skills show the cross-boundary bundle.
7. **Label.** A local language model names each isolated bundle from its skills and an example posting.

### Why a snapshot can detect emergence
An earlier attempt recombined clusters drawn from the same co-occurrence graph and simply kept re-finding the obvious. This method breaks that loop by anchoring the "before" to a role taxonomy and to title-pure postings that are **independent of the Fortune 500 co-occurrence patterns being searched**. Bleed is then the measurable gap between what a role was supposed to require and what employers are actually asking for.

*Prior art note: hybrid-job research already exists (Lightcast, O\*NET, ESCO). The contribution here is not the existence of hybrid roles but a snapshot-compatible, taxonomy-anchored, vintage-scored detector.*

### Detection pipeline & corpus facts
- **47,101** Fortune 500 SDLC software postings read.
- **2,026** skills extracted, placed, and scored.
- Two independent baselines: **43** software-role taxonomy centroids + title-pure posting profiles (single-role titles).
- Bleed metric = `lift × cross-home × distance` → bleed graph of **4,093 cross-home edges** → Louvain communities (resolution 4.0).
- Output: **23** emergent-role candidates; recency score from technology vintage; title divergence and LLM label attached per candidate.

### Technical stack (all local, no external API calls)
| Layer | Component | Role |
|---|---|---|
| Storage | PostgreSQL 17 | Fortune 500 SDLC postings + extracted skills |
| Inference | qwen3-embedding:8b (4096-d) | Embeddings for skill/role matching (acceptance threshold 0.55) |
| Inference | gemma4 | Names each detected bundle: labeling only, never detection |
| Index | FAISS | Vector index over role/skill taxonomy |
| Pipeline | Python | Baselines → home role per skill → bleed metric → bleed graph → Louvain communities → vintage score → label |
| Config/data | `pure_title_roles.yaml`, `tech_vintage.json`, skill blocklist, canonical skill map | side inputs |

---

## Full ranked catalog (23 candidates)

Sorted by recency rank (the ordering used for the featured cards; the page's catalog table defaults to sorting by bleed). Recency bands: **Emerging** = median vintage ≥ 2018 · **Recent** ≥ 2010 · **Settled** < 2010 · **Not scored** = no usable vintage. *Dated* = how many of the bundle's top skills carry a datable vintage; a median resting on one or two dated skills is a weak estimate. Canon: *in canon* = widely recognized in occupation dictionaries; *in the wild* = coherent in Fortune 500 hiring but not yet named there. Coverage counts postings containing the bundle (bundles overlap, so coverage does not sum to the corpus). *Div* = share of pure-title postings already showing the bundle.

| # | Role | Emergence | Band | Median vintage | Dated | Coverage | Skills | Homes | Canon | Div |
|--:|---|--:|---|--:|--:|--:|--:|--:|---|--:|
| 1 | MLOps Pipeline Engineer | 0.244 | Emerging | 2019 | 5/10 | 1,582 | 13 | 5 | in canon | 15% |
| 2 | GitOps Platform Security Engineer | 0.200 | Recent | 2017 | 1/9 | 2,608 | 9 | 4 | in canon | 54% |
| 3 | LLM Application Engineer | 0.178 | Recent | 2016 | 5/10 | 6,758 | 29 | 7 | in the wild | 53% |
| 4 | FinOps Reliability Engineer | 0.156 | Recent | 2015 | 4/10 | 1,839 | 11 | 5 | in the wild | 35% |
| 5 | Governed BI Engineer | 0.156 | Recent | 2015 | 1/10 | 1,314 | 14 | 5 | in the wild | 19% |
| 6 | Docs-as-Code Engineer | 0.156 | Recent | 2015 | 3/6 | 311 | 6 | 3 | in the wild | 8% |
| 7 | GitOps Delivery Security Engineer | 0.133 | Recent | 2014 | 3/5 | 3,218 | 5 | 4 | in canon | 36% |
| 8 | Product Front-End Engineer | 0.111 | Recent | 2013 | 1/10 | 4,689 | 30 | 6 | in canon | 51% |
| 9 | Lakehouse Analytics Engineer | 0.111 | Recent | 2013 | 6/10 | 3,603 | 24 | 4 | in canon | 72% |
| 10 | AWS Data Platform Engineer | 0.111 | Recent | 2013 | 10/10 | 2,856 | 10 | 7 | in the wild | 13% |
| 11 | Zero Trust Cloud Engineer | 0.044 | Recent | 2010 | 6/10 | 3,995 | 13 | 5 | in canon | 57% |
| 12 | Technical Program Delivery Lead | 0.000 | Settled | 1947 | 2/10 | 13,081 | 35 | 11 | in the wild | 50% |
| 13 | Test Automation SDET | 0.000 | Settled | 2003 | 1/10 | 6,700 | 42 | 10 | in the wild | 26% |
| 14 | DevSecOps Security Engineer | 0.000 | Settled | 2007 | 2/10 | 5,890 | 20 | 5 | in canon | 52% |
| 15 | Cloud Network Security Engineer | 0.000 | Settled | 1997 | 2/10 | 4,169 | 32 | 4 | in canon | 40% |
| 16 | Polyglot Backend Integration Engineer | 0.000 | Settled | 2001 | 2/10 | 3,753 | 22 | 7 | in the wild | 17% |
| 17 | Database Reliability Engineer | 0.000 | Settled | 1998 | 4/10 | 3,078 | 16 | 7 | in the wild | 16% |
| 18 | Real-Time Embedded Reliability Engineer | 0.000 | Settled | 1973 | 1/10 | 2,585 | 29 | 4 | in the wild | 10% |
| 19 | Edge ML Embedded Engineer | 0.000 | Settled | 1984 | 2/10 | 2,400 | 12 | 6 | in the wild | 6% |
| 20 | SecOps Observability Engineer | 0.000 | Settled | 2005 | 1/9 | 1,822 | 9 | 5 | in the wild | 12% |
| 21 | Enterprise Integration Architect | 0.000 | Settled | 2006 | 1/10 | 1,485 | 12 | 7 | in the wild | 14% |
| 22 | Android XR Developer (flagged artifact) | 0.000 | Settled | 2008 | 5/6 | 248 | 6 | 2 | in the wild | 11% |
| 23 | Product Experience Designer | n/a | Not scored | n/a | 2/10 | 824 | 15 | 2 | in the wild | 90% |

---

## Featured roles, up close (top 6 by recency)

**#1 MLOps Pipeline Engineer.** Builds and runs the automated infrastructure that takes machine-learning models to production: orchestrated pipelines, CI/CD for ML, model registries and versioning, and ML observability, with serving on Vertex AI. Bridges 5 home roles; median vintage 2019 (5/10 skills dated); 1,582 postings; 618 of 4,223 pure-title postings (15%) already show the bundle.

**#2 GitOps Platform Security Engineer.** Builds secure, highly-available cloud infrastructure through code: GitOps delivery, containerization and PaaS, secret management (e.g. Bridges 4 home roles; median vintage 2017 (1/9 skills dated); 2,608 postings; 368 of 678 pure-title postings (54%) already show the bundle.

**#3 LLM Application Engineer.** The contemporary AI engineer building on foundation models rather than training them from scratch: LLM application and conversational systems wired through the OpenAI API and Vertex AI, and evaluated with structured evals (OpenAI Evals, LM Evaluation Harness). Bridges 7 home roles; median vintage 2016 (5/10 skills dated); 6,758 postings; 972 of 1,832 pure-title postings (53%) already show the bundle.

**#4 FinOps Reliability Engineer.** Runs cloud infrastructure for both reliability and cost: observability and infrastructure monitoring (e.g. Bridges 5 home roles; median vintage 2015 (4/10 skills dated); 1,839 postings; 415 of 1,172 pure-title postings (35%) already show the bundle.

**#5 Governed BI Engineer.** Builds governed reporting on the Microsoft BI stack: Power BI, DAX and Power Query, SQL Server Reporting Services and tabular models, paired with data cataloging, privacy management and scheduled automation. Bridges 5 home roles; median vintage 2015 (1/10 skills dated); 1,314 postings; 408 of 2,135 pure-title postings (19%) already show the bundle.

**#6 Docs-as-Code Engineer.** Treats documentation like software: docs-as-code workflows in version control, automated publishing via GitHub Actions, and content auditing. Bridges 3 home roles; median vintage 2015 (3/6 skills dated); 311 postings; 98 of 1,183 pure-title postings (8%) already show the bundle.

---

## Validation

Tests were specified before the results were read, with two post-hoc refinements made and disclosed: the recall signatures and the clustering resolution (final values in the reproducibility note at the end of this document).

### 1. Recall of known emergent roles: 5/7 (71%)

| Known emergent role | Result | Matched skills |
|---|---|---|
| ML Engineer | ✗ not recovered | n/a |
| MLOps Engineer | ✓ recovered as #1 MLOps Pipeline Engineer | CI/CD for Machine Learning, ML Observability, MLOps, Model Deployment |
| DevSecOps | ✓ recovered as #14 DevSecOps Security Engineer | DevSecOps, Vulnerability Management |
| Platform Engineer | ✓ recovered as #4 FinOps Reliability Engineer | Infrastructure as Code, Observability |
| Analytics Engineer | ✓ recovered as #9 Lakehouse Analytics Engineer | Data Lakehouse, Data Modeling, Data Pipelines |
| AI Engineer | ✓ recovered as #3 LLM Application Engineer | Large Language Models (LLMs), OpenAI API |
| AI Agent Developer | ✗ not recovered | n/a |

### 2. Null model (home-label permutation, n=20)
Observed bleed graph has **4,093** cross-home edges vs null **4,986 ± 18** (z = **-50.41**). Real home labels remove ~18% more co-occurrence as within-role than random assignment, confirming the role taxonomy captures genuine structure. Community count (23 vs 21.7) is not sensitive to home shuffling and is not the headline statistic.

### 3. Synonym-leakage audit
200-edge sample: mean cosine 0.701, max 0.78 (cutoff 0.78); **8%** of sampled edges flagged as possible near-synonyms.

### 4. Home-assignment confidence (dual-baseline agreement)
Of the 2,026 scored skills: **596** were placed identically by both baselines, **432** agreed within the top three, **393** rest on the taxonomy alone (no title-pure evidence either way), and **605** low-confidence placements were excluded from the bleed graph entirely.

### 5. Manual precision audit (this run, 29 July 2026)
A single-rater manual audit of all **23 candidates** judged 8 GENUINE (emerging), 14 HYBRID (real but settled), and **1 ARTIFACT**: Android XR Developer, whose bundle contains no XR-specific skills and whose cross-home edges come from the taxonomy splitting Android tooling across the Mobile and VR/AR home roles. **Non-artifact rate: 22/23 = 96%.** The artifact is kept visible and flagged in the catalog. The full per-candidate audit table is maintained alongside the detection pipeline.

---

## Honest disclosure: what this study does not claim

- **It is a snapshot, not a trend.** Emergence is inferred from canonical drift and technology vintage, never from posting-date growth. The corpus spans about five weeks.
- **It reflects US Fortune 500 hiring.** Findings generalize to that population, not the whole labor market.
- **Skill extraction is imperfect.** An importance floor, a canonical-name map, and a blocklist reduce noise but do not eliminate it; residual false positives (e.g. design tools inside the LLM bundle) stay visible in the skill lists rather than being silently removed. A single-rater manual audit of all 23 candidates (July 2026) flagged one as an artifact (Android XR Developer); it is kept visible and flagged rather than hidden.
- **Technology dating is fuzzy.** Birth years are judgment calls from a versioned, auditable lookup; the emergence score is a rank, not a precise date. Several bundles' medians rest on only one or two dated skills (see *Dated* column).
- **Ubiquitous skills carry no signal.** A skill that co-occurs with nearly everything produces no bleed edge by design.
- **Hybrid jobs are prior art.** The contribution is the detection method, not the phenomenon.

---

*Reproducibility: corpus collected 30 March to 2 May 2026 from public Fortune 500 job boards; 47,101 SDLC postings; 2,026 skills scored; 4,093-edge cross-role bleed graph; embeddings qwen3-embedding:8b at acceptance threshold 0.55; bleed edges require co-occurrence support ≥ 60 and lift ≥ 1.3 with a synonym cosine cutoff of 0.78; Louvain resolution 4.0, seed 42 (deterministic); communities kept at ≥ 5 skills, ≥ 5 internal edges, ≥ 120 postings coverage, hub share ≤ 0.55; null model 20 home-label permutations; recency band = technologies born 2023 or later, the three years preceding the run; run generated 16 June 2026. The interactive report subsamples the graph for display (200 skills, 2713 edges).*
