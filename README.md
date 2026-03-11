# MOSAIC
## A Multi-Objective Signal Aggregation Framework for Forensic Anomaly Detection in Complex Panel Data

**Author:** Lawrence Fulton · Boston College  
**Application Domain:** U.S. Hospital Financial Data (CMS HCRIS, 2011–2024)  
**Target Venue:** Expert Systems with Applications

---

## Overview

**MOSAIC** is a generalizable multi-signal fusion framework for forensic anomaly detection in complex longitudinal panel data. It jointly optimizes signal weights across heterogeneous anomaly detectors — combining statistical, machine learning, and behavioral forensic signals — within regime-aware, peer-stratified cohorts.

The framework is demonstrated on **U.S. hospital financial data** sourced from the Centers for Medicare & Medicaid Services (CMS) **Hospital Cost Report Information System (HCRIS)**. However, the architecture is domain-agnostic and applicable to any high-dimensional panel dataset where anomaly signals are multidimensional and context-dependent.

---

## Framework Architecture

MOSAIC operates through five integrated layers:

| Layer | Function |
|-------|----------|
| **Ingestion** | Raw panel data extraction and normalization |
| **Feature Engineering** | Multi-category forensic feature construction |
| **Peer Stratification** | Regime-aware cohort construction for contextual benchmarking |
| **Signal Fusion** | Joint optimization of heterogeneous anomaly detector weights |
| **Scoring** | Hospital–year anomaly scores with interpretable signal decomposition |

### Anomaly Detectors

MOSAIC fuses four complementary signal classes:

- **Benford's Law** — digit-frequency deviation as a proxy for fabrication or manipulation
- **Mahalanobis Distance** — multivariate outlier detection relative to peer-cohort structure
- **Isolation Forest** — nonparametric isolation of anomalous observations in high-dimensional space
- **Variational Autoencoder (VAE)** — deep generative model capturing latent structural regularities; reconstruction error as anomaly score

Signal weights are **jointly optimized per peer cohort and time regime**, not fixed globally.

---

## Data Source

All raw data originate from:

**CMS Healthcare Cost Report Information System (HCRIS)**  
https://www.cms.gov

Specifically the **HOSP10** annual files:

| File | Description |
|------|-------------|
| `RPT` | Report metadata and provider identifiers |
| `NMRC` | Numeric worksheet data |
| `ALPHA` | Alphanumeric worksheet data |

CMS cost reports store financial values by **worksheet address** (Worksheet / Line / Column). Example: `G300000 / 00300 / 00100` → Net Patient Revenue.

All worksheet mappings were **empirically validated** across fiscal years 2011–2023 using worksheet sniffing techniques rather than relying on documentation alone.

---

## Pipeline Architecture

```
HOSP10FYYYYY/
├── hosp10_YYYY_RPT
├── hosp10_YYYY_NMRC
└── hosp10_YYYY_ALPHA
```

The pipeline processes raw CMS files through five stages:

1. **Raw Data Ingestion** — Parquet-preferred with CSV fallback
2. **Worksheet Address Mapping** — Empirically validated coordinate-to-variable translation
3. **Feature Engineering** — 200+ derived variables across ten conceptual categories
4. **Peer Group Construction** — Stratified cohorts with minimum cell size enforcement
5. **Panel Assembly** — Hospital–year panel in Parquet format

---

## Feature Engineering

Variables are organized into ten conceptual categories:

| # | Category | Examples |
|---|----------|---------|
| 1 | Clinical & Operational Structure | licensed beds, patient days, discharges, ER visits |
| 2 | Revenue Integrity | cost-to-charge ratio, collection efficiency, medicare dependency |
| 3 | Liquidity & Solvency | current ratio, days cash on hand, technical insolvency flag |
| 4 | Cost Structure | labor intensity, admin load, diagnostic weight |
| 5 | Bankruptcy Signals | bankruptcy proxy score, liquidity trap, cash exhaustion |
| 6 | Cash Flow Velocity | AR velocity, AP lag, asset turnover |
| 7 | Structural Transfers | subsidy intensity, siphon ratio |
| 8 | Temporal Velocity | YoY % change metrics, delta financials |
| 9 | Behavioral Forensic Signals | ghost labor, supply starvation, collection collapse |
| 10 | Market Context | relative operating margin vs. state-year median |

All analytic variables are winsorized at **1% tails**.

---

## Peer Group Construction

Hospitals are evaluated relative to structurally similar peers. Peer groups are defined by:

- Ownership type (nonprofit / for-profit / government)
- Hospital size (small / medium / large)
- Teaching status (major teaching / minor teaching / non-teaching)
- Critical Access Hospital (CAH) status
- Case mix tier (fine-grained groups)

Example peer group label: `nonprofit_large_minor_teaching`

Minimum cell sizes are enforced to prevent unstable comparisons.

---

## Time Regimes

| Regime | Years | Notes |
|--------|-------|-------|
| Transition | ≤ 2011 | Pre-panel anchor |
| Baseline | 2012–2018 | Peer-group structure anchor |
| COVID Shock | 2019–2021 | Structural break period |
| Recovery | 2022–2024 | Post-shock normalization |

Signal weights and peer-group benchmarks are conditioned on regime, allowing MOSAIC to distinguish **secular anomalies** from **regime-driven distributional shifts**.

---

## Output

```
processed_panel/
└── master_hospital_research_panel.parquet
```

| Property | Value |
|----------|-------|
| Unit of observation | Hospital–Year |
| Years covered | 2011–2024 |
| Variables | 200+ |
| File format | Parquet (Snappy compression) |

---

## Usage

```python
from mosaic.pipeline import run_year

YEARS_TO_PROCESS = list(range(2011, 2025))

for yr in YEARS_TO_PROCESS:
    result = run_year(yr, BASE_DIR)
```

---

## Dependencies

```
pandas
numpy
pyarrow
scikit-learn
torch          # VAE component
pathlib
```

**Recommended:** Python 3.10+, CUDA-capable GPU for VAE training

---

## Generalizability

Although demonstrated on CMS HCRIS data, MOSAIC's architecture applies to any complex panel setting where:

- Anomaly signals are heterogeneous and not reducible to a single detector
- Peer-relative context matters (cohort-conditioned scoring)
- Temporal regimes create non-stationarity that fixed global thresholds cannot handle
- Interpretability of signal contributions is required

Candidate domains include insurance claims, financial regulatory filings, supply chain audits, and public sector expenditure monitoring.

---

## Disclaimer

This repository processes publicly available CMS data but does **not** contain the raw CMS files. Users must obtain HCRIS data directly from CMS at https://www.cms.gov.

---

## Citation

If you use MOSAIC in your research, please cite:

> Fulton, L. (2025). *MOSAIC: A Multi-Objective Signal Aggregation Framework for Forensic Anomaly Detection in Complex Panel Data.* 
