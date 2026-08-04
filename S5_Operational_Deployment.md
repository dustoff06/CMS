# Supplement S5
## Operational Deployment Guide

## Operational Interpretation and Deployment

### Score interpretation

MOSAIC scores are ordered screening scores, not probabilities of bankruptcy, enforcement, fraud, or financial failure. The score is a convex combination of normalized signals. Its operational meaning is therefore relative rank and tier membership. The preferred manuscript-facing classification is the Baseline-calibrated five-tier structure: Low, Guarded, Elevated, High, and Critical.

### Review workflow

A capacity-constrained review program should first define a report budget rather than assume that every report above a fixed binary threshold can be investigated. Reports may then be sorted by MOSAIC score, with the five-tier labels used as an interpretable summary of position relative to the Baseline distribution. For each selected report, the exact contributions

```math
C_{itk}=w_k^{*(c(i))}\bar S_{it}^{(k)}
```

identify the signal patterns responsible for the ranking.

The binary threshold $`\tau^*=0.5136`$ is retained as an optional constrained-FPR operating point, but it is not interchangeable with the five-tier classification. The threshold is calibrated against the weak supervisory reference, while the five tiers partition the Baseline score distribution.

### Updating and recalibration

A future deployment update should version the signal definitions, encoding rules, normalization reference, model weights, and tier thresholds as one pipeline. Re-estimating only one component can change the meaning of the score. Because the empirical study freezes Baseline weights to evaluate temporal transfer, routine annual re-optimization should not be described as equivalent to the validated study design. Any updated model should be treated as a new version and evaluated on an untouched subsequent period.

The current research implementation computes percentile normalization within observed cohort–regime distributions. For real-time prospective use, a regulator would need to adopt a frozen historical reference, a rolling reference window, or another prespecified online normalization rule. This distinction should be resolved before operational deployment.

### Portability

The general architecture is portable to other administrative panels: partial-coverage encoding, direction alignment, simplex-constrained fusion, hierarchical cohort pooling, Baseline-only fitting, tier calibration, and exact linear attribution do not depend on HCRIS. Domain adaptation is nevertheless required for the signal catalog, peer-cohort definitions, supervisory reference, event-linkage rules, and prospective normalization policy. Portability should therefore be evaluated through a new domain-specific validation rather than assumed from the HCRIS results.
