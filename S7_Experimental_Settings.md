# Supplement S7
## Experimental Settings and Computing Environment

### Purpose

This supplement documents the software environment, hardware configuration, numerical-precision settings, random-seeding procedures, and reproducibility controls underlying the primary MOSAIC estimation and all revision analyses, including MOSAIC-X, the signal-family ablations, GBM and GBM-X, DeepSVDD, and the established financial-distress benchmarks. Optimization hyperparameters and model-specific training settings are reported in Supplement S4 and Section 2.7 of the main text; the present supplement records the computational environment and execution controls needed to reproduce those analyses.

### Hardware

All GPU-dependent components — primary and cohort-level MOSAIC weight optimization, MOSAIC-X, the three signal-family ablations, DeepSVDD training and scoring, and GPU-accelerated XGBoost fitting — were executed on a single NVIDIA GeForce RTX 5080 (16 GB VRAM) via CUDA, under NVIDIA driver 591.86 on a Windows/WDDM system. GPU availability was enforced at the start of each dependent computation block; the pipeline raises an explicit error rather than silently falling back to CPU execution if CUDA is unavailable.

### Software

The pipeline was implemented in Python 3.14 (64-bit). Core dependencies, pinned to exact versions, include: PyTorch 2.10.0 (built against CUDA 12.8) for GPU tensor operations, DeepSVDD, VAE, and Transformer training; XGBoost 3.3.0 for GBM and GBM-X, using GPU histogram trees; scikit-learn 1.8.0 for imputation and scaling utilities; pandas 3.0.1 and NumPy 2.4.4 for data assembly and array operations; and SciPy 1.17.1 for Spearman concordance, hypergeometric, and binomial tests. The complete dependency manifest, including all transitive dependencies, is archived alongside the code repository as `requirements.txt`.

### Numerical precision

PyTorch was installed as the `cu128` build, targeting CUDA Toolkit 12.8, run under NVIDIA driver 591.86 (maximum supported CUDA 13.1) on a single NVIDIA GeForce RTX 5080 (16 GB VRAM, WDDM driver model, Windows). DeepSVDD training and scoring enabled TensorFloat-32 (TF32) matrix multiplication on CUDA (`torch.backends.cuda.matmul.allow_tf32 = True`, `torch.set_float32_matmul_precision("high")`). TF32 trades a small amount of mantissa precision for throughput and is a standard default on Ampere-and-later NVIDIA GPUs; it is not expected to materially affect DeepSVDD's role as an anomaly-detection comparator, but it means DeepSVDD internal-concordance and capture figures are not bit-for-bit reproducible across GPU architectures that do not support TF32. The primary MOSAIC weight-estimation optimizer (Appendix D.15) performs its L-BFGS and Adam-fallback computations without an explicit TF32 override.

### Random seeding and determinism

Seeding is applied at three points in the pipeline:

1. **VAE and Transformer training** use a fixed seed (`SEED = 42`) applied to the relevant framework random-number generators before training.

2. **Revision-era models** (MOSAIC-X, the three signal-family ablations, DeepSVDD) use a fixed seed (`RANDOM_STATE_REV = 2026`) applied via `torch.manual_seed`, `torch.cuda.manual_seed_all`, and `numpy.random.seed` at the start of each dependent block.

3. **Primary MOSAIC cohort-level optimization restarts** are seeded per cohort–regime cell using `numpy.random.default_rng(seed=int(abs(hash((cohort, regime))) % 10000))`. Reported results were confirmed stable across repeated runs in the study environment.

Reproducing a specific reported run requires fixing all three seeding mechanisms above in the same execution environment described in this supplement.

### Recommended reproducibility artifacts

- The pinned dependency manifest (`requirements.txt`), archived alongside the code repository.
- GPU driver and CUDA toolkit version, as reported above, alongside the GPU model.
- Retention of the panel fingerprint (`panel_index_sha256`) already computed and logged at the start of each revision block, which verifies row-set and row-order identity across runs independent of numerical reproducibility.
