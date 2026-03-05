# Hospital Financial Forensics Panel (CMS HCRIS)

## Overview

This project constructs a **longitudinal forensic research panel of U.S. hospitals** using the Centers for Medicare & Medicaid Services (CMS) **Hospital Cost Report Information System (HCRIS)**.

The pipeline extracts financial, operational, and structural data from CMS cost reports and converts them into a standardized **hospital–year panel dataset** suitable for:

- financial distress detection  
- anomaly detection  
- structural hospital comparisons  
- machine learning applications  
- forensic accounting research  
- healthcare market structure analysis  

The resulting dataset spans **2011–2024** and contains hundreds of derived features designed to detect **behavioral, structural, and financial anomalies** in hospital operations.

The final output is a **fully normalized research panel stored in Parquet format** for high-performance analytics.

---

# Data Source

All raw data originate from:

**CMS Healthcare Cost Report Information System (HCRIS)**  
https://www.cms.gov

Specifically the **HOSP10 files**, which include:

| File | Description |
|-----|-------------|
| `RPT` | Report metadata and provider identifiers |
| `NMRC` | Numeric worksheet data |
| `ALPHA` | Alphanumeric worksheet data |

These files contain the full financial statements submitted by U.S. hospitals to CMS for Medicare reimbursement purposes.

---

# Architecture

The pipeline converts raw CMS cost report files into a **structured hospital research panel** through five major stages.

## 1. Raw Data Ingestion

Each fiscal year folder contains:
HOSP10FYYYYY/
hosp10_YYYY_RPT
hosp10_YYYY_NMRC
hosp10_YYYY_ALPHA

The system reads **Parquet versions of these files** (preferred for performance), with automatic fallback to CSV if needed.

Key extraction tables:

- `RPT` — provider identifiers and reporting metadata
- `NMRC` — numeric worksheet data
- `ALPHA` — textual identifiers
---

## 2. Worksheet Address Mapping

CMS cost reports store financial values by **worksheet address**:
Worksheet | Line | Column

Example: G300000 / 00300 / 00100 represents **Net Patient Revenue**.

The pipeline maps these worksheet coordinates into research variables using the configuration:

LOCKED_NUMERIC_FIELDS
IDENTIFIER_FIELDS

Example mapping: ("net_patient_revenue", "G300000", "00300", "00100")

These mappings were **empirically validated across fiscal years 2011–2023** using worksheet sniffing techniques.

---

# Feature Engineering

The system constructs a large number of analytical variables organized into **ten conceptual categories**.
---

## Category 1 — Clinical & Operational Structure
Measures scale and patient throughput.

Examples:
- licensed beds
- patient days
- discharges
- ER visits
- outpatient volume

Derived indicators:
revenue_per_discharge
outpatient_pivot
forensic_efficiency
---

## Category 2 — Revenue Integrity

Measures billing structure and payer mix.

Examples:
ccr cost-to-charge ratio
collection_efficiency
medicare_dependency
charity_commitment
---

## Category 3 — Liquidity & Solvency

Captures short- and long-term financial health.

Examples:
current_ratio
days_cash_on_hand
equity_financing
return_on_assets

Distress indicators:
liquidity_trap
cash_exhaustion
technical_insolvency
---

## Category 4 — Cost Structure

Measures operational cost composition.

Examples:
labor_intensity
admin_load
high_risk_diagnostic_weight
---

## Category 5 — Bankruptcy Signals

Composite indicators of financial collapse.

Example:
bankruptcy_proxy_score = insolvency
liquidity_trap
cash_exhaustion
---

## Category 6 — Cash Flow Velocity

Captures working capital stress.

Examples:
ar_velocity
ap_lag
asset_turnover
---

## Category 7 — Structural Transfers

Measures subsidy dependence and corporate siphoning.

Examples:
subsidy_intensity
siphon_ratio
---

## Category 8 — Temporal Velocity

Year-over-year change metrics:
pct_change_volume_metrics
delta_financial_metrics

These capture **operational momentum and shock events**.
---

## Category 9 — Behavioral Forensic Signals

Examples:
### Ghost Labor
Volume falling while payroll rises.

### Supply Starvation
Clinical supplies reduced while administrative costs increase.

### Collection Collapse
Receivables growing while liquidity deteriorates.

These patterns often appear **before formal distress events**.
---

## Category 10 — Market Context

Hospitals are evaluated relative to their market.

Example:
rel_operating_margin =
hospital_margin − state_year_median
---

# Peer Group Construction

Hospitals are compared within structurally similar peer groups.

Peer groups incorporate:
- ownership type
- hospital size
- teaching status
- Critical Access Hospital status

Example peer group:
nonprofit_large_minor_teaching

Fine peer groups also incorporate **case mix tiers**.

Minimum cell sizes are enforced to prevent unstable comparisons.
---

# Data Regimes

Hospitals are categorized by macroeconomic regime:

| Regime | Years |
|------|------|
| Transition | ≤ 2011 |
| Baseline | 2012–2018 |
| COVID Shock | 2019–2021 |
| Recovery | 2022–2024 |

Baseline years anchor peer-group structure.
---

# Data Cleaning & Quality Control

The pipeline performs extensive normalization:
- worksheet coordinate normalization
- numeric type coercion
- outlier winsorization
- denominator protection
- infinite value removal

All analytic variables are winsorized at **1% tails**.
---

# Output

Final dataset:

processed_panel/
master_hospital_research_panel.parquet

Key characteristics:

| Property | Value |
|--------|-------|
| Unit of observation | Hospital–Year |
| Years covered | 2011–2024 |
| Variables | 200+ |
| File format | Parquet |
| Compression | Snappy |
---

# Performance Design

The pipeline is optimized for large-scale CMS data.

Key design choices:
- Parquet storage for fast IO
- vectorized extraction from worksheet tables
- address-based data mapping
- minimal repeated scans of NMRC tables
---

# Example Use Cases

The dataset supports:
- anomaly detection models
- financial distress prediction
- hospital market structure research
- healthcare fraud detection
- machine learning models
- VAE anomaly detection
- policy analysis
---

# Running the Pipeline

Example workflow:

```python
YEARS_TO_PROCESS = list(range(2011, 2025))

for yr in YEARS_TO_PROCESS:
    result = run_year(yr, BASE_DIR)
```

The pipeline automatically builds the master research panel.

Dependencies

Python libraries:
pandas
numpy
pyarrow
pathlib
re

Recommended environment:
Python 3.10+
Disclaimer

This repository processes publicly available CMS data but does not contain the raw CMS files.

Users must obtain HCRIS data directly from CMS.

Author

Lawrence Fulton
Boston College
