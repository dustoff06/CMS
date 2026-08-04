# Supplement S3
## Robustness Analyses, Supervised Benchmarking, and Exact Attribution

### Common-panel design and evaluation rules

All robustness models were evaluated on the locked panel of 83,512 HCRIS report records in the same row order as the primary specification. MOSAIC-X retained 86 inputs after removing the 22 operating-margin bunching and six-period curvature indicators. The remaining ablations separately removed the behavioral family, the analytical family (statistical plus temporal inputs), or the three explicitly model-based percentile scores. Every variant was re-estimated using the same Baseline-only weighting architecture and was applied to the same Transition, COVID-Shock, and Recovery records.

The event-tier analysis uses model-specific Baseline score distributions. High plus Critical denotes the upper 40% of Baseline scores, while Critical denotes the upper 20%. The Baseline cut points are applied unchanged to matched pre-event records. One-sided exact binomial tests compare observed event capture with null probabilities of 0.40 and 0.20. The bankruptcy analysis contains 86 matched event rows, and the enforcement analysis contains 30. These event-row analyses are distinct from the review-budget analysis, which deduplicates repeated event links and evaluates 84 unique bankruptcy-positive reports and 30 enforcement-positive reports.

### Internal signal-family robustness

The table below reports alignment with the weak supervisory reference under signal exclusion. MOSAIC-X retained most of the primary internal alignment. Removing the behavioral family caused the largest decline in global concordance, within-peer-year rank concordance, fixed-effect concordance, and quartile separation. Removing the three explicit model-based outputs produced only a modest reduction, consistent with their small total weight in the primary solution.

**Table S3.1: Internal alignment under signal exclusion**

| Model | $N$ | $\rho_{\mathrm{global}}$ | $\rho_{\mathrm{peer-year\ rank}}$ | $\rho_{\mathrm{peer/year\ FE}}$ | Separation |
|---|---:|---:|---:|---:|---:|
| MOSAIC | 83,512 | 0.5579 | 0.5364 | 0.5790 | 0.4031 |
| MOSAIC-X | 83,512 | 0.5398 | 0.5197 | 0.5551 | 0.3892 |
| Minus behavioral | 83,512 | 0.3379 | 0.3152 | 0.3385 | 0.2449 |
| Minus analytical | 83,512 | 0.5284 | 0.5040 | 0.5486 | 0.3765 |
| Minus model-based | 83,512 | 0.5501 | 0.5263 | 0.5692 | 0.3931 |

*Note.* All variants use the identical 83,512-report panel. Analytical signals comprise the statistical and temporal implementation dimensions. The peer/year fixed-effect correlation is distinct from the within-peer-year rank correlation.

### External event concentration under signal exclusion

MOSAIC classified 63 of 86 bankruptcy event rows (73.3%) as High or Critical and 45 (52.3%) as Critical (Table S3.2). MOSAIC-X remained close at 61 (70.9%) and 41 (47.7%), respectively. Removing behavioral signals produced the largest decline in broad bankruptcy capture. Removing analytical signals yielded the highest bankruptcy capture among the ablations, but this does not imply that the reduced model is generally preferable because the primary specification estimates one common ranking against the prespecified supervisory reference rather than optimizing separately for each external outcome.

**Table S3.2: Bankruptcy event concentration under signal exclusion**

| Model | High + Critical, $n$ (%) | Exact $p$ | Critical, $n$ (%) | Exact $p$ |
|---|---:|---:|---:|---:|
| MOSAIC | 63 (73.3) | <0.0001 | 45 (52.3) | <0.0001 |
| MOSAIC-X | 61 (70.9) | <0.0001 | 41 (47.7) | <0.0001 |
| Minus behavioral | 56 (65.1) | <0.0001 | 38 (44.2) | <0.0001 |
| Minus analytical | 64 (74.4) | <0.0001 | 47 (54.7) | <0.0001 |
| Minus model-based | 59 (68.6) | <0.0001 | 44 (51.2) | <0.0001 |

*Note.* $N=86$ matched bankruptcy event rows with valid scores for every model. Exact $p$-values are one-sided binomial tests against null capture probabilities of 0.40 for High plus Critical and 0.20 for Critical.

For enforcement, MOSAIC captured 18 of 30 events (60.0%) in the High plus Critical tiers and 15 (50.0%) in the Critical tier (Table S3.3). MOSAIC-X preserved broad capture and classified 13 events (43.3%) as Critical. Removing analytical signals increased broad capture to 66.7% but reduced Critical capture to 33.3%, indicating that the analytical family contributed more to extreme-tail concentration than to inclusion in the upper 40%. Removing the model-based family had a comparatively modest effect.

**Table S3.3: Enforcement event concentration under signal exclusion**

| Model | High + Critical, $n$ (%) | Exact $p$ | Critical, $n$ (%) | Exact $p$ |
|---|---:|---:|---:|---:|
| MOSAIC | 18 (60.0) | 0.0212 | 15 (50.0) | 0.0002 |
| MOSAIC-X | 18 (60.0) | 0.0212 | 13 (43.3) | 0.0031 |
| Minus behavioral | 16 (53.3) | 0.0971 | 11 (36.7) | 0.0256 |
| Minus analytical | 20 (66.7) | 0.0029 | 10 (33.3) | 0.0611 |
| Minus model-based | 17 (56.7) | 0.0481 | 14 (46.7) | 0.0009 |

*Note.* $N=30$ matched enforcement event rows with valid scores for every model. Exact $p$-values are one-sided binomial tests against null capture probabilities of 0.40 for High plus Critical and 0.20 for Critical.

### Supervised gradient-boosting benchmark

The full gradient-boosting model was retained as a supervised challenger rather than as a MOSAIC component. It was fitted to the Baseline distress reference using 105 normalized inputs after excluding the three explicit model-based percentile outputs. Because GBM was directly trained on the reference used to fit MOSAIC's weights, internal anchor concordance is not an independent validation criterion. The comparison below therefore focuses on unique known-positive reports found under fixed review budgets.

The bankruptcy risk set contains 27,707 eligible reports and 84 unique positive reports. The enforcement risk set contains 18,228 eligible reports and 30 positive reports. At a 5% bankruptcy review budget, MOSAIC captured 15 positive reports and GBM captured 16. At 10%, MOSAIC captured 27 and GBM captured 32. The paired differences were not significant. At a 10% enforcement budget, MOSAIC captured six positive reports and GBM captured five, again with no paired difference.

**Table S3.4: MOSAIC and supervised GBM capture under fixed review budgets**

| Outcome | Budget | MOSAIC, $n/N$ (%) | GBM, $n/N$ (%) | MOSAIC only | GBM only | Paired $p$ |
|---|---|---:|---:|---:|---:|---:|
| Bankruptcy | 1% | 3/84 (3.6) | 4/84 (4.8) | 3 | 4 | 1.0000 |
| Bankruptcy | 5% | 15/84 (17.9) | 16/84 (19.0) | 7 | 8 | 1.0000 |
| Bankruptcy | 10% | 27/84 (32.1) | 32/84 (38.1) | 2 | 7 | 0.1797 |
| Enforcement | 1% | 0/30 (0.0) | 0/30 (0.0) | 0 | 0 | 1.0000 |
| Enforcement | 5% | 1/30 (3.3) | 2/30 (6.7) | 1 | 2 | 1.0000 |
| Enforcement | 10% | 6/30 (20.0) | 5/30 (16.7) | 4 | 3 | 1.0000 |

*Note.* Positive-report denominators are 84 for bankruptcy and 30 for enforcement. Paired $p$-values are two-sided exact binomial tests on discordant positive-report classifications, equivalent to exact McNemar tests. Event rows linked repeatedly to the same bankruptcy report are counted once in this analysis.

The supervised challenger was competitive for broad bankruptcy prioritization, but the paired results provide no evidence that it consistently outperformed MOSAIC. The enforcement analysis is underpowered because it contains only 30 known-positive reports.

### Exact case-level attribution

For report record $(i,t)$, the exact contribution of signal $k$ is

```math
C_{itk}
=
w_k^{*(c(i))}
\widetilde{S}_{it}^{(k)},
```

and the implemented score satisfies

```math
F_{it}
=
\sum_{k=1}^K C_{itk}.
```

The full reconstruction audit reproduced every stored primary MOSAIC score with a maximum absolute error of $9.196\times10^{-8}$. The final case-audit file contains all 108 contributions for each of three representative Critical-tier pre-bankruptcy reports. The cases are the records nearest the 25th, 50th, and 75th percentiles of the exact-linked Critical-event score distribution. For each case, the eight largest contributions and the combined remainder sum to the corresponding stored score (Table S3.5 and the accompanying figure).

**Table S3.5: Summary of representative exact-attribution cases**

| Case | Selection percentile | MOSAIC score | Top-eight contribution | Top-eight share (%) | All other signals |
|---|---:|---:|---:|---:|---:|
| Case A | 25th | 0.515070 | 0.196114 | 38.1 | 0.318956 |
| Case B | 50th | 0.517947 | 0.286052 | 55.2 | 0.231895 |
| Case C | 75th | 0.523280 | 0.194688 | 37.2 | 0.328592 |

> *[Insert `S3_case_interpretability_final` figure here — signal-level decomposition of the three cases above.]*

The attribution values explain score construction rather than causation. A large contribution means that a report has a high normalized value for a signal that also receives substantial weight in its peer cohort. It does not establish that the signal caused bankruptcy, enforcement action, financial distress, or reporting misconduct.

### Supplementary interpretation

The robustness analyses support four conclusions. First, the primary internal alignment is not dependent on operating-margin bunching and curvature, although those terms contribute to external event concentration. Second, behavioral signals provide the largest unique contribution to alignment with the supervisory reference, while analytical and model-based families affect broad and extreme-tail event capture differently. Third, the supervised GBM challenger does not significantly outperform MOSAIC under paired report-level review budgets. Fourth, the exact attribution audit is algebraically faithful to the implemented score and requires no post-hoc surrogate explanation.
