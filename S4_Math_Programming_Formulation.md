# Supplement S4 Mathematical Programming Formulation

## D.1 Scope

This appendix gives the mathematical specification of the MOSAIC weight-estimation problem implemented in the locked common-panel analysis. The formulation separates three stages: estimation of a global Baseline weight vector, estimation and hierarchical pooling of Baseline cohort weight vectors, and application of the resulting cohort weights without re-estimation in every time regime. Signal construction, percentile normalization, and median imputation occur before the optimization and are treated here as fixed inputs.

The primary specification contains `K = 108` signals. MOSAIC-X solves the same problem after removing the 22 operating-margin bunching and curvature signals, leaving `K_X = 86`. The optimization architecture, parameter values, temporal training rule, and scoring equation are otherwise unchanged.

## D.2 Sets and indices

| Symbol | Definition |
|---|---|
| `i ∈ I` | Hospital or facility |
| `t ∈ T` | Fiscal year |
| `k ∈ K` | Anomaly signal |
| `c ∈ C` | Peer cohort |
| `r ∈ R` | Time regime |
| `B ⊂ I × T` | All Baseline observations |
| `B_c ⊂ B` | Baseline observations belonging to cohort `c` |
| `H_A` | High-distress observations in fitting set `A` |
| `L_A` | Low-distress observations in fitting set `A` |

The cohort assignment of hospital `i` is denoted by `c(i)`, and the regime assignment of year `t` is denoted by `r(t)`. In the empirical application, the peer cohorts are government, nonprofit, for-profit, other, and critical access hospital. The regimes are Transition, Baseline, COVID-Shock, and Recovery.

The Baseline fitting sets are

```math
B=\{(i,t)\in I\times T:r(t)=\mathrm{Baseline}\},
```

and

```math
B_c=\{(i,t)\in B:c(i)=c\}.
```

## D.3 Fixed inputs

For each observation `(i,t)` and signal `k`, let the direction-corrected, percentile-normalized signal be

```math
\widetilde{S}_{it}^{(k)}\in[0,1],
```

where larger values uniformly represent greater anomaly severity. After normalization, remaining missing values are replaced by the median of the corresponding normalized signal. The resulting fixed optimization input is denoted by

```math
\bar{S}_{it}^{(k)}.
```

The signal vector for observation `(i,t)` is

```math
\bar{\mathbf S}_{it}
=
\left(
\bar S_{it}^{(1)},
\ldots,
\bar S_{it}^{(K)}
\right)^{\mathsf T}.
```

The weak supervisory reference is the financial distress anchor `d_it`. It is constructed from current ratio (`CR_it`), operating margin (`OM_it`), and days cash on hand (`DCOH_it`) using fixed logistic proximity functions:

```math
a_{it}^{CR}
=
\frac{1}
{1+\exp\!\left(5(CR_{it}-1)\right)},
```

```math
a_{it}^{OM}
=
\frac{1}
{1+\exp\!\left(30\,OM_{it}\right)},
```

```math
a_{it}^{DCOH}
=
\frac{1}
{1+\exp\!\left(0.5(DCOH_{it}-15)\right)}.
```

The composite anchor is

```math
d_{it}
=
\frac{
a_{it}^{CR}
+
a_{it}^{OM}
+
a_{it}^{DCOH}
}{3},
\qquad
d_{it}\in[0,1].
```

When an anchor component is missing, that component is assigned the neutral value `0.5` before the three components are averaged. The raw anchor variables do not enter the primary MOSAIC signal catalog.

## D.4 Decision variables and simplex parameterization

The global decision vector is

```math
\mathbf w^{G}
=
\left(
w_1^{G},
\ldots,
w_K^{G}
\right)^{\mathsf T},
```

and the cohort-specific decision vector for cohort `c` is

```math
\mathbf w^{(c)}
=
\left(
w_1^{(c)},
\ldots,
w_K^{(c)}
\right)^{\mathsf T}.
```

Each vector lies on the probability simplex:

```math
\Delta^{K-1}
=
\left\{
\mathbf w\in\mathbb R^K:
w_k\geq 0\ \forall k,
\quad
\sum_{k=1}^{K}w_k=1
\right\}.
```

The GPU implementation enforces the simplex through an unconstrained softmax parameterization. For latent optimization variables `θ_k ∈ ℝ`,

```math
w_k(\boldsymbol\theta)
=
\frac{\exp(\theta_k)}
{\sum_{\ell=1}^{K}\exp(\theta_\ell)}.
```

This parameterization produces strictly positive weights in exact arithmetic, although weights can become numerically indistinguishable from zero.

## D.5 MOSAIC fused score

Given a feasible cohort weight vector, the MOSAIC score for observation `(i,t)` is

```math
F_{it}
=
\sum_{k=1}^{K}
w_k^{(c(i))}
\bar S_{it}^{(k)}
=
\bar{\mathbf S}_{it}^{\mathsf T}
\mathbf w^{(c(i))}.
```

The same cohort vector is used in every regime. Regime-specific variation in `F_it` therefore arises from the normalized signal values rather than from re-estimation of the weights.

For any fitting subset `A`, define the score vector

```math
\mathbf F_A(\mathbf w)
=
\left(
\bar{\mathbf S}_{it}^{\mathsf T}\mathbf w
\right)_{(i,t)\in A}.
```

## D.6 Differentiable rank concordance

The implemented objective uses a differentiable approximation to rank. For a vector `x = (x_1,\ldots,x_n)`, the soft rank of observation `j` is

```math
\mathrm{srank}_{\tau}(x_j)
=
\sum_{\ell=1}^{n}
\sigma\!\left(
\frac{x_j-x_\ell}{\tau}
\right),
```

where

```math
\sigma(z)=\frac{1}{1+\exp(-z)}
```

and the implemented rank temperature is

```math
\tau=0.01.
```

Let `R_τ(x)` denote the vector of soft ranks. The differentiable Spearman approximation used during estimation is the Pearson correlation between the soft ranks of the fused score and the distress anchor:

```math
\rho_{\tau,A}(\mathbf w)
=
\mathrm{Corr}
\left[
R_\tau\!\left(\mathbf F_A(\mathbf w)\right),
R_\tau\!\left(\mathbf d_A\right)
\right].
```

Because the optimizer minimizes the objective, the concordance contribution is `-ρ_{τ,A}(w)`.

## D.7 Directional monotonicity penalty

For a fitting subset `A`, define the high- and low-distress partitions as

```math
H_A
=
\{(i,t)\in A:d_{it}\geq 0.6\},
```

and

```math
L_A
=
\{(i,t)\in A:d_{it}\leq 0.3\}.
```

The score-separation statistic is

```math
\Delta_A(\mathbf w)
=
\frac{1}{|H_A|}
\sum_{(i,t)\in H_A}
F_{it}(\mathbf w)
-
\frac{1}{|L_A|}
\sum_{(i,t)\in L_A}
F_{it}(\mathbf w).
```

A negative value would imply that the low-distress group receives a higher mean MOSAIC score than the high-distress group. The implementation discourages that reversal through the hinge penalty

```math
P_{\mathrm{mono},A}(\mathbf w)
=
\gamma
\max\!\left\{
0,
-\Delta_A(\mathbf w)
\right\},
```

with

```math
\gamma=5.
```

The penalty is activated only when both `H_A` and `L_A` contain at least five observations. Directional ordering is therefore a soft penalty rather than a hard feasibility constraint.

## D.8 Cross-signal Kendall quantity

For fitting set `A` with `n_A` observations, let `r_{it}^{(k)}` be the soft rank of signal `k` within `A`. Define the summed signal rank for observation `(i,t)` as

```math
R_{it}
=
\sum_{k=1}^{K}
r_{it}^{(k)},
```

with mean

```math
\bar R_A
=
\frac{1}{n_A}
\sum_{(i,t)\in A}
R_{it}.
```

The implemented Kendall quantity is

```math
W_A
=
\frac{
12
\sum_{(i,t)\in A}
\left(R_{it}-\bar R_A\right)^2
}{
K^2
\left(n_A^3-n_A\right)
}.
```

The objective records the term

```math
-\eta W_A,
\qquad
\eta=0.25.
```

**Implementation identity.** `W_A` depends only on the fixed signal matrix for fitting set `A`. It does not depend on the decision vector `w`. Consequently, `-ηW_A` shifts the reported objective value but does not alter the minimizing weight vector. In the current implementation, cross-signal Kendall concordance is therefore a fitting-sample diagnostic offset rather than a weight-identifying objective component.

## D.9 Generic penalized objective

For fitting set `A`, optional reference vector `w^ref`, and pooling coefficient `λ`, define

```math
\mathcal L_A
\left(
\mathbf w;
\mathbf w^{ref},
\lambda
\right)
=
-\rho_{\tau,A}(\mathbf w)
-\eta W_A
+
P_{\mathrm{mono},A}(\mathbf w)
+
\lambda
\left\|
\mathbf w-\mathbf w^{ref}
\right\|_2^2.
```

When no reference vector is supplied, the final term is omitted. Because the Kendall quantity is constant with respect to `w`, the decision-equivalent objective is

```math
\mathcal L_A^{\mathrm{eff}}
\left(
\mathbf w;
\mathbf w^{ref},
\lambda
\right)
=
-\rho_{\tau,A}(\mathbf w)
+
P_{\mathrm{mono},A}(\mathbf w)
+
\lambda
\left\|
\mathbf w-\mathbf w^{ref}
\right\|_2^2.
```

Thus, the implemented weights are identified by distress-anchor rank concordance, the directional separation penalty, and (for cohort fits) hierarchical pooling.

## D.10 Global Baseline optimization

The global weight vector is estimated from all Baseline observations. The mathematical program is

```math
\widehat{\mathbf w}^{G}
\in
\underset{
\mathbf w\in\Delta^{K-1}
}{
\mathrm{argmin}
}
\left[
-\rho_{\tau,B}(\mathbf w)
-\eta W_B
+
P_{\mathrm{mono},B}(\mathbf w)
\right].
```

Equivalently, because `W_B` is constant with respect to the decision variables,

```math
\widehat{\mathbf w}^{G}
\in
\underset{
\mathbf w\in\Delta^{K-1}
}{
\mathrm{argmin}
}
\left[
-\rho_{\tau,B}(\mathbf w)
+
P_{\mathrm{mono},B}(\mathbf w)
\right].
```

No pooling or smoothness penalty is applied in the global problem.

## D.11 Cohort Baseline optimization

For cohort `c` with a sufficient Baseline sample, an unshrunk cohort vector is obtained from

```math
\widehat{\mathbf w}^{(c)}
\in
\underset{
\mathbf w\in\Delta^{K-1}
}{
\mathrm{argmin}
}
\left[
-\rho_{\tau,B_c}(\mathbf w)
-\eta W_{B_c}
+
P_{\mathrm{mono},B_c}(\mathbf w)
+
\lambda_p
\left\|
\mathbf w-\widehat{\mathbf w}^{G}
\right\|_2^2
\right],
```

where the implemented pooling coefficient is

```math
\lambda_p=0.15.
```

The decision-equivalent cohort problem is

```math
\widehat{\mathbf w}^{(c)}
\in
\underset{
\mathbf w\in\Delta^{K-1}
}{
\mathrm{argmin}
}
\left[
-\rho_{\tau,B_c}(\mathbf w)
+
P_{\mathrm{mono},B_c}(\mathbf w)
+
0.15
\left\|
\mathbf w-\widehat{\mathbf w}^{G}
\right\|_2^2
\right].
```

The cohort-specific program is solved only when

```math
n_c
=
|B_c|
\geq
n_{\min},
```

with

```math
n_{\min}=150.
```

When `n_c < 150`, the final cohort vector is set directly to the global vector.

## D.12 Post-optimization hierarchical shrinkage

For an estimated cohort vector, MOSAIC applies an additional sample-size-dependent shrinkage step. Let

```math
\alpha_c
=
\mathrm{clip}
\left(
\frac{n_c}{n_c+n_0},
0.1,
1
\right),
```

with prior sample size

```math
n_0=50.
```

The final deployed cohort weight vector is

```math
\mathbf w^{*(c)}
=
\alpha_c
\widehat{\mathbf w}^{(c)}
+
\left(1-\alpha_c\right)
\widehat{\mathbf w}^{G}.
```

The vector remains on the simplex because it is a convex combination of two simplex vectors. For cohorts failing the minimum-sample rule,

```math
\mathbf w^{*(c)}
=
\widehat{\mathbf w}^{G}.
```

The primary implementation therefore pools cohort weights twice: first through the quadratic penalty inside the cohort optimization and second through the post-estimation convex shrinkage rule.

## D.13 Frozen application across regimes

The final Baseline-estimated cohort vector is copied unchanged to every observed regime:

```math
\mathbf w^{*(c,r)}
=
\mathbf w^{*(c)}
\qquad
\forall c\in C,\ \forall r\in R.
```

The final score is consequently

```math
F_{it}
=
\bar{\mathbf S}_{it}^{\mathsf T}
\mathbf w^{*(c(i))}.
```

The current primary implementation does not estimate separate cohort-by-regime weight vectors. It also does not activate the generic temporal smoothness penalty available in the optimizer (`λ_s = 0` and no previous-regime reference vector is supplied). Temporal stability is imposed through exact freezing of the Baseline cohort weights, not through penalized smoothing between successive regime-specific solutions.

## D.14 Exact signal attribution

Because the final score is linear in the normalized, imputed signals, the exact contribution of signal `k` to observation `(i,t)` is

```math
C_{itk}
=
w_k^{*(c(i))}
\bar S_{it}^{(k)}.
```

The contributions reconstruct the score exactly:

```math
F_{it}
=
\sum_{k=1}^{K}
C_{itk}.
```

This identity supports case-level explanations without requiring a secondary surrogate or post hoc approximation model.

## D.15 Numerical solution

The global and cohort problems are nonlinear because the soft-rank correlation depends nonlinearly on the fused score. They are solved on the GPU using multi-start L-BFGS with a strong-Wolfe line search. The production settings are summarized below.

| Quantity | Implemented value |
|---|---:|
| Number of independent restarts | 10 |
| Maximum optimization sample | 3,000 observations |
| Subsampling seed | 42 |
| L-BFGS learning rate | 0.5 |
| L-BFGS maximum iterations | 150 |
| Line search | Strong Wolfe |
| Soft-rank temperature | 0.01 |
| Monotonicity coefficient | 5.0 |
| Kendall coefficient | 0.25 |
| Cohort pooling coefficient | 0.15 |
| Minimum Baseline cohort size | 150 |
| Shrinkage prior sample size | 50 |
| Temporal smoothness coefficient in the primary model | 0 |
| Adam fallback steps | 500 |
| Adam fallback learning rate | 0.05 |

If a fitting set contains more than 3,000 observations, a deterministic simple random sample of 3,000 is selected for optimization. For the global problem, every restart begins from an independently generated Dirichlet simplex vector. For a cohort problem, the first restart begins at the global solution and the remaining restarts begin from Dirichlet draws. The candidate producing the smallest objective value is retained. If L-BFGS fails or produces a nonfinite objective, the corresponding initialization is optimized using Adam as a fallback.

The objective reported by the optimizer is evaluated before the post-optimization sample-size shrinkage step. The final deployed cohort vector is the convexly shrunk vector defined in Section D.12.

## D.16 MOSAIC-X

Let `K_X ⊂ K` denote the restricted signal set obtained by removing the operating-margin bunching and operating-margin curvature families. The MOSAIC-X global and cohort problems are identical to the primary problems after replacing `K` by `K_X` and rebuilding the normalized signal matrix on the retained columns:

```math
|K|=108,
\qquad
|K_X|=86.
```

No other objective coefficient, fitting rule, minimum-sample requirement, shrinkage parameter, or temporal deployment rule changes.

## D.17 Compact implemented formulation

The complete primary estimator can be summarized as follows.

First, estimate the global Baseline vector:

```math
\widehat{\mathbf w}^{G}
\in
\underset{
\mathbf w\in\Delta^{K-1}
}{
\mathrm{argmin}
}
\left[
-\rho_{\tau,B}(\mathbf w)
+
5\max\{0,-\Delta_B(\mathbf w)\}
\right].
```

Second, for each cohort with at least 150 Baseline observations, estimate

```math
\widehat{\mathbf w}^{(c)}
\in
\underset{
\mathbf w\in\Delta^{K-1}
}{
\mathrm{argmin}
}
\left[
-\rho_{\tau,B_c}(\mathbf w)
+
5\max\{0,-\Delta_{B_c}(\mathbf w)\}
+
0.15
\left\|
\mathbf w-\widehat{\mathbf w}^{G}
\right\|_2^2
\right].
```

Third, apply sample-size shrinkage:

```math
\mathbf w^{*(c)}
=
\frac{n_c}{n_c+50}
\widehat{\mathbf w}^{(c)}
+
\frac{50}{n_c+50}
\widehat{\mathbf w}^{G},
```

subject to the implemented clipping rule for `α_c`. Cohorts with fewer than 150 Baseline observations receive the global vector.

Finally, freeze the resulting cohort vector and score every observation:

```math
F_{it}
=
\sum_{k=1}^{K}
w_k^{*(c(i))}
\bar S_{it}^{(k)}.
```

The Kendall term may be retained when reproducing the numerical objective value:

```math
-\frac{1}{4}W_A,
```

but it is omitted from the compact decision-equivalent formulation because it is constant with respect to the signal weights.
