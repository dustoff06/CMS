# MOSAIC
## A Fusion Framework for Prospective, Forensic Anomaly Detection in Complex Administrative Panel Data

**Author:** Lawrence Fulton · Boston College  
**Application Domain:** U.S. Hospital Financial Data (CMS HCRIS, FY2011–2024)  
**Target Venue:** *Expert Systems with Applications*  
**Status:** Under Review

---

## Overview

**MOSAIC** (Multi-Objective Signal Aggregation for Interpretable Classification) is a weakly supervised forensic prioritization framework for complex longitudinal administrative panel data. It integrates heterogeneous anomaly signals—spanning analytical diagnostics, behavioral forensic indicators, and machine learning model outputs—into a unified, interpretable composite score through multi-objective optimization.

The framework is motivated by a central challenge in healthcare regulatory oversight: CMS and Medicare Administrative Contractors must prioritize limited investigative resources across tens of thousands of annual hospital cost report filings, yet existing audit selection processes rely largely on professional judgment rather than systematic data-driven anomaly detection. MOSAIC addresses this gap directly.

Applied to **83,512 hospital facility-years** from the Medicare HCRIS dataset (FY2011–2024), MOSAIC achieves:

| Metric | Value |
|--------|-------|
| Spearman concordance with distress anchor | **ρ_S = 0.5618** |
| Peer × year adjusted concordance | **ρ_S = 0.5796** |
| Quartile separation | **0.4076** |
| Best standalone benchmark (VAE) | ρ_S = 0.1173 |
| Transformer MAE (negative result) | ρ_S = −0.1189 |

Signal-exclusion robustness (MOSAIC-X, K=86) yields ρ_S = 0.5394, confirming that performance is not driven by anchor-adjacent variables or any single signal family.

---

## Forensic Framing

MOSAIC operationalizes a three-layer conceptual structure:

1. **Institutional Distress** — margin compression, quality deterioration, operational instability
2. **Reporting Anomaly** — structural irregularity, digit-pattern deviations, reconstruction failure
3. **Forensic Screening Priority** — the joint signal space integrating both layers into a composite risk rank

This framing does not assume that distress-linked anomalies are fraudulent or that reporting anomalies arise from distress. It distinguishes among institutional state, reporting behavior, and their joint contribution to audit prioritization.

---

## Framework Architecture

MOSAIC operates through six integrated pipeline stages:

| Stage | Description |
|-------|-------------|
| **Data Ingestion** | Raw HCRIS extraction, harmonization, and normalization |
| **Preprocessing** | Regime split, peer cohort assignment, decile encoding, distress anchor construction |
| **Signal Channel Construction** | Analytical, Behavioral, and Model-based signal families |
| **Multi-Objective Fusion** | Baseline-trained weight optimization via L-BFGS with empirical Bayes shrinkage |
| **Decision Output** | Auto-threshold τ*, five-tier risk stratification, per-facility score decomposition |
| **External Validation** | Prospective linkage to bankruptcy filings and Medicare enforcement actions |

---

## Signal Architecture

MOSAIC integrates **K = 108 signals** organized across three channels:

### Analytical Channel
Peer-normalized scores, facility-specific residual deviations, temporal dynamics, and statistical diagnostics. Residual standardization uses an 8-year rolling peer median and 12-year rolling standard deviation:

```
Z_resid,imt = (X_imt − X̄_im,8) / σ̂_im,12
```

### Behavioral Channel
The **Strategic Reporting Risk Index (SRRI)** aggregates J=9 base behavioral signals with persistence and coordination components:

```
SRRI_it = 100 × (0.8 × (1/J) Σ b_j,it + 0.2 × (b_persist + b_coord) / 4)
```

SRRI sensitivity analysis across nine alternative parameterizations yields rank correlations of 0.989–1.000, confirming robustness to reasonable perturbations.

### Model-Based Channel
Three machine learning detectors operating on an 11-bin decile-encoded feature matrix (10 ordinal bins + explicit missing-value bin):

- **Isolation Forest (IF)** — recursive partitioning anomaly scores
- **Variational Autoencoder (VAE)** — latent structural reconstruction error
- **Transformer Masked Autoencoder (MAE)** — included as a benchmark; exhibits *negative* concordance (ρ_S = −0.1189), a headline finding demonstrating that high-capacity reconstruction models may capture dominant data variation rather than distress-relevant structure

---

## Weak Supervision: Financial Distress Anchor

MOSAIC employs a parsimonious financial distress anchor as a weak supervisory reference—not a label. The anchor is constructed from three logistic proximity scores:

```
a_CR   = σ(−5 × (CR − 1))
a_OM   = σ(−30 × OM)
a_DCOH = σ(−0.5 × (DCOH − 15))

d_it = (a_CR + a_OM + a_DCOH) / 3
```

Centering values reflect standard financial benchmarks: current ratio = 1 (liquidity break-even), operating margin = 0, days cash on hand = 15. Anchor variables are excluded from the signal catalog and used solely to construct d_it. Single-component anchor variants (CR-only, OM-only, DCOH-only) yield rank correlations with the full-anchor MOSAIC scores exceeding 0.94, confirming that behavioral preferences are a property of the signal architecture, not the anchor specification.

---

## Multi-Objective Optimization

For each peer cohort c, MOSAIC solves a constrained optimization over Baseline observations only:

- **Objective:** Minimize composite loss combining Spearman concordance with d_it, cross-signal agreement (Kendall's W), and a monotonicity violation penalty
- **Constraint:** Weights on the probability simplex (Σ w_k = 1, w_k ≥ 0)
- **Parameterization:** Softmax transformation of unconstrained z_k, enabling gradient-based L-BFGS optimization
- **Shrinkage:** Empirical Bayes pooling toward global weights — α_c = n_c^Baseline / (n_c^Baseline + 50)
- **Frozen weights:** Cohort-level weights estimated on Baseline (FY2012–2018, n=43,529) are applied unchanged to all subsequent regimes

The MOSAIC composite score is:

```
F_it = Σ_k w_k^(c(i)) × S̃_it^(k)
```

where S̃_it^(k) ∈ [0,1] are percentile-normalized signals constructed within cohort × regime cells.

---

## Time Regimes

| Regime | Years | Role |
|--------|-------|------|
| Transition | FY2011 | Pre-panel alignment |
| Baseline | FY2012–2018 | Training period (n = 43,529) — all parameters estimated here |
| COVID Shock | FY2019–2021 | Structural break — fully out-of-sample |
| Recovery | FY2022–2024 | Post-shock normalization — fully out-of-sample |

Weights, normalization parameters, decile thresholds, and the auto-threshold τ* are all estimated exclusively on the Baseline regime and frozen for all subsequent periods. This eliminates temporal leakage and ensures that post-shock anomalies reflect genuine deviations from pre-shock structure.

---

## Performance Results

### Overall (N = 83,512 facility-years)

| Model | ρ_global | ρ_peer×yr | Separation |
|-------|----------|-----------|------------|
| Fixed-weight (0.4/0.4/0.2) | 0.0184*** | 0.0401*** | 0.0507 |
| Standalone VAE | 0.1173*** | 0.1145*** | 0.1156 |
| Standalone IF | 0.1104*** | 0.1592*** | 0.1223 |
| Transformer MAE | −0.1189*** | −0.0626*** | −0.0554 |
| **MOSAIC** | **0.5618***  | **0.5796*** | **0.4076** |

### Regime-Stratified Generalization

| Regime | Fixed-wt | VAE | IF | MOSAIC | MOSAIC − IF |
|--------|----------|-----|----|--------|-------------|
| Baseline | 0.0505 | 0.1322 | 0.1516 | **0.5991** | +0.4475 |
| Transition | 0.1652 | 0.1958 | 0.2013 | **0.5663** | +0.3649 |
| COVID Shock | −0.0254 | 0.0930 | 0.0917 | **0.5172** | +0.4255 |
| Recovery | −0.0134 | 0.1418 | 0.1061 | **0.5237** | +0.4176 |

MOSAIC strictly dominates all benchmarks in every regime despite weights frozen on pre-pandemic data.

### Score Decomposition

Across representative high-risk facility-years, dimensional contributions to the MOSAIC composite are approximately:

- Behavioral: 57–59%
- Analytical: 35–37%
- Model-based: 5–7%

Elevated risk is primarily driven by systematic reporting behavior patterns rather than latent structural reconstruction anomalies.

---

## External Validation: Prospective Design

MOSAIC outputs were linked to two independent outcome datasets using a **strictly prospective design**: prior-period (t−1) scores predict adverse events in the following period (t). Signal weights were estimated on FY2012–2018 and never updated.

- **Hospital bankruptcies:** 73 matched observations (2022–2025)
- **Medicare enforcement actions:** 24 matched observations (same period)

| Outcome | Top 2 quintiles (observed) | Top 2 quintiles (null) | p-value |
|---------|---------------------------|------------------------|---------|
| Bankruptcy | 53.4% | 40.0% | 0.014 |
| Enforcement | 54.2% | 40.0% | 0.114 |
| Critical tier — Bankruptcy | 34.2% | 20.0% | 0.003 |
| Critical tier — Enforcement | 37.5% | 20.0% | 0.036 |

Consistency across two independent adverse outcome types provides evidence that MOSAIC captures a persistent, cross-domain structure of institutional risk rather than overfitting to a single outcome.

---

## Model Variants

| Variant | K | Description |
|---------|---|-------------|
| **MOSAIC** | 108 | Primary specification |
| **MOSAIC-X** | 86 | Signal-exclusion robustness — anchor-adjacent families removed; ρ_S = 0.5394 |
| **MOSAIC-F** | 111 | Financially augmented — anchor variables added after percentile-ranking; ρ_S = 0.8087 (upper-bound benchmark) |

---

## Operational Deployment

MOSAIC scores are **ordered risk ranks**, not probabilities. The operationally meaningful quantities are: (a) rank within the relevant cohort–regime cell; (b) the assigned risk tier (Low / Guarded / Elevated / High / Critical); and (c) the per-dimension weight decomposition for investigation guidance.

**Recommended triage workflow:**
1. Apply auto-threshold τ* (α = 0.05 FPR) to identify flagged population (~10.4% of facility-years at current scale, n ≈ 8,689)
2. Sort flagged set by risk tier — 188 Critical facility-years represent highest-priority subset for immediate forensic review
3. Within tier, sort by cohort–regime rank rather than raw score to control for cohort-level distributional heterogeneity
4. Retrieve per-dimension score decomposition and top contributing signal identifiers to guide document requests and on-site review

Per-facility dimensional contributions are computable directly from the weight matrix and normalized signal matrix without additional estimation:

```
Contrib_it^(d) = Σ_{k ∈ K_d} w_k^*(c(i), r(t)) × S̃_it^(k)
```

---

## Data Source

**CMS Hospital Cost Report Information System (HCRIS) — HOSP10 Annual Releases**  
https://www.cms.gov

| File | Description |
|------|-------------|
| `RPT` | Report metadata and provider identifiers |
| `NMRC` | Numeric worksheet data |
| `ALPHA` | Alphanumeric worksheet data |

Financial values are stored using worksheet–line–column coordinates. All variable mappings were empirically validated across fiscal years due to undocumented schema drift in the HCRIS field definitions. Variables are winsorized at the 1% tails.

---

## Portability

The following MOSAIC components are portable without modification to other administrative datasets:
- 11-bin decile encoding scheme for partial-coverage signals
- Multi-objective optimization formulation
- Hierarchical empirical Bayes shrinkage estimator
- Auto-threshold (Bastian) rule

Components requiring domain-specific adaptation:
- Signal catalog (must be rebuilt from the target dataset's field structure)
- Peer cohort definitions (must reflect the institutional taxonomy of the new domain)
- Distress anchor (must be constructed from outcome-adjacent variables free of algebraic overlap with the signal catalog)

The framework has been extended without modification to skilled nursing facility (SNF) cost reports in ongoing work, suggesting broad portability within the CMS administrative data ecosystem.

---

## Repository Structure

```
HOSP10FYYYYY/
├── data
├── paper
└── code

processed_panel/
└── master_hospital_research_panel.parquet
```

| Property | Value |
|----------|-------|
| Unit of observation | Hospital–Year |
| Years covered | FY2011–2024 |
| Observations | 83,512 |
| Signals | K = 108 |
| Format | Parquet (Snappy compression) |

---

## Disclaimer

This repository processes publicly available CMS data and does not include raw HCRIS files. Raw data must be obtained directly from CMS. Provider identifiers are used solely for matching and are anonymized in all reported outputs.
