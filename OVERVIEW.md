# 🧭 AI-Enabled Labour Market Intelligence and Skill Demand-Supply Forecasting Engine

**SIH 2026 | Problem Statement ID: SIH26246**
**Ministry:** Skill Development and Entrepreneurship (MSDE) | **Problem Creator:** Sarim Moin, Ministry of Education's Innovation Cell (MIC)
**Category:** Software | **Technology Bucket:** Miscellaneous
**Project name:** `[TBD]`
**Team:** 6 members | **Build window:** 1 month

---

## 📌 1. Project Overview

### The problem
India trains lakhs of candidates every year across sectors, but training capacity is misaligned with real industry demand at the district and sector level. Some trades have chronic oversupply of certified candidates, while emerging trades are under-served. Planners lack a real-time, granular, decision-ready view of labour demand. Labour market data (PLFS, NCO-coded postings, industry hiring signals, e-Shram) is fragmented. Mismatches are discovered only after placement outcomes are reported, which is too late to change targets.

### What we are building
A **Labour Market Intelligence System (LMIS)** for MSDE, NCVET, Sector Skill Councils and state planning units that:

1. Aggregates and normalises labour demand signals from many sources.
2. Cross-references demand against training capacity and seat allocation by sector, trade and district.
3. Forecasts demand-supply gaps forward in time, refreshed periodically, not annually.
4. Ranks trades and geographies by severity of oversupply or undersupply.
5. Presents everything in an interactive, multilingual, accessible dashboard with national-to-district drill-down.
6. Exposes forecasts through an API and export layer for scheme target-setting workflows.

### Our differentiating idea
Most teams will submit: *scrape jobs, run a forecasting model, draw a heatmap.* We close the loop:

> **Signal → Forecast → Diagnosis → Intervention → Validation**

The system does not only say "shortage in district X." It says **why** the gap exists (skills, geography, wages, eligibility, timing, or weak evidence), **what to do** about it, **what happens if you do it**, and **proves it** by replaying history.

### Architectural argument: the closed loop
A thin **worker and employer layer** (job matching, ratings, protection) generates live labour signals that feed the planner dashboard. The worker side is not a second product. It is a data-generating layer for the LMIS.

```
Workers / Employers  --->  Live signals (wages, trust, registrations, travel-time)
        |                                   |
        v                                   v
Public data (PLFS, NCO, NQR, CPPP, Census, NCS snapshots, synthetic)
        |                                   |
        +-----------> Normalisation -> Demand Index -> Forecast -> Gap Ranking
                                                                   |
                         Diagnosis -> Simulator -> Planner Brief -> API / Export
```

---

## 👥 2. Users

| User | Needs |
|---|---|
| MSDE / NCVET planners | National and state view, target-setting, NSQF revisions |
| Sector Skill Councils | Sector demand, curriculum drift, emerging trades |
| State planning units | District view, regional-language UI, capacity planning |
| Training centre admins | Centre-level capacity and conversion guidance |
| Workers (thin layer) | Nearby fair jobs, safe employers, voice onboarding in own language |
| Employers (thin layer) | Verified postings, access to rated, nearby workers |

---

## 🧱 3. Layer 1: Core Modules (baseline)

| # | Module | What it does | Key components |
|---|---|---|---|
| 1 | 📥 Data Ingestion | Pulls demand and supply signals into one pipeline | Connectors for job portals (NCS, Naukri-type), PLFS, e-Shram, EPFO payroll, ASI, SSC hiring data, PMKVY/Skill India training data. Scheduler, CSV/API/scraper adapters |
| 2 | 🧹 Normalisation and Taxonomy | Makes messy data comparable | Job title to NCO-2015 mapping, trade to NSQF/QP mapping, NIC industry codes, LGD district codes, dedup, entity resolution, Hindi/regional title handling |
| 3 | 📊 Demand Index Engine | Fuses heterogeneous sources into one score per trade x district | Source weighting, recency decay, volume normalisation, confidence score |
| 4 | 🏫 Training Supply Module | Models the supply side | Centre capacity, annual seat allocation, enrolled, certified, placed, by trade/district/scheme |
| 5 | 🔮 Forecasting Engine | Forward-looking demand and supply | Hierarchical time series (national > state > district > trade), reconciliation, prediction intervals, periodic retraining |
| 6 | ⚖️ Gap Analysis and Ranking | Demand minus supply, ranked | Gap Severity Score, oversupply/undersupply ranking, leaderboards |
| 7 | 🚨 Early-Warning System | Flags problems before placement data shows them | Saturation and shortage thresholds, trend-break detection, alert feed |
| 8 | 🖥️ Interactive Dashboard | The planner's main screen | National > State > District drill-down, choropleth map, trade filters, time slider, compare mode |
| 9 | 🔌 API and Export Layer | Feeds scheme workflows | REST API, API keys, CSV/Excel/PDF export, scheduled exports |
| 10 | 🌐 Multilingual and Accessibility | State-level usability | Hindi + 3 to 4 regional languages, WCAG 2.1 AA, keyboard nav, screen-reader support, low-bandwidth mode |
| 11 | 🔐 Admin and Governance | Makes it deployable | RBAC (MSDE / NCVET / SSC / State), audit logs, data-quality dashboard, model monitoring |
| 12 | 📄 Methodology Documentation | Explicit deliverable | Data lineage, index formula, model cards, assumptions, limitations |

---

## ✅ 4. Layer 0: Basic Essentials (build before any USP)

| # | Essential | What it does | Feeds the LMIS how |
|---|---|---|---|
| E1 | 🤖 AI Overview | Grounded plain-language summary at every drill level (national, state, district, trade). The LLM only narrates numbers computed by the pipeline and never invents them. Each sentence links to its evidence | Decision-ready brief without reading charts |
| E2 | 📍 Proximity Job Matching | Jobs ranked by **travel time** (OSRM or Distance Matrix), not straight-line distance. Shows **net wage after commute cost** | Travel-time matrix doubles as the labour-shed engine |
| E3 | ⭐ Two-Sided Rating System | Workers rated by employers, employers rated by workers, verified completions only | Employer trust score down-weights demand from bad employers |
| E4 | 🛡️ Abuse and Scam Protection | See section 5 | Flagged employers are excluded from the demand index |
| E5 | 💰 Fair-Wage Benchmark | Every posting compared to state minimum wage and market median for trade and district. Below-floor postings flagged or blocked | Becomes the wage-pressure signal |
| E6 | 🗣️ Voice onboarding, multilingual | Worker registers by speaking in Hindi or a regional language (reuse from KisanLink) | Cleaner supply data |
| E7 | 🆔 Verified profiles | Phone OTP, optional e-Shram ID linking, employer verification via Udyam or GSTIN | Removes fake demand and fake supply |
| E8 | 🔔 Alerts | SMS or WhatsApp for matches, pay due, grievance status | Retention |

### Worker ranking rule
Do **not** rank workers by rating alone. New workers would never get hired and rating bias is real.

```
Match score = skill fit + travel time + smoothed rating + availability
```

- Bayesian-smoothed rating with a minimum review count
- New-worker boost
- Weights configurable by planner or state

---

## 🛡️ 5. Protection Measures (E4 in detail)

| Threat | Countermeasure |
|---|---|
| Wage below legal floor | Auto-check against state minimum wage notifications. Block or red-flag the posting |
| Employer retaliation via bad ratings | Verified-job-only ratings, outlier employers discounted, dispute option for workers |
| Delayed or unpaid wages | Both sides confirm payment. Overdue payments raise employer penalty score and trigger escalation prompt |
| Placement-fee scams | Classifier flags "pay to apply" language. Report button |
| Fake or ghost postings | Dedup, employer verification, posting expiry |
| Unsafe or night work, especially for women | Shift and location risk flags, safety rating category, SOS contact |
| Underage workers | Age gate. Flag hazardous-trade postings |
| Contract opacity | AI explains the offer in the worker's language by voice and text before acceptance |
| Grievance with no outlet | In-app complaint routed to the state labour department. Status tracked |
| Data misuse | DPDP Act 2023 consent, minimal data, no Aadhaar storage |

### 🔁 How worker-side signals strengthen the LMIS

| Worker-side signal | LMIS use |
|---|---|
| Employer trust score | Replaces blind vacancy counts (Vacancy Reality Filter) |
| Posted wages and payment records | Wage-Pressure Validator, Job-Quality Adjusted Demand |
| Travel-time matrix | Labour-Shed Catchments, Cannibalisation Detector |
| Registered workers by trade and district | Live supply signal, not just certified-candidate counts |
| Abuse and grievance reports | Early-warning flag, e.g. "trade X in district Y shows a wage-theft pattern" |

---

## 🏆 6. Layer 2: USPs (full catalogue)

### 6.1 Our original USPs

| # | USP | Why it matters | Difficulty |
|---|---|---|---|
| U1 | ⏪ **Counterfactual Time Machine** | Replay 2019 to 2024: "Had MSDE used this, X seats in Y trades would have moved, Z% fewer unplaced candidates." Proof, not promise | Medium |
| U2 | 🎯 **Target-Setting Simulator** | "What if I cut 500 electrician seats in District X?" Directly serves annual target setting | Medium |
| U3 | ⏳ **Pipeline-Adjusted Supply** | Counts trainees currently in training as future supply with a 3 to 12 month lag by course duration | Medium |
| U4 | 🗣️ **Natural-Language and Voice Query** | "Which districts in UP have welder oversupply?" in Hindi or regional language, answer as chart plus spoken reply | Easy-Medium |
| U5 | 🧠 **Explainable Flags** | "Flagged because postings fell 32% while seats rose 18%." | Easy |
| U6 | 📉 **Data Confidence Score per Cell** | Every number carries a reliability grade | Easy |
| U7 | 📐 **Hierarchical Forecast Reconciliation** | District forecasts sum to state and national, with uncertainty bands | Medium |
| U8 | 📑 **Auto Policy Brief Generator** | One-click state-wise PDF with charts and recommendations | Easy |
| U9 | 🔄 **Scenario Mode** | Test shocks: new factory cluster, PLI scheme, monsoon impact on agriculture | Medium |
| U10 | 👩 **Gender and Inclusion Lens** | Gap view split by gender, SC/ST, rural/urban | Easy |
| U11 | 🧪 **Synthetic Data Fallback** | Plausible generated data where real data is missing, clearly labelled | Easy |
| U12 | 📡 **Leading-Indicator Radar** | NLP on factory announcements and PLI approvals predicts demand 6 to 12 months ahead | Medium-Hard |
| U13 | 🏗️ **Centre Sanction Optimiser** | OR-Tools output: where to open or expand, which trades, how many seats | Hard |

### 6.2 Diagnosis and intervention USPs (from ChatGPT brainstorm)

Scores are Impact x Feasibility, each out of 5, for a 6-person SIH team.

| # | USP | Score | Planner problem solved | Person-days | Demo moment |
|---|---|---:|---|---:|---|
| C1 | **Mismatch Cause Engine** | 25 | A shortage does not automatically mean "train more people". Identifies whether the cause is skills, geography, wages, eligibility, timing or weak evidence | 5 | "Electrician, Gurugram: shortage 1,240. Cause: 68% mobility friction. Do not add seats." |
| C2 | **SkillBridge (Reskilling Path Graph)** | 25 | What to do with workers in an oversupplied trade | 5-6 | "Sewing Machine Operator to Industrial Sewing Technician: 72% skills reusable, 3 missing modules, 96 bridge hours" |
| C3 | **Curriculum Drift Detector** | 25 | Courses keep producing candidates while employer skills have moved on | 5-6 | "31% of vacancies request PLC diagnostics, curriculum does not cover it" |
| C4 | **Training Centre Reconfiguration Engine** | 20 | A new centre may be wasteful when an existing one can be converted | 7-8 | "Centre #42 oversupplies welding: convert to solar technician instead of opening new" |
| C5 | **Pre-Vacancy Demand Radar** | 20 | Postings appear after investment decisions. Detect demand earlier from tenders and awards | 7-8 | Solar EPC tender appears: "electricians +18%, solar installers +32%" |
| C6 | **Eligibility Bottleneck Detector** | 20 | 2,000 seats are useless if the district lacks eligible candidates | 6 | "Only ~740 locally eligible, add bridge course instead" |
| C7 | **Emerging Occupation / NCO Blind-Spot Detector** | 20 | New jobs may not map to existing taxonomy | 5 | 600 postings fail NCO mapping, cluster surfaces as "EV Battery Diagnostic Technician" |
| C8 | **Vacancy Reality Filter** | 20 | Portals exaggerate demand via duplicates and mass recruitment | 3-4 | Raw 1,430 vacancies to deduped 820, one employer is 61%, confidence downgraded |
| C9 | **Demand Arrival Clock** | 20 | A right trade is useless if candidates graduate after the hiring peak | 3 | "Demand peaks Jan 2028, 6-month course must start by June 2027" |
| C10 | **Skill Bundle Miner** | 20 | Employers hire skill combinations, qualifications teach isolated skills | 4 | "Welder + blueprint reading + basic CNC appear together in 64% of vacancies" |
| C11 | **Wage-Pressure Validator** | 20 | Vacancy count alone does not prove scarcity | 3 | Two trades with +40% postings, only one has rising salaries |
| C12 | **Job-Quality Adjusted Demand** | 20 | Maximising vacancy count can push training toward low-quality jobs | 4 | 5,000 low-wage temp openings no longer outweigh 2,500 stable higher-wage ones |
| C13 | **Skill Portability / Resilience Index** | 16 | Narrow qualifications are risky when demand shifts | 4 | Course B maps to 6 growing occupations, shown as more resilient |
| C14 | **Skill Volatility Index** | 16 | Some occupations change too fast for full-course revisions | 4 | "Data-centre technician skills changed 29% in 18 months, use modular add-ons" |
| C15 | **Hidden-Demand Correction Layer** | 15 | Job portals under-represent blue-collar and informal work | 7-8 | 200 online mason vacancies corrected upward |
| C16 | **Labour-Shed / Commuting Catchments** | 15 | District boundaries are not labour-market boundaries | 7 | Gurugram shortage accounts for Delhi and Faridabad supply |
| C17 | **Shock-to-Occupation Ripple Engine** | 15 | Planners know a project is coming but cannot translate it to occupations | 6-7 | "EV plant: 3,000 direct jobs" expands to technicians, electricians, logistics, QA |
| C18 | **Analog District / Cold-Start Forecasting** | 16 | Sparse districts lack history | 5 | Borrow calibrated pattern from nearest labour-market twins |
| C19 | **Training Capacity Cannibalisation Detector** | 16 | Adjacent centres flood the same labour market | 5 | "720 competing seats for only 410 jobs" |
| C20 | **Forecast Evidence Ledger** | 16 | Planner must defend why a forecast changed | 4 | "Why did shortage jump? NCS +14%, tender +8%, enterprise +3%, supply -4%, model v1.4" |

---

## 🎯 7. Build Priority (final ranking)

| Order | Item | Verdict |
|---|---|---|
| 1 | Core 12 modules (section 3) | ✅ Must |
| 2 | Essentials E1 to E8 (section 4) | ✅ Must |
| 3 | Mismatch Cause Engine (C1) | ✅ Hero feature |
| 4 | Counterfactual Time Machine (U1) | ✅ Best pitch moment, build early |
| 5 | Target-Setting Simulator (U2) | ✅ Direct use case |
| 6 | SkillBridge (C2) | ✅ |
| 7 | Curriculum Drift Detector (C3) | ✅ |
| 8 | Vacancy Reality Filter (C8), Wage-Pressure Validator (C11), Evidence Ledger (C20) | ✅ Cheap, 3-4 days each |
| 9 | Pipeline-Adjusted Supply (U3), Explainable Flags (U5), Confidence Score (U6), Reconciliation (U7) | ✅ Small, high credibility |
| 10 | Pre-Vacancy Demand Radar (C5) | 🟡 Demo-grade only |
| 11 | Labour-Shed Catchments (C16) | 🟡 Cheap because of E2 |
| 12 | NL/Voice Query (U4), Policy Brief PDF (U8) | 🟡 If time allows |
| 13 | Centre Reconfiguration (C4), Eligibility Bottleneck (C6), Centre Sanction Optimiser (U13) | ❌ Cut unless ahead of schedule |
| 14 | Hidden-Demand Correction (C15), Analog District (C18), Skill Volatility (C14) | ❌ Cut, needs data we will not have |

**Rule:** do not attempt everything. Ship the ✅ rows well.

---

## 🧩 8. Mapping to Problem Statement Deliverables

| Expected outcome | Where we satisfy it |
|---|---|
| Functioning forecasting dashboard for a few pilot sectors and States | Module 8 + pilot dataset |
| Documented methodology for combining heterogeneous sources into one demand index | Modules 3 and 12 |
| Early-warning flags for saturation or acute shortage | Module 7 + Explainable Flags |
| API/export layer for scheme target-setting workflows | Module 9 + Target-Setting Simulator |
| Multilingual, accessible interface for state planning units | Module 10 + E6 |

---

## 🗄️ 9. Data Strategy

- **Do not make scraping the project.** Build a reproducible pilot dataset snapshot plus synthetic extension. Connectors are replaceable ingestion adapters.
- **Label provenance on every value:** `observed`, `estimated`, `synthetic`, or `modelled`.
- **Public primitives available:** NCO 2015 (searchable/downloadable), NQR (qualification titles, NSQF levels, duration), NCS public job search (district, sector, salary fields), CPPP tenders and awards, Census migration tables, PLFS releases.
- **Caveats:**
  - Census migration tables are 2011. Use as a structural prior, not a live mobility signal.
  - PLFS is not district-level real-time. Monthly all-India and quarterly State/UT estimates, annual unit-level data for deeper analysis.
- **Real NCS and e-Shram feeds are assumed unavailable.** Pilot runs on public data plus clearly labelled synthetic data.
- **Worker and employer volumes in the demo are seeded or simulated.**

---

## 🖥️ 10. UX Principles

1. **Organise around decisions, not charts.** Hero flow: *Where is the problem? > Why does it exist? > What intervention fixes it? > What happens if I do it?*
2. **Planner Brief card is the signature element.** Example: `Gurugram | CNC Operator | shortage 1,180 | high confidence | root cause: curriculum mismatch | action: shift 320 seats + add PLC module | decision needed before 15 June`
3. **Voice-first and multilingual from day 1.** Use i18n keys from the start, not a retrofit.
4. **Low-bandwidth mode and WCAG 2.1 AA** for state planning units on weak connections.
5. **Every number shows its confidence and provenance.**

---

## 👥 11. Team Split and Timeline

| Person | Owns | Weeks 1-2 | Weeks 3-4 |
|---|---|---|---|
| 1 | Data and pipeline | Ingestion, NCO/NSQF mapping, synthetic data | Vacancy Reality Filter, Evidence Ledger |
| 2 | Forecasting and ML | Demand index, hierarchical forecasts | Time Machine backtest |
| 3 | Diagnosis | Gap ranking | Mismatch Cause Engine, SkillBridge, Drift Detector |
| 4 | Worker-side backend | Auth, jobs, proximity, ratings | Protection rules, fair-wage checks |
| 5 | Frontend and dashboard | Drill-down, map, i18n | Planner Brief card, simulator UI |
| 6 | AI, voice and design | Voice onboarding (KisanLink), AI Overview | NL query, policy brief PDF, pitch deck, demo script |

**Feature freeze at end of week 3.** Week 4 is for polish, demo, and the methodology document.

---

## 🎤 12. Pitch Narrative

- Do not claim "we predict jobs."
- Say: *"Existing LMIS approaches tell planners what the labour market looks like. We close the loop from signal to forecast to diagnosis to intervention to validation."*
- Demo beats:
  1. National map to district drill-down, AI Overview narrates it.
  2. Two equal-sized shortages. The Mismatch Cause Engine prescribes completely different interventions.
  3. One case where the system explicitly says **"do not increase training seats"** despite a headline shortage.
  4. Simulator: cut or add seats, see the gap move.
  5. Time Machine: replay history and show the measurable gain.
  6. Worker voice onboarding in Hindi, with the signal appearing on the planner dashboard.

---

## ⚠️ 13. Risks

| Risk | Mitigation |
|---|---|
| Scope creep from the worker side | Keep it thin. It exists to feed the LMIS |
| No real government data access | Public snapshots plus labelled synthetic data |
| Too many USPs, none polished | Freeze at week 3, ship only ✅ rows |
| LLM hallucination in AI Overview | LLM narrates pipeline numbers only, every sentence links to evidence |
| Rating bias and retaliation | Verified-only ratings, smoothing, dispute path |
| Over-claiming forecast accuracy | Prediction intervals and confidence score everywhere |

---

## 📌 14. Assumptions and Open Questions

**Assumptions**
- Stack is free to choose.
- Team covers ML, full-stack and design.
- Voice onboarding and i18n are reused from KisanLink.

**Open**
- Pilot states (suggested: Haryana + UP)
- Pilot sectors (suggested: manufacturing + construction)
- Final tech stack
- Project name
- Languages to support beyond Hindi

---

## 📚 15. Reference: ChatGPT Brainstorm Prompt

```
GOAL: Brainstorm original, judge-impressing USPs for SIH26246 (MSDE labour market intelligence and skill demand-supply forecasting).

CONTEXT: 6 students, 1 month. Already planned: ingestion, NCO/NSQF mapping, demand index, hierarchical forecasts, gap ranking, early warnings, dashboard, API, multilingual voice interface, target-setting simulator, pipeline-adjusted supply, counterfactual backtest. Users: MSDE, NCVET, SSC, state planners.

TASK: Propose 20 USPs beyond the planned list. Each: idea, planner problem solved, data needed, person-days, demo moment.

CONSTRAINTS: Public or synthetic data only. Under 10 person-days each. No generic dashboard features.

OUTPUT: Table ranked by judge impact times feasibility; 3-line build sketch for top 5; then 5 side notes on data strategy, UI/UX, pitch.
```
