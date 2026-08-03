# MOSAIC
## A Fusion Framework for Prospective Anomaly Prioritization in Complex Administrative Panel Data

**Author:** Lawrence Fulton · Boston College  
**Application Domain:** U.S. Hospital Financial and Administrative Data (CMS HCRIS, FY2011–2024)  
**Target Venue:** *Measurement*  
**Status:** Revised Manuscript

---

## Overview

**MOSAIC** (Multi-Objective Signal Aggregation for Interpretable Classification) is a weakly supervised prioritization framework for complex longitudinal administrative panel data. It combines heterogeneous analytical, behavioral, temporal, statistical, and model-based signals into a unified and interpretable composite score through constrained multi-objective optimization.

The framework addresses a practical oversight problem: regulators and Medicare Administrative Contractors must prioritize limited review resources across tens of thousands of hospital cost reports. MOSAIC is designed to rank report records for further review, not to classify fraud, misconduct, or regulatory violations directly.

Applied to **83,512 distinct HCRIS report records** from FY2011–2024, MOSAIC estimates its aggregation structure during a pre-pandemic Baseline period and applies the fitted weights unchanged across later fiscal regimes.

| Primary Metric | Value |
|----------------|-------|
| Global Spearman concordance with distress reference | **ρ_S = 0.5622** |
| Peer-year rank concordance | **ρ_S = 0.5386** |
| Peer/year fixed-effect concordance | **ρ_S = 0.5814** |
| Quartile separation | **0.4035** |
| Baseline-calibrated operating threshold | **τ = 0.5153** |
| False-positive rate | **4.82%** |
| True-positive rate | **41.02%** |
| Share of records flagged | **10.40%** |

Signal-exclusion robustness through **MOSAIC-X** (K = 86) yields global concordance of **ρ_S = 0.5385**, confirming that the primary findings are not driven solely by operating-margin-adjacent signals.

---

## Interpretive Framing

MOSAIC separates three related but distinct concepts:

1. **Financial-Distress Reference** — a weak supervisory orientation based on liquidity, operating margin, and days cash on hand
2. **Reporting-Pattern Signals** — statistical, temporal, behavioral, structural, and model-derived deviations in submitted administrative data
3. **Review Priority** — a composite ranking used to identify report records warranting closer examination

In this project, *reporting anomaly* is an operational label for deviations in submitted data. It does not denote an independently adjudicated error, regulatory violation, or instance of misconduct. External validation therefore evaluates prospective institutional-risk prioritization rather than confirmed fraud detection.

---

## Framework Architecture

MOSAIC operates through six integrated pipeline stages:

| Stage | Description |
|-------|-------------|
| **Data Ingestion** | Raw HCRIS extraction, report-record alignment, harmonization, and normalization |
| **Preprocessing** | Regime assignment, peer-cohort construction, feature encoding, and distress-reference construction |
| **Signal Construction** | Analytical, behavioral, temporal, statistical, structural, and model-based signals |
| **Multi-Objective Fusion** | Baseline-only weight optimization via L-BFGS with empirical-Bayes cohort shrinkage |
| **Decision Output** | Composite score, Baseline-calibrated risk tiers, operating threshold, and exact signal-level attribution |
| **External Validation** | Prospective linkage to hospital bankruptcy filings and Medicare enforcement actions |

---

## Signal Architecture

MOSAIC integrates **K = 108 signals** across five dimensions that are summarized operationally within three broader channels.

### Analytical Channel

The analytical channel includes statistical and temporal signals such as peer-relative deviations, residual behavior, longitudinal changes, breaks, persistence, and cohort-normalized patterns.

### Behavioral Channel

Behavioral signals capture reporting patterns such as bunching, repeated values, persistence, coordination, tradeoffs, and strategically unusual combinations of reported values.

### Model-Based Channel

Three machine-learning outputs enter the primary MOSAIC signal catalog:

- **Isolation Forest (IF)** — recursive-partitioning anomaly score
- **Variational Autoencoder (VAE)** — nonlinear reconstruction-error score
- **Transformer Masked Autoencoder** — sequence-aware reconstruction score

These model-based signals collectively receive less than 1% of the final global MOSAIC weight, indicating that the primary ranking is driven mainly by behavioral and temporal information rather than latent reconstruction error.

### Global Weight Distribution

| Signal Dimension | Share of Global Weight |
|------------------|------------------------|
| Behavioral | **56.73%** |
| Temporal | **35.03%** |
| Statistical | **7.37%** |
| Model-based | **0.87%** |
| Structural | **<0.01%** |

---

## Weak Supervision: Financial-Distress Reference

MOSAIC uses a parsimonious financial-distress reference as weak supervision rather than as a labeled outcome. The reference averages three logistic proximity scores:

```text
a_CR   = sigmoid[-5 × (CR − 1)]
a_OM   = sigmoid[-30 × OM]
a_DCOH = sigmoid[-0.5 × (DCOH − 15)]

d_it = (a_CR + a_OM + a_DCOH) / 3
```

The anchor variables themselves are excluded from the primary signal catalog. MOSAIC-X additionally removes 22 operating-margin-adjacent signal families, retaining 86 signals, to test whether indirect overlap with the reference drives the primary findings.

---

## Multi-Objective Optimization

For each peer cohort, MOSAIC estimates nonnegative weights on the probability simplex using Baseline observations only.

- **Objective:** Align the composite score with the financial-distress reference while encouraging monotonic separation and stable signal aggregation
- **Constraint:** Σ w_k = 1 and w_k ≥ 0
- **Parameterization:** Softmax transformation of unconstrained optimization parameters
- **Optimization:** Multi-start L-BFGS
- **Cohort stabilization:** Empirical-Bayes shrinkage toward the global weight vector
- **Temporal discipline:** Baseline-estimated aggregation weights are frozen and applied unchanged to later regimes

The MOSAIC score is:

```text
F_it = Σ_k w_k^(c(i)) × S̃_it^(k)
```

Exact case-level attribution is retained through:

```text
C_itk = w_k^(c(i)) × S̃_it^(k)
F_it  = Σ_k C_itk
```

This decomposition is exact and additive. It describes score construction and should not be interpreted causally.

---

## Time Regimes

| Regime | Years | Records | Role |
|--------|-------|---------|------|
| Transition | FY2011 | 6,152 | Pre-Baseline alignment |
| Baseline | FY2012–2018 | 43,529 | Weight estimation and model fitting |
| COVID Shock | FY2019–2021 | 18,241 | Structural break and out-of-sample deployment |
| Recovery | FY2022–2024 | 15,590 | Post-shock deployment |

The Baseline period is the only regime used to estimate MOSAIC aggregation weights. Those weights are carried forward without re-estimation.

---

## Primary Performance Results

### Internal Concordance and Generalization

| Regime | MOSAIC ρ_S |
|--------|------------|
| Baseline | **0.5986** |
| Transition | **0.5675** |
| COVID Shock | **0.5170** |
| Recovery | **0.5271** |

The decline after Baseline is expected under genuine temporal holdout, but concordance remains stable across the pandemic shock and recovery periods.

### Prospective External Validation

MOSAIC scores were linked prospectively to two independent adverse-event datasets using prior-period report information.

| Outcome | Matched Events | High + Critical | Null | p-value | Critical | Null | p-value |
|---------|----------------|-----------------|------|---------|----------|------|---------|
| Hospital bankruptcy | 86 | **76.7%** | 40% | **<0.001** | **53.5%** | 20% | **<0.001** |
| Medicare enforcement | 30 | **60.0%** | 40% | **0.021** | **43.3%** | 20% | **0.003** |

Because the intended use is prioritization under limited review capacity, the **Critical tier** is the principal operational outcome. MOSAIC produced the highest Critical-tier bankruptcy concentration among the evaluated comparators and tied for the highest Critical-tier enforcement concentration while maintaining complete event coverage.

---

## Comparator Models

The revised evaluation includes supervised, one-class, anomaly-detection, and established financial-distress comparators.

| Model | Bankruptcy Coverage | Bankruptcy H+C | Bankruptcy Critical | Enforcement Coverage | Enforcement H+C | Enforcement Critical |
|-------|---------------------|----------------|---------------------|----------------------|-----------------|----------------------|
| **MOSAIC** | **100.0%** | **76.7%** | **53.5%** | **100.0%** | **60.0%** | **43.3%** |
| MOSAIC-X | 100.0% | 69.8% | 48.8% | 100.0% | 60.0% | 43.3% |
| GBM | 100.0% | 79.1% | 46.5% | 100.0% | 63.3% | 20.0% |
| GBM-X | 100.0% | 66.3% | 39.5% | 100.0% | 63.3% | 30.0% |
| DeepSVDD | 100.0% | 61.6% | 39.5% | 100.0% | 56.7% | 43.3% |
| Ohlson | 96.5% | 68.7% | 44.6% | 83.3% | 56.0% | 24.0% |
| Zmijewski | 96.5% | 67.5% | 53.0% | 83.3% | 44.0% | 24.0% |
| BRKFSST | 95.3% | 53.7% | 20.7% | 93.3% | 50.0% | 17.9% |

GBM produced a small descriptive advantage in broad High-or-Critical capture, but MOSAIC concentrated more bankruptcy and enforcement events in the Critical tier. Paired bankruptcy comparisons showed no evidence of a GBM advantage in High-or-Critical capture, while MOSAIC significantly exceeded DeepSVDD on that endpoint.

DeepSVDD remains an independent nonlinear one-class comparator and is not included as an additional MOSAIC input.

---

## Robustness and Ablation Results

### Signal-Exclusion Robustness

MOSAIC-X excludes 22 operating-margin-adjacent signals that account for approximately 25.04% of the primary global weight.

| Specification | Global ρ_S | Bankruptcy H+C | Bankruptcy Critical | Enforcement H+C | Enforcement Critical |
|---------------|------------|----------------|---------------------|-----------------|----------------------|
| **MOSAIC** | **0.5622** | **76.7%** | **53.5%** | **60.0%** | **43.3%** |
| MOSAIC-X | 0.5385 | 69.8% | 48.8% | 60.0% | 43.3% |
| Minus behavioral signals | 0.3357 | 65.1% | 45.3% | 53.3% | 36.7% |
| Minus analytical signals | 0.5248 | 72.1% | 54.7% | 66.7% | 46.7% |
| Minus model-based signals | 0.5499 | 73.3% | 54.7% | 53.3% | 50.0% |

Removing behavioral signals produces the largest deterioration in internal concordance and broad external capture. Outcome-specific rankings differ across endpoints, so global weights should not be interpreted as universal outcome-specific importance measures.

---

## Exact Case-Level Attribution

The revised analysis includes a visual decomposition of representative Critical-tier pre-bankruptcy report records. For each case, the displayed signal contributions and the combined contribution of all remaining signals sum exactly to the stored MOSAIC score.

The figure is generated directly from the fitted MOSAIC objects as a native PGFPlots/TikZ file:

```text
revision_case_interpretability.tex
```

The corresponding audit table records the normalized signal value, cohort-specific weight, contribution, rank, and exact reconstruction error for every signal included in each selected case.

---

## Model Variants

| Variant | K | Description |
|---------|---|-------------|
| **MOSAIC** | 108 | Primary specification |
| **MOSAIC-X** | 86 | Excludes 22 operating-margin-adjacent signals |

MOSAIC-F has been removed from the revised manuscript. DeepSVDD, GBM, GBM-X, Ohlson, Zmijewski, and BRKFSST are external comparators rather than MOSAIC components.

---

## Supplementary Materials

The revised manuscript retains only the evidence required to understand and evaluate the primary claims. Detailed reproducibility, sensitivity, and audit material is organized into seven online supplements.

### Supplement S1: Complete Signal and Variable Catalog

- Full 108-signal catalog
- Source variables and HCRIS worksheet mappings
- Signal dimensions and polarity conventions
- Coverage and missingness summaries
- Baseline-derived encoding rules

### Supplement S2: Complete MOSAIC Mathematical Specification

- Sets, indices, and parameters
- Global and cohort-specific optimization programs
- Probability-simplex constraints
- Empirical-Bayes shrinkage
- Threshold and tier definitions
- Exact attribution identities

### Supplement S3: Computational and Experimental Settings

- Software and package versions
- Hardware and GPU environment
- Random seeds
- L-BFGS and optimization settings
- IF, VAE, Transformer, GBM, and DeepSVDD specifications
- Runtime and convergence diagnostics

### Supplement S4: Robustness and Sensitivity Analyses

- Anchor-coefficient sensitivity
- Pooling and shrinkage sensitivity
- Separation-penalty sensitivity
- Threshold and FPR sensitivity
- MOSAIC-X diagnostics
- Signal-class ablations
- Restart stability and regime-specific checks

### Supplement S5: Extended Comparator and External-Validation Results

- Complete model-by-model capture tables
- Available-score and common-event denominators
- Paired exact tests
- Coverage audits
- Confidence intervals
- Expanded bankruptcy and enforcement diagnostics

### Supplement S6: Case-Level Attribution Audit

- Prespecified case-selection rule
- All 108 contributions for each selected case
- Cohort-specific weights
- Normalized signal values
- Exact reconstruction checks
- Anonymized case-level audit output

### Supplement S7: Operational Deployment Notes

- Baseline-calibrated tier interpretation
- Review-prioritization workflow
- Monitoring and governance recommendations
- Implementation safeguards
- Limits on substantive interpretation

---

## Operational Use

MOSAIC scores are **ordered review-priority scores**, not event probabilities. Operational interpretation should focus on:

1. the assigned risk tier;
2. rank within the relevant comparison structure;
3. exact signal-level contributions;
4. review capacity and investigative cost.

The framework is intended to narrow a large administrative universe into a smaller, interpretable set of report records deserving closer examination. It should supplement, not replace, established financial-distress models, anomaly detectors, regulatory rules, and expert judgment.

---

## Data Source

**CMS Hospital Cost Report Information System (HCRIS) — HOSP10 Annual Releases**  
https://www.cms.gov

| File | Description |
|------|-------------|
| `RPT` | Report metadata and report-record identifiers |
| `NMRC` | Numeric worksheet data |
| `ALPHA` | Alphanumeric worksheet data |

Financial and operational values are stored using worksheet-line-column coordinates. Variable mappings were validated across fiscal years to address schema changes in HCRIS releases. Raw CMS files are not distributed in this repository.

---

## Portability

The following MOSAIC components are broadly portable:

- heterogeneous signal fusion under simplex constraints;
- Baseline-only weight estimation and frozen deployment;
- empirical-Bayes cohort shrinkage;
- explicit handling of partial signal coverage;
- Baseline-calibrated review tiers;
- exact additive case-level attribution.

Components requiring domain-specific adaptation include:

- the signal catalog;
- the peer-cohort taxonomy;
- the weak supervisory reference;
- the external validation outcomes;
- operational tier thresholds and review capacity.

The contribution is therefore the domain-constrained fusion, temporal deployment, exact attribution, and prospective evaluation of heterogeneous signals in a complex administrative panel rather than a new general-purpose optimization algorithm.

---

## Repository Structure

```text
MOSAIC/
├── code/
│   ├── primary/
│   ├── comparators/
│   ├── validation/
│   └── figures/
├── data/
│   ├── raw/
│   ├── intermediate/
│   └── processed/
├── manuscript/
│   ├── main/
│   ├── figures/
│   └── tables/
├── supplement/
│   ├── S1_signal_catalog/
│   ├── S2_mathematical_specification/
│   ├── S3_computational_settings/
│   ├── S4_robustness_sensitivity/
│   ├── S5_extended_validation/
│   ├── S6_case_attribution/
│   └── S7_deployment_notes/
└── README.md
```

| Property | Value |
|----------|-------|
| Unit of observation | Distinct HCRIS report record (`RPT_REC_NUM`) |
| Years covered | FY2011–2024 |
| Observations | 83,512 |
| Primary signals | K = 108 |
| MOSAIC-X signals | K = 86 |
| Baseline observations | 43,529 |
| Bankruptcy events | 86 |
| Enforcement events | 30 |
| Processed format | Parquet |

---

## Reproducibility

The repository is organized to preserve the locked 83,512-record analytic panel, Baseline-only model fitting, frozen post-Baseline deployment, comparator independence, prospective event linkage, exact case-level attribution, and auditable supplementary outputs.

Stale analyses based on the superseded 86,120-row expanded frame, earlier 73-bankruptcy and 24-enforcement samples, and the removed MOSAIC-F specification are not part of the revised submission branch.

---

## Disclaimer

This repository processes publicly available CMS data and does not distribute raw HCRIS files. Provider and report identifiers are used only for panel alignment and external-event matching and are anonymized in reported outputs.

MOSAIC identifies report records associated with elevated prospective institutional risk. It does not establish fraud, reporting misconduct, regulatory noncompliance, or causal responsibility.
