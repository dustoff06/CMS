# Supplement S2
## Complete Variable Construction Reference

**Study:** *A Fusion Framework for Prospective Anomaly Prioritization in Complex Administrative Panel Data*  
**Author:** Lawrence Fulton · Boston College  
**Application:** Medicare Hospital Cost Report Information System (HCRIS), FY2011–2024  
**Analytic unit:** Distinct cost-report record identified by `RPT_REC_NUM`  
**Locked primary panel:** 83,512 report records  
**Primary MOSAIC specification:** 108 inputs  
**Related supplement:** [Supplement S1: Complete MOSAIC Signal Matrix](MOSAIC_Supplement_S1_Complete_Signal_Matrix.md)

---

## Purpose

This supplement provides the complete variable-construction reference for the MOSAIC analysis. It documents the engineered financial indicators, analytical statistics, behavioral features, model-derived scores, directional conventions, partial-coverage encoding, and final normalization steps used to construct the 108-input MOSAIC matrix.

The three financial-distress reference variables (current ratio, operating margin, and days cash on hand) are reported here because they define the weak supervisory reference. They are not direct inputs to the primary MOSAIC specification. Operating-margin-derived anomaly signals remain part of the primary catalog, while MOSAIC-X removes the operating-margin bunching and quadratic-trend families as a targeted overlap sensitivity analysis.

---

## 1. Data Harmonization and Analytic Unit

The source data consist of annual CMS HCRIS HOSP10 releases for FY2011–FY2024. Report-level files are harmonized across years despite changes in delimiters, schemas, and field layouts. The primary analytic unit is a distinct accepted cost-report record identified by `RPT_REC_NUM`.

The locked primary panel contains 83,512 report records:

| Regime | Fiscal years | Records | Role |
|---|---:|---:|---|
| Transition | 2011 | 6,152 | Pre-baseline transition |
| Baseline | 2012–2018 | 43,529 | Model estimation and calibration |
| COVID Shock | 2019–2021 | 18,241 | Temporal holdout |
| Recovery | 2022–2024 | 15,590 | Temporal holdout and prospective validation |
| **Total** | **2011–2024** | **83,512** |  |

Continuous source variables are winsorized at the 1st and 99th percentiles before downstream ratio, residual, and anomaly construction.

---

## 2. Engineered Financial Indicators

| Category | Indicator | Formula | Primary role |
|---|---|---|---|
| Operational efficiency | Revenue per patient day | $\mathrm{RevPerDay}_{it}=\dfrac{\mathrm{Net\ Patient\ Revenue}_{it}}{\mathrm{Patient\ Days}_{it}}$ | Signal construction |
| Operational efficiency | Expense per patient day | $\mathrm{ExpPerDay}_{it}=\dfrac{\mathrm{Total\ Operating\ Expenses}_{it}}{\mathrm{Patient\ Days}_{it}}$ | Signal construction |
| Liquidity | Days cash on hand | $\mathrm{DCOH}_{it}=\dfrac{\mathrm{Cash\ \&\ Investments}_{it}}{\mathrm{Total\ Operating\ Expenses}_{it}/365}$ | Distress reference only |
| Revenue integrity | Cost-to-charge ratio | $\mathrm{CCR}_{it}=\dfrac{\mathrm{Total\ Operating\ Expenses}_{it}}{\mathrm{Total\ Charges}_{it}}$ | Feature engineering |
| Structural transfers | Siphon ratio | $\mathrm{SR}_{it}=\dfrac{\mathrm{Home\ Office\ Adjustments}_{it}}{\mathrm{Total\ Operating\ Costs}_{it}}$ | Feature engineering |
| Solvency | Current ratio | $\mathrm{CR}_{it}=\dfrac{\mathrm{Current\ Assets}_{it}}{\mathrm{Current\ Liabilities}_{it}}$ | Distress reference only |
| Profitability | Return on assets | $\mathrm{ROA}_{it}=\dfrac{\mathrm{Net\ Income}_{it}}{\mathrm{Total\ Assets}_{it}}$ | Feature engineering |
| Profitability | Operating margin | $\mathrm{OM}_{it}=\dfrac{\mathrm{Net\ Patient\ Revenue}_{it}-\mathrm{Total\ Operating\ Expenses}_{it}}{\mathrm{Net\ Patient\ Revenue}_{it}}$ | Distress reference and source for specific anomaly signals |
| Cost structure | Labor intensity | $\mathrm{LaborIntensity}_{it}=\dfrac{\mathrm{Total\ Salaries\ \&\ Benefits}_{it}}{\mathrm{Total\ Operating\ Expenses}_{it}}$ | Feature engineering |
| Cash-flow dynamics | Accounts receivable days | $\mathrm{ARDays}_{it}=\dfrac{\mathrm{Accounts\ Receivable}_{it}}{\mathrm{Net\ Patient\ Revenue}_{it}/365}$ | Feature engineering |
| Cash-flow dynamics | Accounts payable days | $\mathrm{APDays}_{it}=\dfrac{\mathrm{Accounts\ Payable}_{it}}{\mathrm{Total\ Operating\ Expenses}_{it}/365}$ | Feature engineering |
| Payer mix | Charity-care share | $\mathrm{CharityShare}_{it}=\dfrac{\mathrm{Charity\ Care\ Cost}_{it}}{\mathrm{Total\ Operating\ Expenses}_{it}}$ | Feature engineering |

The labels above describe the conceptual HCRIS inputs. Exact worksheet, line, and column mappings are maintained in the repository extraction code because HCRIS field layouts vary across annual releases.

---

## 3. Directional Harmonization

For measure $m$, let $s_m\in\{-1,+1\}$ indicate whether higher raw values represent more favorable institutional performance. Peer-relative deviations are direction-adjusted as

$$
Z_{\mathrm{peer,signed},imt}=s_m Z_{\mathrm{peer},imt}.
$$

Positive values therefore represent unusually favorable reported performance after orientation. Several behavioral signals are constructed from the positive tail of these direction-adjusted deviations because unusually strong, persistent, or coordinated favorable reporting patterns can warrant review.

The final MOSAIC catalog separately stores whether higher values of each signal are treated as more anomalous. The active Transformer score is reversed during MOSAIC normalization because the current catalog sets `TRANSFORMER_HIGHER_WORSE = False`.

---

## 4. Analytical and Temporal Statistics

### 4.1 Facility self-baseline residual

For measure $m$, the facility-level residual deviation is

$$
Z_{\mathrm{resid},imt}
=
\frac{X_{imt}-\bar{X}_{im,8}}
{\widehat{\sigma}_{im,12}},
$$

where $\bar{X}_{im,8}$ is the trailing eight-period facility mean and $\widehat{\sigma}_{im,12}$ is the trailing twelve-period standard deviation.

### 4.2 Temporal acceleration

$$
Z_{\mathrm{resid\_delta},imt}
=
Z_{\mathrm{resid},imt}
-
Z_{\mathrm{resid},im,t-1}.
$$

This captures abrupt movement relative to the facility’s own recent baseline.

### 4.3 Cross-benchmark divergence

A general divergence statistic compares the direction-adjusted position of a report under two benchmark systems:

$$
Z_{\delta,imt}
=
\left|
Z_{\mathrm{peer},imt}
-
Z_{\mathrm{regional},imt}
\right|.
$$

In the executed catalog, `z_delta_mean` averages absolute divergence between peer-group and state-by-year robust z-scores across the core operational measures.

### 4.4 Robust relational Mahalanobis distance

For the relational vector containing net patient revenue, total operating expense, and total assets,

$$
D_{it}
=
\sqrt{
(\mathbf{X}_{it}-\boldsymbol{\mu}_{c})^\top
\boldsymbol{\Sigma}_{c}^{-1}
(\mathbf{X}_{it}-\boldsymbol{\mu}_{c})
}.
$$

The executed `mahal_relational_score` uses a minimum-covariance-determinant fit estimated from Baseline observations within peer group.

### 4.5 Cross-measure dispersion

$$
\sigma_{\mathrm{facility},it}
=
\operatorname{SD}_{m}
\left(
Z_{\mathrm{peer,signed},imt}
\right).
$$

This represents inconsistency across multiple reported performance measures within a report record.

### 4.6 Revenue-expense decoupling

$$
\mathrm{DCPL}_{it}
=
\frac{\mathrm{Revenue}_{it}-\mathrm{Revenue}_{i,t-1}}
{\mathrm{Revenue}_{i,t-1}}
-
\frac{\mathrm{Expense}_{it}-\mathrm{Expense}_{i,t-1}}
{\mathrm{Expense}_{i,t-1}}.
$$

The implementation uses guarded provider-level percentage changes to limit instability from small or invalid denominators.

### 4.7 Operating-margin curvature

For a six-report rolling window, a second-order polynomial is fitted to operating margin:

$$
\mathrm{OM}_{i,t-j}
=
\beta_{0i}
+
\beta_{1i}j
+
\beta_{2i}j^2
+
\varepsilon_{i,t-j},
\qquad j=0,\ldots,5.
$$

The signal `margin_curv6` is the estimated quadratic coefficient $\widehat{\beta}_{2i}$ and requires at least five usable observations.

### 4.8 Residual volatility

$$
\sigma^{\mathrm{resid},4}_{it}
=
\operatorname{SD}
\left(
Z_{\mathrm{resid},i,t-3},
\ldots,
Z_{\mathrm{resid},it}
\right).
$$

The executed `z_resid_roll_std4` uses a four-report rolling standard deviation with at least two observations.

### 4.9 Peer-year divergence

$$
\mathrm{PDIV}_{it}
=
\frac{1}{|M|}
\sum_{m\in M}
\left|
Z_{\mathrm{peer}\times\mathrm{year},imt}
\right|.
$$

The executed `peer_divergence_year` averages absolute robust peer-group-by-year z-scores across operating margin, revenue per patient day, expense per patient day, and days per discharge.

### 4.10 Peer-rank instability

$$
\mathrm{RINST}_{it}
=
\frac{1}{|M|}
\sum_{m\in M}
\left|
\mathrm{Rank}_{im,t}
-
\mathrm{Rank}_{im,t-1}
\right|.
$$

The executed `rank_instability_mean` averages absolute provider-level changes in peer-group-by-year percentile rank across the same core measures.

### 4.11 Benford first-digit divergence

For eligible peer-year cells, the observed first-digit distribution $p_d$ is compared with the Benford reference distribution $b_d=\log_{10}(1+1/d)$:

$$
\mathrm{BenfordKL}
=
\sum_{d=1}^{9}
p_d
\log\left(\frac{p_d}{b_d}\right).
$$

The executed `benford_anomaly_score` is computed for total operating expense only when a fine peer-group-by-year cell contains at least 200 usable values.

---

## 5. Behavioral Features

### 5.1 Favorable-deviation magnitude

$$
\mathrm{GoodZ}_{imt}
=
\max
\left(
0,
Z_{\mathrm{peer,signed},imt}
\right).
$$

This retains the positive tail of direction-adjusted peer deviations.

### 5.2 Robust peer score

$$
Z^{\mathrm{rob}}_{\mathrm{peer},imt}
=
\frac{
0.6745
\left(
X_{imt}
-
\operatorname{median}_{g}(X_{mt})
\right)
}{
\operatorname{MAD}_{g}(X_{mt})
}.
$$

### 5.3 Peer-weighted favorable-deviation score

$$
\mathrm{GoodZ}_{\mathrm{weighted},imt}
=
\frac{
\mathrm{GoodZ}_{imt}
}{
\sigma_{\mathrm{group},gmt}
}.
$$

The report-level `goodz_weighted_sum` is the sum of positive direction-adjusted peer robust z-scores across operating margin, revenue per patient day, expense per patient day, and days per discharge, weighted by inverse peer-group dispersion.

### 5.4 Operating-margin bunching

Let $\mathcal{T}=\{-0.05,-0.02,0,0.02,0.05,0.10\}$ denote the prespecified operating-margin thresholds. Then

$$
\mathrm{BUNCH}_{it}
=
\frac{
1
}{
\min_{\tau\in\mathcal{T}}
|\mathrm{OM}_{it}-\tau|
+
\varepsilon
}.
$$

Higher values indicate proximity to one of the reporting thresholds.

### 5.5 Coordinated improvement

$$
\mathrm{COORD}_{it}
=
\frac{1}{|M|}
\sum_{m\in M}
\mathbf{1}
\left[
Z_{\mathrm{resid\_delta},imt}>0
\right].
$$

The executed `pct_measures_improving` is the share of operating margin, revenue per patient day, expense per patient day, and days per discharge showing a positive provider-level residual change after direction harmonization.

### 5.6 Tradeoff signal

$$
\mathrm{TRADE}_{it}
=
\max
\left(
0,
\mathrm{COORD}^{\mathrm{targeted}}_{it}
-
\mathrm{COORD}^{\mathrm{untargeted}}_{it}
\right).
$$

The signal captures selective improvement among targeted measures relative to untargeted measures.

### 5.7 Consecutive elevated-pattern streak

The historical identifier `tslf` is retained for reproducibility, but the executed code does not calculate elapsed time since a prior event. It calculates a provider-level consecutive streak:

$$
\mathrm{TSLF}_{it}
=
\begin{cases}
\mathrm{TSLF}_{i,t-1}+1, & \text{if }\mathrm{TooGoodFlag}_{it}=1,\\
0, & \text{otherwise}.
\end{cases}
$$

The elevated flag is based on the summed positive direction-adjusted peer z-scores crossing the prespecified threshold.

### 5.8 Strategic Reporting Risk Index

Let $b_{j,it}\in[0,1]$ denote the percentile-ranked value of behavioral component $j$, with $J=9$. The executed SRRI is

$$
\mathrm{SRRI}_{it}
=
100
\left[
0.8
\left(
\frac{1}{J}
\sum_{j=1}^{J}
b_{j,it}
\right)
+
0.2
\left(
\frac{
b_{\mathrm{persist},it}
+
b_{\mathrm{coord},it}
}{4}
\right)
\right].
$$

The candidate components represent relational breaks, decoupling, coordinated improvement, bunching, persistence, digit anomaly, peer divergence, rank volatility, and trajectory jumps. The 0.8/0.2 split and divisor of 4 are fixed before MOSAIC weight estimation.

---

## 6. Model-Based Components

| Component | Input representation | Training scope | Output used by MOSAIC |
|---|---|---|---|
| Isolation Forest | Common 11-bin decile-encoded feature matrix | Baseline-only fit | `if_pct_decile` |
| Variational Autoencoder | Common 11-bin decile-encoded feature matrix | Baseline-oriented training | Raw score `vae_gaming_score` and percentile score `vae_pct` |
| Transformer masked autoencoder | Common 11-bin decile-encoded feature matrix | Baseline-oriented training | `transformer_pct` |

The common encoded feature matrix uses ten ordinal source-value bins plus one explicit missing-value bin. This representation allows non-reporting to remain visible rather than being erased by ordinary numeric imputation.

The VAE raw anomaly score combines reconstruction loss and a weighted Kullback-Leibler term:

$$
\mathrm{VAE}_{it}
=
\mathrm{BCE}_{it}
+
0.05\,\mathrm{KL}_{it}.
$$

The Transformer score is based on masked reconstruction binary cross-entropy and is converted to a panel percentile before MOSAIC ingestion.

---

## 7. Partial-Coverage Source Signals

Nine conceptual signals are partially observed before encoding:

| Conceptual source | Observed records | Missing records | Source coverage | Resulting MOSAIC inputs |
|---|---:|---:|---:|---:|
| `benford_anomaly_score` | 31,613 | 51,899 | 37.9% | 11 |
| `bunch_signal_operating_margin` | 79,013 | 4,499 | 94.6% | 11 |
| `goodz_weighted_sum` | 78,808 | 4,704 | 94.4% | 11 |
| `decoupling_index` | 73,097 | 10,415 | 87.5% | 11 |
| `margin_curv6` | 53,034 | 30,478 | 63.5% | 11 |
| `rank_instability_mean` | 75,384 | 8,128 | 90.3% | 11 |
| `peer_divergence_year` | 82,844 | 668 | 99.2% | 11 |
| `z_resid_roll_std4` | 47,174 | 36,338 | 56.5% | 11 |
| `z_delta_mean` | 78,808 | 4,704 | 94.4% | 11 |

Each source produces ten decile-membership indicators and one explicit missingness indicator. The resulting 99 encoded inputs have complete matrix coverage.

---

## 8. Decile Encoding

For each partial-coverage source signal $x$, interior decile boundaries are estimated from records satisfying `source_year <= 2018`. Let

$$
q_{0.1},q_{0.2},\ldots,q_{0.9}
$$

denote the nine estimated boundaries. An observed value activates exactly one of the indicators

```text
<signal>_d01, ..., <signal>_d10
```

and leaves `<signal>_missing = 0`. A missing value activates only

```text
<signal>_missing
```

while all ten decile indicators remain zero.

These indicators are categorical basis terms. MOSAIC does not impose a monotonic ordering across `_d01` through `_d10`; each bin receives its own nonnegative learned weight.

Numeric cut points should be archived in `mosaic_signal_decile_cutpoints.csv` after the final clean run.

---

## 9. MOSAIC-Level Normalization

After the 108-input catalog has been assembled, each input is direction-harmonized and percentile-normalized within peer cohort × time regime.

For signal $k$ in cohort-regime cell $(c,r)$,

$$
\widetilde{S}_{it}^{(k)}
=
\operatorname{PctRank}_{c,r}
\left(
S_{it}^{(k)}
\right).
$$

The implemented fallback sequence is:

1. peer cohort × regime rank when at least 30 usable observations are available;
2. peer cohort rank when the first cell is too small;
3. global rank when the cohort fallback is also unavailable.

Normalized values are clipped to $[0,1]$. Remaining missing normalized values are median-imputed before optimization and scoring. The cohort-specific weights are estimated from Baseline records and then applied unchanged to all regimes.

---

## 10. Distress Reference Construction

The weak supervisory reference is the mean of three logistic proximity scores:

$$
a^{CR}_{it}
=
\sigma
\left[
-5
\left(
\mathrm{CR}_{it}-1
\right)
\right],
$$

$$
a^{OM}_{it}
=
\sigma
\left(
-30\,\mathrm{OM}_{it}
\right),
$$

$$
a^{DCOH}_{it}
=
\sigma
\left[
-0.5
\left(
\mathrm{DCOH}_{it}-15
\right)
\right],
$$

and

$$
d_{it}
=
\frac{
a^{CR}_{it}
+
a^{OM}_{it}
+
a^{DCOH}_{it}
}{3}.
$$

Here $\sigma(u)=1/(1+e^{-u})$. The reference orients the Baseline aggregation weights but is not a fraud label, adjudicated audit finding, or direct MOSAIC input.

---

## 11. Relationship to MOSAIC-X

MOSAIC-X removes the following two conceptual source families before optimization:

- `bunch_signal_operating_margin`
- `margin_curv6`

Because each source is represented by ten decile indicators plus one missingness indicator, the exclusion removes 22 inputs and reduces the catalog from 108 to 86. All other preprocessing, normalization, weight estimation, shrinkage, scoring, and tier calibration steps are unchanged.

---

## 12. Reproducibility Files

The following repository artifacts should accompany this supplement:

| File | Purpose |
|---|---|
| `MOSAIC_Supplement_S1_Complete_Signal_Matrix.md` | Complete 108-input matrix |
| `mosaic_signal_decile_cutpoints.csv` | Numeric source-signal decile boundaries |
| `mosaic_primary_global_weights.csv` | Ordered global weight vector |
| `mosaic_primary_weight_matrix.csv` | Frozen cohort-specific weights |
| `mosaic_primary_df.parquet` | Report-level fitted outputs |
| `mosaic_primary_metadata.json` | Panel fingerprint and execution metadata |

The numeric decile cut points and model outputs should be regenerated together after any change to the source panel, signal catalog, execution order, preprocessing state, or random seed.
