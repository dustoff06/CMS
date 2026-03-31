# MOSAIC
## A Multi-Objective Signal Aggregation Framework for Forensic Anomaly Detection in Complex Panel Data

**Author:** Lawrence Fulton · Boston College  
**Application Domain:** U.S. Hospital Financial Data (CMS HCRIS, 2011–2024)  
**Target Venue:** Expert Systems with Applications  

---

## Overview

**MOSAIC** is a multi-objective signal fusion framework for forensic anomaly detection in complex longitudinal panel data. It integrates heterogeneous anomaly signals — spanning statistical diagnostics, behavioral indicators, and machine learning models — into a unified, interpretable scoring system.

Unlike conventional anomaly detection approaches that rely on a single model or fixed heuristic weights, MOSAIC jointly optimizes signal weights within **peer-stratified cohorts** and **time regimes**, enabling context-aware prioritization in non-stationary environments.

The framework is demonstrated on **U.S. hospital financial data** from the Centers for Medicare & Medicaid Services (CMS) **Hospital Cost Report Information System (HCRIS)**. However, the architecture is domain-agnostic and applicable to any high-dimensional panel where anomaly signals are multidimensional and context-dependent.

---

## Framework Architecture

MOSAIC operates through five integrated layers:

| Layer | Function |
|-------|----------|
| **Ingestion** | Raw panel data extraction and normalization |
| **Feature Engineering** | Construction of financial, behavioral, and statistical signals |
| **Peer Stratification** | Cohort construction for context-aware benchmarking |
| **Signal Fusion** | Joint optimization of heterogeneous anomaly signal weights |
| **Scoring** | Facility–year anomaly scores with interpretable signal contributions |

---

## Anomaly Detectors

MOSAIC combines four complementary anomaly detection mechanisms:

- **Benford's Law** — detects digit-frequency irregularities indicative of manipulation  
- **Mahalanobis Distance** — measures multivariate deviation from peer-cohort structure  
- **Isolation Forest** — identifies anomalous observations through recursive partitioning  
- **Variational Autoencoder (VAE)** — learns latent structural representations; reconstruction error signals deviation from learned norms  

These detectors are not used independently. Instead, their outputs are **fused through a constrained optimization framework**, allowing MOSAIC to reconcile conflicting signals and produce a single interpretable anomaly score.

---

## Data Source

All data originate from:

**CMS Hospital Cost Report Information System (HCRIS)**  
https://www.cms.gov  

Specifically, the **HOSP10** annual releases:

| File | Description |
|------|-------------|
| `RPT` | Report metadata and provider identifiers |
| `NMRC` | Numeric worksheet data |
| `ALPHA` | Alphanumeric worksheet data |

Financial values are stored using **worksheet–line–column coordinates**, requiring explicit mapping to reconstruct usable variables. All mappings were empirically validated across fiscal years due to undocumented schema drift.

---

## Data Pipeline
HOSP10FYYYYY/
├── hosp10_YYYY_RPT
├── hosp10_YYYY_NMRC
└── hosp10_YYYY_ALPHA


The pipeline processes raw CMS data through:

1. **Data Ingestion** — conversion to a unified columnar structure  
2. **Worksheet Mapping** — coordinate-based variable reconstruction  
3. **Feature Engineering** — generation of 200+ derived variables  
4. **Peer Group Construction** — cohort assignment with minimum cell sizes  
5. **Panel Assembly** — final hospital–year dataset  

---

## Feature Engineering

Signals are constructed across multiple conceptual domains:

- Clinical and operational structure  
- Revenue integrity and billing behavior  
- Liquidity and solvency  
- Cost structure  
- Cash-flow dynamics  
- Structural transfers  
- Temporal dynamics  
- Behavioral forensic indicators  
- Market context  

All continuous variables are winsorized at the **1% tails** to mitigate extreme outliers.

---

## Peer Stratification

Hospitals are evaluated relative to structurally similar peers. Cohorts are defined using:

- Ownership type (nonprofit, for-profit, government)  
- Critical Access Hospital (CAH) designation  
- Additional stratification features where sample size permits  

Minimum cell sizes are enforced to ensure stable estimation.

---

## Time Regimes

| Regime | Years | Description |
|--------|-------|-------------|
| Transition | ≤ 2011 | Pre-panel alignment |
| Baseline | 2012–2018 | Training period |
| Shock | 2019–2021 | Structural break (COVID-era) |
| Recovery | 2022–2024 | Post-shock normalization |

---

## Training and Out-of-Sample Scoring

MOSAIC follows a **strict regime-based training design**:

- All normalization parameters, decile thresholds, and signal weights are estimated **exclusively on the Baseline period (2012–2018)**  
- These parameters are then **applied unchanged** to all subsequent periods (2019–2024)  

This produces fully **out-of-sample anomaly scores**, ensuring that:

- no temporal leakage occurs  
- post-shock anomalies reflect genuine deviations from pre-shock structure  
- regime shifts are not absorbed into model re-estimation  

This design is critical for forensic applications where stability and interpretability under distributional shift are required.

---

## Output
processed_panel/
└── master_hospital_research_panel.parquet

| Property | Value |
|----------|-------|
| Unit of observation | Hospital–Year |
| Years covered | 2011–2024 |
| Variables | 200+ |
| Format | Parquet (Snappy compression) |

---

Generalizability

MOSAIC is applicable to any domain where:

anomaly signals are heterogeneous
peer-relative benchmarking is required
temporal non-stationarity is present
interpretability is essential

Example domains include:

insurance claims auditing
financial regulatory filings
supply chain monitoring
public sector expenditure analysis
Disclaimer

This repository processes publicly available CMS data but does not include raw HCRIS files. Users must obtain data directly from CMS.



