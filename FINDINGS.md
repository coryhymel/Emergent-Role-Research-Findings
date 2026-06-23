# Emergent Role Classification from Skill Mapping

**Andela Research — findings summary**
*Generated 5 June 2026. Deterministic (Louvain seed 42).*

> This Markdown file is a faithful, standalone distillation of `index.html` so the findings can be referenced without loading the 428 KB report (most of which is the embedded bleed-graph visualization data). All numbers below are transcribed directly from the report's source data (`CANDS`, `VAL`) and narrative. The interactive bleed-graph node/edge data lives only in `index.html`.

---

## The question / thesis

The central claim: **job titles lag the work.** A company hires a "data scientist" and then asks them to ship LLM agents; it posts for a "security engineer" who turns out to live inside the deployment pipeline. The skills move first, and the title catches up years later. Andela's interest is in identifying these shifts early — this study is a lens into catching that movement while it is happening.

This is how the **ML Engineer** formed in the gap between software engineering and statistics, how **DevSecOps** formed between development, security, and operations, and how the **AI Agent Developer** is forming right now between backend engineering and LLM orchestration. A role emerges in the seam between two older ones as their skill sets *bleed* together.

**Wider thesis — talent debt:** skill half-lives are shrinking from roughly a decade to a couple of years or less. Half-life cannot be measured on a five-week window (so it is not a focus here), but which skill bundles are fusing into new roles *can* be seen, and that is the leading edge of the same phenomenon.

## Definition

> **An emergent role is a stabilized pattern of skill "bleed" across historically distinct roles** — more formally, a recurring bundle of skills whose canonical homes belong to different jobs.

### Detecting emergence without a clock

The corpus is a single ~five-week snapshot of job postings from current, publicly facing Fortune 500 job boards. "Historical" is therefore reframed as **structure rather than date**: every established role requires a certain set of skills (drawn from Andela's internal skill taxonomy). The method measures how far postings have drifted from that baseline, and dates the technologies involved — a bundle built on tools born after 2023 is emergent by construction, while one built on decade-old tools is a hybrid that has already settled. **Canonical drift + technology vintage stand in for the missing time axis.**

---

## Key findings

1. **One role is unmistakably new.** The **LLM Agent Engineer** — who chains LLMs into autonomous agents, gives them memory and tools, and debugs emergent behavior — scores highest on emergence by a wide margin (**0.933**). Its signature stack is almost entirely **post-2023**, and it appears across **2,301 postings** with no settled job title yet.

2. **The live frontier is AI.** Four of the most-emergent candidates are AI roles, spanning agent systems, foundation-model application work, conversational-AI product work, and on-device computer vision. They sit at the recent end of the technology timeline while every other cluster trails years behind.

3. **Employers name the old role, hire the new one.** For the foundation-model role (Generative AI Engineer), **54%** of postings titled for an established home role already list the cross-boundary skill bundle. The title says "AI Engineer" or "data scientist"; the skills say something newer.

4. **The method recovers roles we already know.** As a check, the detector rediscovers **all seven** roles known to have emerged through skill bleed (e.g. DevSecOps, MLOps, Analytics Engineer, SRE) as candidates — without being told to look for them. The same machinery that finds settled roles surfaces the new ones.

---

## Method

Seven steps turn a pile of job postings into a ranked list of forming roles. The algorithm isolates each bundle; a language model only *names* it afterward, and never decides what counts as emergent.

1. **Baselines** — build each role's "before" two independent ways: taxonomy skill centroids and the profiles of title-pure postings.
2. **Home roles** — assign every skill the established role it historically belongs to, keeping only confident placements.
3. **Bleed metric** — keep skill pairs that co-occur far more than chance, span different home roles, and are not synonyms.
4. **Communities** — cluster the bleed graph alone, so every community bridges two or more historical roles by construction.
5. **Vintage** — date each bundle's technologies; recent stacks rank as live emergence, old stacks as settled hybrids.
6. **Title divergence** — measure how often a posting's title names one role while its skills show the cross-boundary bundle.
7. **Label** — a local language model names each isolated bundle from its skills and an example posting.

### Why a snapshot can detect emergence
An earlier attempt recombined clusters drawn from the same co-occurrence graph and simply kept re-finding the obvious. This method breaks that loop by anchoring the "before" to a role taxonomy and to title-pure postings that are **independent of the Fortune 500 co-occurrence patterns being searched**. Bleed is then the measurable gap between what a role was supposed to require and what employers are actually asking for.

### What is distinctive here
- **Cross-sectional bleed.** Emergence is recovered from canonical-vs-observed drift in a single snapshot, not from posting-volume growth over time.
- **Technology-vintage scoring.** Dating the stack separates genuinely emerging bleed from long-settled hybrids — a stand-in for the missing time axis. (Future work: incorporate temporal signals directly, data permitting.)
- **An embedding-distance synonym filter.** Requiring the two bleeding skills to be semantically distant removes the near-synonym noise that defeats volume-based methods.

*Prior art note: hybrid-job research already exists (Lightcast, O\*NET, ESCO). The contribution here is not the existence of hybrid roles but a snapshot-compatible, taxonomy-anchored, vintage-scored detector.*

### Detection pipeline & corpus facts
- **42,621** Fortune 500 SDLC software postings read.
- **2,521** skills extracted, placed, and scored.
- Two independent baselines: **43** software-role taxonomy centroids + title-pure posting profiles (single-role titles).
- Bleed metric = `lift × cross-home × distance` → bleed graph → Louvain communities.
- Bleed graph of **734** skills.
- Output: **26** emergent-role candidates, scored by vintage + title divergence + LLM label.

### Technical stack (all local, no external API calls)
| Layer | Component | Role |
|---|---|---|
| Storage | PostgreSQL 17 | Fortune 500 SDLC postings + extracted skills |
| Inference | mxbai-embed-large (1024-d) | Embeddings for skill/role matching |
| Inference | qwen2.5:7b-instruct | Names each detected bundle — labeling only, never detection |
| Index | FAISS | Vector index over role/skill taxonomy |
| Pipeline | Python | Baselines → home role per skill → bleed metric → bleed graph → Louvain communities → vintage score → label |
| Config/data | `pure_title_roles.yaml`, `tech_vintage.json`, skill blocklist, canonical skill map | side inputs |

---

## Full ranked catalog (26 candidates)

Emergence tiers: **Live** ≥ 0.60 · **Warm** ≥ 0.25 · **Stable** < 0.25 · **n/a** = no datable technologies.
Canon: *in-canon* = a known/established emergent role; *in-the-wild* = surfaced by the detector without a canonical label.

| # | Role | Emergence | Tier | Median vintage | % born ≥2023 | Coverage (postings) | Skills | Home roles | Canon |
|---:|---|---:|---|---:|---:|---:|---:|---:|---|
| 1 | LLM Agent Engineer | 0.933 | Live | 2023 | 100% | 2,301 | 10 | 4 | in-the-wild |
| 2 | Conversational AI Researcher | 0.544 | Warm | 2019 | 50% | 295 | 5 | 3 | in-the-wild |
| 3 | Azure AI Frontend Developer | 0.306 | Warm | 2015 | 25% | 2,870 | 20 | 13 | in-the-wild |
| 4 | Generative AI Engineer | 0.304 | Warm | 2019 | 10% | 5,421 | 33 | 4 | in-the-wild |
| 5 | Quality Engineering SDET | 0.200 | Stable | 2017 | 0% | 6,453 | 23 | 16 | in-the-wild |
| 6 | MLOps Engineer | 0.178 | Stable | 2016 | 0% | 3,838 | 28 | 6 | in-canon |
| 7 | Platform Engineer | 0.156 | Stable | 2015 | 0% | 2,057 | 8 | 4 | in-the-wild |
| 8 | Lakehouse Data Engineer | 0.133 | Stable | 2014 | 0% | 7,341 | 35 | 10 | in-canon |
| 9 | Full-Stack Web Engineer | 0.111 | Stable | 2013 | 0% | 7,198 | 49 | 10 | in-canon |
| 10 | Applied Data Scientist | 0.111 | Stable | 2013 | 0% | 2,669 | 20 | 7 | in-the-wild |
| 11 | Edge AI Mobile Engineer | 0.089 | Stable | 2012 | 0% | 1,419 | 8 | 6 | in-the-wild |
| 12 | Cloud Reliability Engineer | 0.067 | Stable | 2011 | 0% | 7,459 | 31 | 10 | in-the-wild |
| 13 | Enterprise Solutions Architect | 0.067 | Stable | 2011 | 0% | 1,019 | 10 | 5 | in-the-wild |
| 14 | Cloud Infrastructure Engineer | 0.044 | Stable | 2010 | 0% | 14,988 | 47 | 17 | in-the-wild |
| 15 | Product Analytics BI Developer | 0.044 | Stable | 2010 | 0% | 3,720 | 22 | 9 | in-the-wild |
| 16 | Technical Delivery Lead | 0.000 | Stable | 2001 | 0% | 16,699 | 73 | 19 | in-the-wild |
| 17 | QA Test Engineer | 0.000 | Stable | 2003 | 0% | 8,112 | 62 | 10 | in-the-wild |
| 18 | DevSecOps Engineer | 0.000 | Stable | 1998 | 0% | 7,334 | 39 | 8 | in-canon |
| 19 | Embedded Systems Engineer | 0.000 | Stable | 1993 | 0% | 5,522 | 43 | 12 | in-the-wild |
| 20 | Network Security Engineer | 0.000 | Stable | 2000 | 0% | 5,154 | 45 | 6 | in-the-wild |
| 21 | Linux Infrastructure Administrator | 0.000 | Stable | 2000 | 0% | 3,531 | 19 | 7 | in-the-wild |
| 22 | Site Reliability Engineer | 0.000 | Stable | 2003 | 0% | 1,284 | 6 | 4 | in-the-wild |
| 23 | Technical Product Owner | n/a | Not scored | — | — | 2,473 | 11 | 6 | in-the-wild |
| 24 | Product UX Designer | n/a | Not scored | — | — | 2,197 | 34 | 5 | in-the-wild |
| 25 | Database Reliability Engineer | n/a | Not scored | — | — | 1,411 | 17 | 7 | in-the-wild |
| 26 | Signal Processing Engineer | n/a | Not scored | — | — | 411 | 5 | 3 | in-the-wild |

*Roles 23–26 have no emergence score because their skills are craft/practice skills without datable technology vintages.*

---

## Featured roles, up close (top 6)

### #1 — LLM Agent Engineer · emergence 0.933 (Live) · in-the-wild
Builds autonomous LLM-agent systems: chaining models into multi-step workflows, giving them memory and tool access, and debugging the resulting emergent behavior. Bridges the AI/ML engineering world (model orchestration, selection) with classical software architecture (event-driven control flow, the ReAct reasoning-action loop). Its signature stack — Agent Orchestration, memory-based agents, agent debugging — is almost entirely 2023+, making it the most genuinely emergent role in the corpus: a coherent, recurring skill bundle with no settled job title yet.
- **Bridges:** AI Engineer (5), ML Engineer (3), Software Architect (1), Data Scientist (1)
- **Bridging skills:** Agent Orchestration, AI Agent Development, Mode, caret, Prefect, Agent Debugging & Logging, Memory-based Agents, AI Agent Orchestration, shapely, ReAct Architecture
- **Vintage:** median 2023 · 100% born ≥2023 (5/10 dated) · coverage 2,301 postings
- **Title divergence:** 25.1% (699 divergent / 2,785 pure). Example: posting titled *"data scientist (r50031841) remote"* (home role: Data Scientist) already lists Mode, Prefect.

### #2 — Conversational AI Researcher · emergence 0.544 (Warm) · in-the-wild
A small but coherent bleed between conversational-AI engineering and UX/data research: building chatbot/LLM-backed experiences while running user-research interviewing, handling consent and privacy, and sanitizing data. Unusual because it pairs model-facing skills with human-subjects research practice. Genuinely emerging but on thin volume (~300 postings).
- **Bridges:** AI Engineer (2), UX Researcher (2), Data Engineer (1)
- **Bridging skills:** Semi-structured Interviewing, Conversational AI & Chatbots, Consent & Privacy Management, FastChat, Data Validation and Sanitization
- **Vintage:** median 2019 · 50% born ≥2023 (2/5 dated) · coverage 295 postings
- **Title divergence:** 0.7% (13 / 1,755). Example: *"chief data/ai engineer senior"* (AI Engineer) lists Data Validation and Sanitization, FastChat.

### #3 — Azure AI Frontend Developer · emergence 0.306 (Warm) · in-the-wild
Front-end development on the Microsoft/Azure stack with an AI twist: building UIs wired to Azure OpenAI and provisioned through Azure Resource Manager. Bridges front-end engineering with cloud and AI-platform work. Partly emergent (Azure OpenAI is 2023) but mostly a vendor-stack hybrid rather than a distinct new role.
- **Bridges (13 home roles):** Front-End Engineer (4), AI Engineer (3), QA Engineer (2), Full-Stack Engineer (2), + others
- **Bridging skills:** Azure, XML, Markdown, Azure OpenAI, Style Guides, Azure Resource Manager (ARM), XAML, Element UI, Microsoft Project, Lit, .NET, Azure Bot Service, MSTest, JSON
- **Vintage:** median 2015 · 25% born ≥2023 (4/10 dated) · coverage 2,870 postings
- **Title divergence:** 7.1% (301 / 4,257). Example: *"devsecops engineer – identity & access management"* (DevSecOps Engineer) lists Azure, Azure Resource Manager (ARM).

### #4 — Generative AI Engineer · emergence 0.304 (Warm) · in-the-wild
The contemporary AI Engineer who builds applications *on* foundation models — calling services such as OpenAI and Vertex APIs, fine-tuning with LoRA, shipping with the Vercel AI SDK — rather than training models from scratch. Bridges classical ML engineering with API-first generative-AI development. The bundle is recent-vintage (LLMs, Generative AI, Vercel AI SDK all 2022–23), confirming a live, fast-emerging role distinct from both the data scientist and the agent developer.
- **Bridges:** ML Engineer (17), AI Engineer (14), Data Scientist (1), Data Architect (1)
- **Bridging skills:** OpenAI API, Large Language Models, LoRA, Core ML, Generative AI, Vercel AI SDK, MLOps, Google Vertex AI, Stanford CoreNLP, Large Language Models (LLMs), Gensim, MLflow, Uvicorn, Metadata Management
- **Vintage:** median 2019 · 10% born ≥2023 (10/10 dated) · coverage 5,421 postings
- **Title divergence:** **54.0%** (1,497 / 2,773) — the headline finding #3. Example: *"sr manager, data science & ai engineering"* (AI Engineer) lists Altair, Core ML, Large Language Models, LoRA, Metadata Management, OpenAI API.

### #5 — Quality Engineering SDET · emergence 0.200 (Stable) · in-the-wild
Engineering-quality work sitting between software development and QA: code-quality management, linting, debugging, and test tooling alongside core architecture skills. Bridges QA and software engineering — the long-established "quality engineer"/SDET hybrid rather than a new role; its skills are decade-old, so it scores low on vintage.
- **Bridges (16 home roles):** QA Engineer (5), Software Architect (2), Mobile Engineer (2), QA Analyst (2), + others
- **Bridging skills:** Code Quality Management, C++, Debugging, Software Testing, Design Systems, Code Analysis and Linting, Software Architecture, Cypress, Reactor Core, Version Control & GitOps, Software Composition Analysis (SCA), Ruby on Rails, Code Coverage, REST-assured
- **Vintage:** median 2017 · 0% born ≥2023 (3/10 dated) · coverage 6,453 postings
- **Title divergence:** 12.0% (742 / 6,206). Example: *"sr. software qa engineer"* (QA Engineer) lists Code Quality Management, Debugging, Software Testing.

### #6 — MLOps Engineer · emergence 0.178 (Stable) · in-canon
Takes trained models to production: CI/CD for machine learning, model serving and deployment, experiment tracking, plus the modeling core (PyTorch, Scikit-Learn, RLHF). Bridges data science, ML engineering, and DevOps — the canonical bleed that produced MLOps. Moderately recent (RLHF 2022, ML observability ~2021) but built on a now-maturing practice, so it reads as established-but-still-growing rather than brand-new.
- **Bridges:** ML Engineer (12), AI Engineer (8), Data Scientist (4), DevOps Engineer (2), + others
- **Bridging skills:** CI/CD for Machine Learning, spaCy, Model Deployment, Model Serving & Deployment, Scikit-Learn, Machine Learning, PyTorch, RLHF, Experiment Tracking, Model Selection, Supervised Learning, Model Training, Reinforcement Learning, SciPy
- **Vintage:** median 2016 · 0% born ≥2023 (10/10 dated) · coverage 3,838 postings
- **Title divergence:** 31.2% (1,502 / 4,818). Example: *"sr manager, data science & ai engineering"* (AI Engineer) lists CI/CD for Machine Learning, Experiment Tracking, Machine Learning, Machine Teaching, Model Deployment, Model Serving & Deployment.

---

## How we know the signal is real

The tests below were **pre-registered before reading the results.**

| Metric | Result | Meaning |
|---|---|---|
| Known emergent roles recovered | **7/7** | All roles known to have emerged via skill bleed appear as candidates. |
| Candidates judged non-artifact | **96%** | Manual audit. |
| Null-model separation | **z = −61** | Real role labels remove far more within-role noise than random labels. |
| Synonym leakage among bleed edges | **2.5%** | Against a 5% target. |

### The seven recovered roles
Each known role was matched to one of the detector's candidates without prompting:

| Known role | Matched candidate (rank) | Matched worlds | Example matched skills |
|---|---|---|---|
| AI Agent Developer | LLM Agent Engineer (#1) | AI Engineer, Software Architect | AI Agent Development, AI Agent Orchestration, Agent Orchestration, Memory-based Agents |
| AI Engineer | Generative AI Engineer (#4) | AI Engineer, ML Engineer | Generative AI, Large Language Models, LoRA, OpenAI API |
| ML Engineer | MLOps Engineer (#6) | Data Scientist, ML Engineer | Machine Learning, Model Deployment, Model Serving & Deployment, Model Training, Scikit-Learn |
| MLOps Engineer | MLOps Engineer (#6) | Data Scientist, DevOps Engineer, ML Engineer | CI/CD for Machine Learning, Experiment Tracking, ML Observability, Model Serving & Deployment |
| Analytics Engineer | Lakehouse Data Engineer (#8) | BI Developer, Data Analyst, Data Engineer | Data Lakehouse, Data Lineage, Data Modeling, Data Pipelines, Data Warehousing |
| Platform Engineer | Cloud Reliability Engineer (#12) | Cloud Engineer, DevOps Engineer, Site Reliability Engineer | IaC, Infrastructure as Code (IaC), Observability |
| DevSecOps | DevSecOps Engineer (#18) | DevSecOps Engineer, Security Engineer | Application Security, DevSecOps, Policy as Code, Vulnerability Management, Web Security |

### Null-model detail
20 permutations. Observed bleed graph has **9,875** cross-home edges vs null **12,137 ± 37** (z = −61.3). Real home labels remove ~19% more co-occurrence as within-role than random assignment, confirming the role taxonomy captures genuine structure. Community count (27 observed vs 23.75 ± 0.94 null) is not sensitive to home shuffling and is *not* the headline statistic.

### Synonym-filter detail
200 sampled edges · mean cosine 0.703 · max 0.779 · leak rate 2.5%. Suspected near-synonym pairs that survived include: UI/UX Design ↔ Survey Design (0.75), Azure ↔ Azure Resource Manager (0.77), Azure ↔ Azure OpenAI (0.76).

---

## What this study does NOT claim

- **It is a snapshot, not a trend.** Emergence is inferred from canonical drift and technology vintage, never from posting-date growth. The corpus spans ~five weeks.
- **It reflects US Fortune 500 hiring.** Large-enterprise, software-role filtered. Findings generalize to that population, not the whole labor market.
- **Skill extraction is imperfect.** An importance floor, a canonical-name map, and a blocklist reduce noise but do not eliminate it. One candidate is flagged an artifact and kept visible rather than hidden.
- **Technology dating is fuzzy.** Birth years are judgment calls from a versioned, auditable lookup; the emergence score is a rank, not a precise date.
- **Ubiquitous skills carry no signal.** A skill that co-occurs with nearly everything produces no bleed edge by design — a feature of the method, not a gap.
- **Hybrid jobs are prior art.** The phenomenon is well established (Lightcast, O\*NET, ESCO). The contribution is the snapshot-compatible, taxonomy-anchored, vintage-scored detection method.

---

## Why it matters

Andela assesses, matches, places, and upskills engineering talent to help enterprises adopt AI responsibly in production. This study is part of reading the technology talent market **by its skills rather than its job titles** — to see which capabilities are compounding in value before the labels catch up, and to tailor learning programs ahead of the market.

As skill half-lives shorten, the roles forming in the gaps between today's jobs are where tomorrow's **talent debt** accrues. Seeing them early is the point.

**Reproducibility.** Detected from 42,621 SDLC Fortune 500 postings, 2,521 skills scored, on a bleed graph of 734 skills. Deterministic (Louvain seed 42). Generated 5 June 2026.

---

*Distilled from `index.html`. The interactive ranking chart, vintage timeline, and bleed-graph node/edge data (the `GRAPH` blob) are visualization scaffolding and are not reproduced here — consult the HTML for those.*
