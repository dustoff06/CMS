# Supplement S1
## Complete MOSAIC Signal Matrix

**Study:** *A Fusion Framework for Prospective Anomaly Prioritization in Complex Administrative Panel Data*  
**Author:** Lawrence Fulton · Boston College  
**Application:** Medicare Hospital Cost Report Information System (HCRIS), FY2011–2024  
**Analytic unit:** Distinct cost-report record identified by `RPT_REC_NUM`  
**Locked primary panel:** 83,512 report records  
**Primary MOSAIC matrix:** 108 inputs

---

## Purpose

This supplement documents the complete input matrix used by the primary MOSAIC specification. It reports every executed signal identifier, its catalog dimension, its manuscript-level channel, its conceptual source, its representation after preprocessing, its directional treatment, and its coverage. The three financial-distress reference variables (current ratio, operating margin, and days cash on hand) are not MOSAIC inputs and are used only to construct the weak supervisory reference.

The executed catalog contains five stored dimensions that map to the three channels used in the manuscript. Statistical and temporal signals form the analytical channel. Behavioral signals form the behavioral channel. Structural and explicitly model-based scores form the model-based channel.

---

## Matrix Composition

| Stored dimension | Number of inputs | Manuscript channel |
|---|---:|---|
| `behavioral` | 26 | Behavioral |
| `temporal` | 66 | Analytical |
| `statistical` | 12 | Analytical |
| `model_based` | 3 | Model-based |
| `structural` | 1 | Model-based |
| **Total** | **108** |  |

The matrix consists of six fully observed continuous forensic signals, nine partially observed conceptual signals expanded into 99 binary inputs, and three explicit learned-detector scores. Each partially observed source produces ten Baseline-derived decile-membership indicators plus one explicit missingness indicator. Thus, 6 + 99 + 3 = 108 inputs.

---

## Conceptual Signal Sources and Executed Construction

| Conceptual source               | Plain name                                      | Catalog dimension   | Paper channel   | Source coverage   | Resulting MOSAIC inputs                        | Executed construction                                                                                                                                                                                                                                            |
|:--------------------------------|:------------------------------------------------|:--------------------|:----------------|:------------------|:-----------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `vae_gaming_score`              | VAE raw anomaly score                           | structural          | Model-based     | 100.0%            | 1 continuous input                             | Per-record variational-autoencoder score: reconstruction BCE plus 0.05 times the per-sample KL term.                                                                                                                                                             |
| `mahal_relational_score`        | Robust relational Mahalanobis distance          | statistical         | Analytical      | 100.0%            | 1 continuous input                             | Minimum-covariance-determinant Mahalanobis distance for net patient revenue, total operating expense, and total assets; preprocessing and covariance fit within peer group using Baseline records.                                                               |
| `benford_anomaly_score`         | Benford first-digit divergence                  | statistical         | Analytical      | 37.9%             | 10 decile indicators + 1 missingness indicator | Kullback-Leibler divergence between observed and Benford first-digit distributions for total operating expense within fine peer group by fiscal year; computed only when at least 200 usable values are available.                                               |
| `SRRI`                          | Strategic Reporting Risk Index                  | behavioral          | Behavioral      | 100.0%            | 1 continuous input                             | 100 × [0.8 × mean component percentile + 0.2 × persistence/coordination bonus]. Candidate components include relational break, decoupling, coordinated improvement, bunching, persistence, digit anomaly, peer divergence, rank volatility, and trajectory jump. |
| `bunch_signal_operating_margin` | Operating-margin bunching                       | behavioral          | Behavioral      | 94.6%             | 10 decile indicators + 1 missingness indicator | Inverse distance from operating margin to the thresholds −0.05, −0.02, 0, 0.02, 0.05, and 0.10, with a small numerical constant added to the denominator.                                                                                                        |
| `pct_measures_improving`        | Coordinated improvement share                   | behavioral          | Behavioral      | 100.0%            | 1 continuous input                             | Share of operating margin, revenue per patient day, expense per patient day, and days per discharge with a positive change in facility residual z-score after direction harmonization.                                                                           |
| `tslf`                          | Consecutive elevated-performance streak         | behavioral          | Behavioral      | 100.0%            | 1 continuous input                             | Provider-level running streak of consecutive records for which the summed positive direction-adjusted peer z-scores exceed 1.0. Despite the historical identifier, the executed code is a streak length rather than elapsed time since the last event.           |
| `tradeoff_signal`               | Targeted-versus-untargeted improvement tradeoff | behavioral          | Behavioral      | 100.0%            | 1 continuous input                             | max(0, targeted improvement share − untargeted improvement share), where targeted and untargeted financial/operational measures are evaluated from provider-level changes.                                                                                       |
| `goodz_weighted_sum`            | Peer-weighted favorable-deviation magnitude     | behavioral          | Behavioral      | 94.4%             | 10 decile indicators + 1 missingness indicator | Sum of positive direction-adjusted peer robust z-scores for operating margin, revenue per patient day, expense per patient day, and days per discharge, each weighted by the inverse peer-group standard deviation.                                              |
| `decoupling_index`              | Revenue-expense growth decoupling               | temporal            | Analytical      | 87.5%             | 10 decile indicators + 1 missingness indicator | Provider-level guarded percentage change in net patient revenue minus the guarded percentage change in total operating expense.                                                                                                                                  |
| `margin_curv6`                  | Six-period operating-margin curvature           | temporal            | Analytical      | 63.5%             | 10 decile indicators + 1 missingness indicator | Quadratic coefficient from a rolling second-order fit to operating margin over six report periods, requiring at least five observations.                                                                                                                         |
| `rank_instability_mean`         | Mean peer-rank instability                      | temporal            | Analytical      | 90.3%             | 10 decile indicators + 1 missingness indicator | Mean absolute provider-level change in peer-group-by-year percentile rank across operating margin, revenue per patient day, expense per patient day, and days per discharge.                                                                                     |
| `peer_divergence_year`          | Peer-year divergence                            | temporal            | Analytical      | 99.2%             | 10 decile indicators + 1 missingness indicator | Mean absolute robust z-score within peer group by fiscal year across operating margin, revenue per patient day, expense per patient day, and days per discharge.                                                                                                 |
| `z_resid_roll_std4`             | Four-period residual volatility                 | temporal            | Analytical      | 56.5%             | 10 decile indicators + 1 missingness indicator | Rolling four-period standard deviation, with at least two observations, of the provider operating-margin residual z-score relative to its rolling mean and volatility.                                                                                           |
| `z_delta_mean`                  | Mean peer-state z-score divergence              | temporal            | Analytical      | 94.4%             | 10 decile indicators + 1 missingness indicator | Mean absolute difference between peer-group robust z-scores and state-by-year robust z-scores across operating margin, revenue per patient day, expense per patient day, and days per discharge.                                                                 |
| `if_pct_decile`                 | Isolation Forest percentile score               | model_based         | Model-based     | 100.0%            | 1 continuous input                             | Percentile-oriented Isolation Forest anomaly score from the common decile-encoded feature matrix; higher values are treated as more anomalous.                                                                                                                   |
| `vae_pct`                       | VAE percentile score                            | model_based         | Model-based     | 100.0%            | 1 continuous input                             | Panel percentile rank of the VAE per-record anomaly score; higher values are treated as more anomalous.                                                                                                                                                          |
| `transformer_pct`               | Transformer percentile score                    | model_based         | Model-based     | 100.0%            | 1 continuous input                             | Panel percentile rank of Transformer reconstruction BCE. In the active MOSAIC catalog, its direction is reversed during signal normalization because TRANSFORMER_HIGHER_WORSE is set to False.                                                                   |

---

## Complete 108-Input Matrix

|   No. | Signal identifier                       | Catalog dimension   | Paper channel   | Conceptual source               | Representation                                            | Direction in matrix                        | Source coverage   | Matrix coverage   |
|------:|:----------------------------------------|:--------------------|:----------------|:--------------------------------|:----------------------------------------------------------|:-------------------------------------------|:------------------|:------------------|
|     1 | `vae_gaming_score`                      | structural          | Model-based     | `vae_gaming_score`              | Continuous source signal                                  | Higher = more anomalous                    | 100.0%            | 100.0%            |
|     2 | `mahal_relational_score`                | statistical         | Analytical      | `mahal_relational_score`        | Continuous source signal                                  | Higher = more anomalous                    | 100.0%            | 100.0%            |
|     3 | `SRRI`                                  | behavioral          | Behavioral      | `SRRI`                          | Continuous source signal                                  | Higher = more anomalous                    | 100.0%            | 100.0%            |
|     4 | `pct_measures_improving`                | behavioral          | Behavioral      | `pct_measures_improving`        | Continuous source signal                                  | Higher = more anomalous                    | 100.0%            | 100.0%            |
|     5 | `tslf`                                  | behavioral          | Behavioral      | `tslf`                          | Continuous source signal                                  | Higher = more anomalous                    | 100.0%            | 100.0%            |
|     6 | `tradeoff_signal`                       | behavioral          | Behavioral      | `tradeoff_signal`               | Continuous source signal                                  | Higher = more anomalous                    | 100.0%            | 100.0%            |
|     7 | `benford_anomaly_score_d01`             | statistical         | Analytical      | `benford_anomaly_score`         | Baseline-derived decile 1 membership indicator            | 1 = membership in the stated source decile | 37.9%             | 100.0%            |
|     8 | `benford_anomaly_score_d02`             | statistical         | Analytical      | `benford_anomaly_score`         | Baseline-derived decile 2 membership indicator            | 1 = membership in the stated source decile | 37.9%             | 100.0%            |
|     9 | `benford_anomaly_score_d03`             | statistical         | Analytical      | `benford_anomaly_score`         | Baseline-derived decile 3 membership indicator            | 1 = membership in the stated source decile | 37.9%             | 100.0%            |
|    10 | `benford_anomaly_score_d04`             | statistical         | Analytical      | `benford_anomaly_score`         | Baseline-derived decile 4 membership indicator            | 1 = membership in the stated source decile | 37.9%             | 100.0%            |
|    11 | `benford_anomaly_score_d05`             | statistical         | Analytical      | `benford_anomaly_score`         | Baseline-derived decile 5 membership indicator            | 1 = membership in the stated source decile | 37.9%             | 100.0%            |
|    12 | `benford_anomaly_score_d06`             | statistical         | Analytical      | `benford_anomaly_score`         | Baseline-derived decile 6 membership indicator            | 1 = membership in the stated source decile | 37.9%             | 100.0%            |
|    13 | `benford_anomaly_score_d07`             | statistical         | Analytical      | `benford_anomaly_score`         | Baseline-derived decile 7 membership indicator            | 1 = membership in the stated source decile | 37.9%             | 100.0%            |
|    14 | `benford_anomaly_score_d08`             | statistical         | Analytical      | `benford_anomaly_score`         | Baseline-derived decile 8 membership indicator            | 1 = membership in the stated source decile | 37.9%             | 100.0%            |
|    15 | `benford_anomaly_score_d09`             | statistical         | Analytical      | `benford_anomaly_score`         | Baseline-derived decile 9 membership indicator            | 1 = membership in the stated source decile | 37.9%             | 100.0%            |
|    16 | `benford_anomaly_score_d10`             | statistical         | Analytical      | `benford_anomaly_score`         | Baseline-derived decile 10 membership indicator           | 1 = membership in the stated source decile | 37.9%             | 100.0%            |
|    17 | `benford_anomaly_score_missing`         | statistical         | Analytical      | `benford_anomaly_score`         | Explicit source-missingness indicator                     | 1 = source value missing                   | 37.9%             | 100.0%            |
|    18 | `bunch_signal_operating_margin_d01`     | behavioral          | Behavioral      | `bunch_signal_operating_margin` | Baseline-derived decile 1 membership indicator            | 1 = membership in the stated source decile | 94.6%             | 100.0%            |
|    19 | `bunch_signal_operating_margin_d02`     | behavioral          | Behavioral      | `bunch_signal_operating_margin` | Baseline-derived decile 2 membership indicator            | 1 = membership in the stated source decile | 94.6%             | 100.0%            |
|    20 | `bunch_signal_operating_margin_d03`     | behavioral          | Behavioral      | `bunch_signal_operating_margin` | Baseline-derived decile 3 membership indicator            | 1 = membership in the stated source decile | 94.6%             | 100.0%            |
|    21 | `bunch_signal_operating_margin_d04`     | behavioral          | Behavioral      | `bunch_signal_operating_margin` | Baseline-derived decile 4 membership indicator            | 1 = membership in the stated source decile | 94.6%             | 100.0%            |
|    22 | `bunch_signal_operating_margin_d05`     | behavioral          | Behavioral      | `bunch_signal_operating_margin` | Baseline-derived decile 5 membership indicator            | 1 = membership in the stated source decile | 94.6%             | 100.0%            |
|    23 | `bunch_signal_operating_margin_d06`     | behavioral          | Behavioral      | `bunch_signal_operating_margin` | Baseline-derived decile 6 membership indicator            | 1 = membership in the stated source decile | 94.6%             | 100.0%            |
|    24 | `bunch_signal_operating_margin_d07`     | behavioral          | Behavioral      | `bunch_signal_operating_margin` | Baseline-derived decile 7 membership indicator            | 1 = membership in the stated source decile | 94.6%             | 100.0%            |
|    25 | `bunch_signal_operating_margin_d08`     | behavioral          | Behavioral      | `bunch_signal_operating_margin` | Baseline-derived decile 8 membership indicator            | 1 = membership in the stated source decile | 94.6%             | 100.0%            |
|    26 | `bunch_signal_operating_margin_d09`     | behavioral          | Behavioral      | `bunch_signal_operating_margin` | Baseline-derived decile 9 membership indicator            | 1 = membership in the stated source decile | 94.6%             | 100.0%            |
|    27 | `bunch_signal_operating_margin_d10`     | behavioral          | Behavioral      | `bunch_signal_operating_margin` | Baseline-derived decile 10 membership indicator           | 1 = membership in the stated source decile | 94.6%             | 100.0%            |
|    28 | `bunch_signal_operating_margin_missing` | behavioral          | Behavioral      | `bunch_signal_operating_margin` | Explicit source-missingness indicator                     | 1 = source value missing                   | 94.6%             | 100.0%            |
|    29 | `goodz_weighted_sum_d01`                | behavioral          | Behavioral      | `goodz_weighted_sum`            | Baseline-derived decile 1 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    30 | `goodz_weighted_sum_d02`                | behavioral          | Behavioral      | `goodz_weighted_sum`            | Baseline-derived decile 2 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    31 | `goodz_weighted_sum_d03`                | behavioral          | Behavioral      | `goodz_weighted_sum`            | Baseline-derived decile 3 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    32 | `goodz_weighted_sum_d04`                | behavioral          | Behavioral      | `goodz_weighted_sum`            | Baseline-derived decile 4 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    33 | `goodz_weighted_sum_d05`                | behavioral          | Behavioral      | `goodz_weighted_sum`            | Baseline-derived decile 5 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    34 | `goodz_weighted_sum_d06`                | behavioral          | Behavioral      | `goodz_weighted_sum`            | Baseline-derived decile 6 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    35 | `goodz_weighted_sum_d07`                | behavioral          | Behavioral      | `goodz_weighted_sum`            | Baseline-derived decile 7 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    36 | `goodz_weighted_sum_d08`                | behavioral          | Behavioral      | `goodz_weighted_sum`            | Baseline-derived decile 8 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    37 | `goodz_weighted_sum_d09`                | behavioral          | Behavioral      | `goodz_weighted_sum`            | Baseline-derived decile 9 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    38 | `goodz_weighted_sum_d10`                | behavioral          | Behavioral      | `goodz_weighted_sum`            | Baseline-derived decile 10 membership indicator           | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    39 | `goodz_weighted_sum_missing`            | behavioral          | Behavioral      | `goodz_weighted_sum`            | Explicit source-missingness indicator                     | 1 = source value missing                   | 94.4%             | 100.0%            |
|    40 | `decoupling_index_d01`                  | temporal            | Analytical      | `decoupling_index`              | Baseline-derived decile 1 membership indicator            | 1 = membership in the stated source decile | 87.5%             | 100.0%            |
|    41 | `decoupling_index_d02`                  | temporal            | Analytical      | `decoupling_index`              | Baseline-derived decile 2 membership indicator            | 1 = membership in the stated source decile | 87.5%             | 100.0%            |
|    42 | `decoupling_index_d03`                  | temporal            | Analytical      | `decoupling_index`              | Baseline-derived decile 3 membership indicator            | 1 = membership in the stated source decile | 87.5%             | 100.0%            |
|    43 | `decoupling_index_d04`                  | temporal            | Analytical      | `decoupling_index`              | Baseline-derived decile 4 membership indicator            | 1 = membership in the stated source decile | 87.5%             | 100.0%            |
|    44 | `decoupling_index_d05`                  | temporal            | Analytical      | `decoupling_index`              | Baseline-derived decile 5 membership indicator            | 1 = membership in the stated source decile | 87.5%             | 100.0%            |
|    45 | `decoupling_index_d06`                  | temporal            | Analytical      | `decoupling_index`              | Baseline-derived decile 6 membership indicator            | 1 = membership in the stated source decile | 87.5%             | 100.0%            |
|    46 | `decoupling_index_d07`                  | temporal            | Analytical      | `decoupling_index`              | Baseline-derived decile 7 membership indicator            | 1 = membership in the stated source decile | 87.5%             | 100.0%            |
|    47 | `decoupling_index_d08`                  | temporal            | Analytical      | `decoupling_index`              | Baseline-derived decile 8 membership indicator            | 1 = membership in the stated source decile | 87.5%             | 100.0%            |
|    48 | `decoupling_index_d09`                  | temporal            | Analytical      | `decoupling_index`              | Baseline-derived decile 9 membership indicator            | 1 = membership in the stated source decile | 87.5%             | 100.0%            |
|    49 | `decoupling_index_d10`                  | temporal            | Analytical      | `decoupling_index`              | Baseline-derived decile 10 membership indicator           | 1 = membership in the stated source decile | 87.5%             | 100.0%            |
|    50 | `decoupling_index_missing`              | temporal            | Analytical      | `decoupling_index`              | Explicit source-missingness indicator                     | 1 = source value missing                   | 87.5%             | 100.0%            |
|    51 | `margin_curv6_d01`                      | temporal            | Analytical      | `margin_curv6`                  | Baseline-derived decile 1 membership indicator            | 1 = membership in the stated source decile | 63.5%             | 100.0%            |
|    52 | `margin_curv6_d02`                      | temporal            | Analytical      | `margin_curv6`                  | Baseline-derived decile 2 membership indicator            | 1 = membership in the stated source decile | 63.5%             | 100.0%            |
|    53 | `margin_curv6_d03`                      | temporal            | Analytical      | `margin_curv6`                  | Baseline-derived decile 3 membership indicator            | 1 = membership in the stated source decile | 63.5%             | 100.0%            |
|    54 | `margin_curv6_d04`                      | temporal            | Analytical      | `margin_curv6`                  | Baseline-derived decile 4 membership indicator            | 1 = membership in the stated source decile | 63.5%             | 100.0%            |
|    55 | `margin_curv6_d05`                      | temporal            | Analytical      | `margin_curv6`                  | Baseline-derived decile 5 membership indicator            | 1 = membership in the stated source decile | 63.5%             | 100.0%            |
|    56 | `margin_curv6_d06`                      | temporal            | Analytical      | `margin_curv6`                  | Baseline-derived decile 6 membership indicator            | 1 = membership in the stated source decile | 63.5%             | 100.0%            |
|    57 | `margin_curv6_d07`                      | temporal            | Analytical      | `margin_curv6`                  | Baseline-derived decile 7 membership indicator            | 1 = membership in the stated source decile | 63.5%             | 100.0%            |
|    58 | `margin_curv6_d08`                      | temporal            | Analytical      | `margin_curv6`                  | Baseline-derived decile 8 membership indicator            | 1 = membership in the stated source decile | 63.5%             | 100.0%            |
|    59 | `margin_curv6_d09`                      | temporal            | Analytical      | `margin_curv6`                  | Baseline-derived decile 9 membership indicator            | 1 = membership in the stated source decile | 63.5%             | 100.0%            |
|    60 | `margin_curv6_d10`                      | temporal            | Analytical      | `margin_curv6`                  | Baseline-derived decile 10 membership indicator           | 1 = membership in the stated source decile | 63.5%             | 100.0%            |
|    61 | `margin_curv6_missing`                  | temporal            | Analytical      | `margin_curv6`                  | Explicit source-missingness indicator                     | 1 = source value missing                   | 63.5%             | 100.0%            |
|    62 | `rank_instability_mean_d01`             | temporal            | Analytical      | `rank_instability_mean`         | Baseline-derived decile 1 membership indicator            | 1 = membership in the stated source decile | 90.3%             | 100.0%            |
|    63 | `rank_instability_mean_d02`             | temporal            | Analytical      | `rank_instability_mean`         | Baseline-derived decile 2 membership indicator            | 1 = membership in the stated source decile | 90.3%             | 100.0%            |
|    64 | `rank_instability_mean_d03`             | temporal            | Analytical      | `rank_instability_mean`         | Baseline-derived decile 3 membership indicator            | 1 = membership in the stated source decile | 90.3%             | 100.0%            |
|    65 | `rank_instability_mean_d04`             | temporal            | Analytical      | `rank_instability_mean`         | Baseline-derived decile 4 membership indicator            | 1 = membership in the stated source decile | 90.3%             | 100.0%            |
|    66 | `rank_instability_mean_d05`             | temporal            | Analytical      | `rank_instability_mean`         | Baseline-derived decile 5 membership indicator            | 1 = membership in the stated source decile | 90.3%             | 100.0%            |
|    67 | `rank_instability_mean_d06`             | temporal            | Analytical      | `rank_instability_mean`         | Baseline-derived decile 6 membership indicator            | 1 = membership in the stated source decile | 90.3%             | 100.0%            |
|    68 | `rank_instability_mean_d07`             | temporal            | Analytical      | `rank_instability_mean`         | Baseline-derived decile 7 membership indicator            | 1 = membership in the stated source decile | 90.3%             | 100.0%            |
|    69 | `rank_instability_mean_d08`             | temporal            | Analytical      | `rank_instability_mean`         | Baseline-derived decile 8 membership indicator            | 1 = membership in the stated source decile | 90.3%             | 100.0%            |
|    70 | `rank_instability_mean_d09`             | temporal            | Analytical      | `rank_instability_mean`         | Baseline-derived decile 9 membership indicator            | 1 = membership in the stated source decile | 90.3%             | 100.0%            |
|    71 | `rank_instability_mean_d10`             | temporal            | Analytical      | `rank_instability_mean`         | Baseline-derived decile 10 membership indicator           | 1 = membership in the stated source decile | 90.3%             | 100.0%            |
|    72 | `rank_instability_mean_missing`         | temporal            | Analytical      | `rank_instability_mean`         | Explicit source-missingness indicator                     | 1 = source value missing                   | 90.3%             | 100.0%            |
|    73 | `peer_divergence_year_d01`              | temporal            | Analytical      | `peer_divergence_year`          | Baseline-derived decile 1 membership indicator            | 1 = membership in the stated source decile | 99.2%             | 100.0%            |
|    74 | `peer_divergence_year_d02`              | temporal            | Analytical      | `peer_divergence_year`          | Baseline-derived decile 2 membership indicator            | 1 = membership in the stated source decile | 99.2%             | 100.0%            |
|    75 | `peer_divergence_year_d03`              | temporal            | Analytical      | `peer_divergence_year`          | Baseline-derived decile 3 membership indicator            | 1 = membership in the stated source decile | 99.2%             | 100.0%            |
|    76 | `peer_divergence_year_d04`              | temporal            | Analytical      | `peer_divergence_year`          | Baseline-derived decile 4 membership indicator            | 1 = membership in the stated source decile | 99.2%             | 100.0%            |
|    77 | `peer_divergence_year_d05`              | temporal            | Analytical      | `peer_divergence_year`          | Baseline-derived decile 5 membership indicator            | 1 = membership in the stated source decile | 99.2%             | 100.0%            |
|    78 | `peer_divergence_year_d06`              | temporal            | Analytical      | `peer_divergence_year`          | Baseline-derived decile 6 membership indicator            | 1 = membership in the stated source decile | 99.2%             | 100.0%            |
|    79 | `peer_divergence_year_d07`              | temporal            | Analytical      | `peer_divergence_year`          | Baseline-derived decile 7 membership indicator            | 1 = membership in the stated source decile | 99.2%             | 100.0%            |
|    80 | `peer_divergence_year_d08`              | temporal            | Analytical      | `peer_divergence_year`          | Baseline-derived decile 8 membership indicator            | 1 = membership in the stated source decile | 99.2%             | 100.0%            |
|    81 | `peer_divergence_year_d09`              | temporal            | Analytical      | `peer_divergence_year`          | Baseline-derived decile 9 membership indicator            | 1 = membership in the stated source decile | 99.2%             | 100.0%            |
|    82 | `peer_divergence_year_d10`              | temporal            | Analytical      | `peer_divergence_year`          | Baseline-derived decile 10 membership indicator           | 1 = membership in the stated source decile | 99.2%             | 100.0%            |
|    83 | `peer_divergence_year_missing`          | temporal            | Analytical      | `peer_divergence_year`          | Explicit source-missingness indicator                     | 1 = source value missing                   | 99.2%             | 100.0%            |
|    84 | `z_resid_roll_std4_d01`                 | temporal            | Analytical      | `z_resid_roll_std4`             | Baseline-derived decile 1 membership indicator            | 1 = membership in the stated source decile | 56.5%             | 100.0%            |
|    85 | `z_resid_roll_std4_d02`                 | temporal            | Analytical      | `z_resid_roll_std4`             | Baseline-derived decile 2 membership indicator            | 1 = membership in the stated source decile | 56.5%             | 100.0%            |
|    86 | `z_resid_roll_std4_d03`                 | temporal            | Analytical      | `z_resid_roll_std4`             | Baseline-derived decile 3 membership indicator            | 1 = membership in the stated source decile | 56.5%             | 100.0%            |
|    87 | `z_resid_roll_std4_d04`                 | temporal            | Analytical      | `z_resid_roll_std4`             | Baseline-derived decile 4 membership indicator            | 1 = membership in the stated source decile | 56.5%             | 100.0%            |
|    88 | `z_resid_roll_std4_d05`                 | temporal            | Analytical      | `z_resid_roll_std4`             | Baseline-derived decile 5 membership indicator            | 1 = membership in the stated source decile | 56.5%             | 100.0%            |
|    89 | `z_resid_roll_std4_d06`                 | temporal            | Analytical      | `z_resid_roll_std4`             | Baseline-derived decile 6 membership indicator            | 1 = membership in the stated source decile | 56.5%             | 100.0%            |
|    90 | `z_resid_roll_std4_d07`                 | temporal            | Analytical      | `z_resid_roll_std4`             | Baseline-derived decile 7 membership indicator            | 1 = membership in the stated source decile | 56.5%             | 100.0%            |
|    91 | `z_resid_roll_std4_d08`                 | temporal            | Analytical      | `z_resid_roll_std4`             | Baseline-derived decile 8 membership indicator            | 1 = membership in the stated source decile | 56.5%             | 100.0%            |
|    92 | `z_resid_roll_std4_d09`                 | temporal            | Analytical      | `z_resid_roll_std4`             | Baseline-derived decile 9 membership indicator            | 1 = membership in the stated source decile | 56.5%             | 100.0%            |
|    93 | `z_resid_roll_std4_d10`                 | temporal            | Analytical      | `z_resid_roll_std4`             | Baseline-derived decile 10 membership indicator           | 1 = membership in the stated source decile | 56.5%             | 100.0%            |
|    94 | `z_resid_roll_std4_missing`             | temporal            | Analytical      | `z_resid_roll_std4`             | Explicit source-missingness indicator                     | 1 = source value missing                   | 56.5%             | 100.0%            |
|    95 | `z_delta_mean_d01`                      | temporal            | Analytical      | `z_delta_mean`                  | Baseline-derived decile 1 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    96 | `z_delta_mean_d02`                      | temporal            | Analytical      | `z_delta_mean`                  | Baseline-derived decile 2 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    97 | `z_delta_mean_d03`                      | temporal            | Analytical      | `z_delta_mean`                  | Baseline-derived decile 3 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    98 | `z_delta_mean_d04`                      | temporal            | Analytical      | `z_delta_mean`                  | Baseline-derived decile 4 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|    99 | `z_delta_mean_d05`                      | temporal            | Analytical      | `z_delta_mean`                  | Baseline-derived decile 5 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|   100 | `z_delta_mean_d06`                      | temporal            | Analytical      | `z_delta_mean`                  | Baseline-derived decile 6 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|   101 | `z_delta_mean_d07`                      | temporal            | Analytical      | `z_delta_mean`                  | Baseline-derived decile 7 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|   102 | `z_delta_mean_d08`                      | temporal            | Analytical      | `z_delta_mean`                  | Baseline-derived decile 8 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|   103 | `z_delta_mean_d09`                      | temporal            | Analytical      | `z_delta_mean`                  | Baseline-derived decile 9 membership indicator            | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|   104 | `z_delta_mean_d10`                      | temporal            | Analytical      | `z_delta_mean`                  | Baseline-derived decile 10 membership indicator           | 1 = membership in the stated source decile | 94.4%             | 100.0%            |
|   105 | `z_delta_mean_missing`                  | temporal            | Analytical      | `z_delta_mean`                  | Explicit source-missingness indicator                     | 1 = source value missing                   | 94.4%             | 100.0%            |
|   106 | `if_pct_decile`                         | model_based         | Model-based     | `if_pct_decile`                 | Continuous percentile score                               | Higher = more anomalous                    | 100.0%            | 100.0%            |
|   107 | `vae_pct`                               | model_based         | Model-based     | `vae_pct`                       | Continuous percentile score                               | Higher = more anomalous                    | 100.0%            | 100.0%            |
|   108 | `transformer_pct`                       | model_based         | Model-based     | `transformer_pct`               | Continuous percentile score; direction reversed by MOSAIC | Reversed before MOSAIC normalization       | 100.0%            | 100.0%            |

---

## Encoding and Normalization Rules

For each of the nine partial-coverage source signals, interior decile boundaries are estimated from records with `source_year <= 2018`. An observed source value activates exactly one of the ten indicators (`_d01` through `_d10`) and leaves the `_missing` indicator equal to zero. A missing source value sets all ten decile indicators to zero and activates the `_missing` indicator.

All 108 inputs are subsequently percentile-normalized within peer cohort × time regime. When a cohort × regime cell has fewer than 30 usable observations for a signal, the code falls back first to cohort-level ranking and then to a global rank. The normalized values are clipped to [0, 1]. The Transformer signal is the only active input whose direction is explicitly reversed in the current primary catalog.

After normalization, remaining missing normalized values are median-imputed before optimization and scoring. Cohort-specific weights are estimated using Baseline records and then applied unchanged to Transition, COVID Shock, and Recovery records.

---

## Source Coverage for Decile-Encoded Signals

| Conceptual source | Observed records | Missing records | Source coverage | Expanded inputs |
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

---

## Interpretation Notes

1. Decile indicators are categorical basis terms. The `_d01` through `_d10` suffixes identify source-value bins, not an assumed monotonic risk ordering. Each indicator receives its own learned nonnegative weight.
2. Missingness is represented directly. A `_missing` input equals one when its conceptual source was unavailable and zero otherwise.
3. `vae_gaming_score` and `vae_pct` both appear in the executed primary catalog. The former is the raw VAE anomaly score and the latter is its panel percentile rank. This supplement reports the executed specification without collapsing either input.
4. The historical identifier `tslf` is retained for reproducibility. In the executed notebook, it is calculated as a consecutive streak of elevated `too_good_flag` observations rather than elapsed time since the last elevated observation.
5. `transformer_pct` is generated so that larger percentile values indicate greater reconstruction error, but the active primary MOSAIC catalog sets `TRANSFORMER_HIGHER_WORSE = False`; MOSAIC therefore reverses this signal during its own normalization.

---

## Numeric Decile Cut Points

The notebook creates the `decile_cutpoints` dictionary in memory when the partial-coverage signals are encoded. The current saved result files do not persist those arrays. Numeric cut points should be exported from the completed notebook run and archived with Supplement S2 so that they correspond exactly to the final executed panel.

```python
cutpoint_rows = []
for signal, cuts in decile_cutpoints.items():
    for boundary, value in enumerate(cuts, start=1):
        cutpoint_rows.append({
            "signal": signal,
            "boundary": boundary,
            "cutpoint": float(value),
        })

pd.DataFrame(cutpoint_rows).to_csv(
    "mosaic_signal_decile_cutpoints.csv",
    index=False,
)
```

---

## Reproducibility Files

The complete matrix order is recoverable from `mosaic_primary_global_weights.csv`. Cohort-specific frozen weights are stored in `mosaic_primary_weight_matrix.csv`, and the fitted report-level output is stored in `mosaic_primary_df.parquet`. Those result files should be regenerated together after any change to the signal catalog, panel lock, preprocessing state, or random seed.
