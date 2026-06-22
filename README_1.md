# Clustering Analysis in Active Matter

Detecting and quantifying **vortex-centre clustering** in turbulence using spatial statistics — Voronoi tessellation, square-grid binning, K-means clustering, and nearest-neighbour distance — validated against synthetic ground truth and applied to a real 3D isotropic turbulence slice.

Scientific basis: Kashyap, Kiran & Gupta, *Emergence of Local Ordering and Mesoscale Giant Number Fluctuations in Active Turbulence* (arXiv:2507.04890v2), §S4.

## Overview

A 2D point pattern (e.g. turbulent vortex centres) can look "clustered" by eye, but turning that into a defensible, quantitative claim requires statistics. This project builds and validates a pipeline that does exactly that, using four independent estimators cross-checked against each other and against analytical / Monte-Carlo complete spatial randomness (CSR) null models.

**Methods used:**
1. **Voronoi tessellation** — normalized cell-area distribution vs. the Poisson–Voronoi gamma law
2. **Square-grid binning** — cell occupancy vs. Poisson distribution, tracked via the Fano factor
3. **K-means clustering** — point-to-centroid distance vs. a Monte-Carlo uniform null
4. **Nearest-neighbour distance** — vs. the theoretical Rayleigh distribution and the Clark–Evans index

**Agreement metrics:** R², Kolmogorov–Smirnov (D, p), CV ratios, CSR-based z-scores, and the Clark–Evans index (R_CE).

## Pipeline

1. Generate synthetic uniform and clustered point patterns (controlled ground truth)
2. Apply all four spatial-statistics estimators to both patterns
3. Benchmark empirical distributions against analytical/Monte-Carlo null models (R², KS test)
4. Extract real vortex centres from a turbulence velocity field and run the same pipeline on them

## Headline Result

The pipeline is validated on synthetic data — uniform patterns match theory to **R² ≈ 0.999** (KS test cannot reject the null), while clustered patterns fail every metric. Applied to a real isotropic turbulence slice, all four methods agree the **vortex centres are significantly clustered**:

| Method | R²(CSR) | KS p-value | Spread Measure | z-score | Verdict |
|---|---|---|---|---|---|
| Voronoi area | 0.525 | 3.9e-109 | CV ratio = 2.00 | 48.1 | Clustered |
| Grid-binning | -0.386 | 2.0e-21 | CV ratio = 2.14 | 31.4 | Clustered |
| K-means distance | 0.932 | 1.6e-02 | CV ratio = 1.06 | 2.2 | Clustered (weak) |
| Nearest-neighbour | 0.607 | 7.8e-70 | R_CE = 0.933 | -7.8 | Clustered |

The Voronoi cell-area spread is **2× wider than spatial randomness**, sitting **48 standard deviations beyond the CSR null**. Voronoi and grid-binning give the strongest signal; K-means is a weaker but corroborating detector.

## Data

`isotropic1024_slice.npz` — a 1024×1024 slice of the Johns Hopkins **3D forced isotropic turbulence** dataset (velocity fields `u, v, w`; periodic domain `[0, 2π]²`). Vortex centres are extracted as local maxima of out-of-plane vorticity `|ω_z|` above a `mean + 2·std` threshold, yielding **N = 3186** centres.

> Note: this is *passive 3D isotropic* turbulence — a different physical system from the *2D active* turbulence in the reference paper. The pipeline transfers identically; the interpretation here is "are these turbulent vortices clustered?", not a reproduction of the paper's activity-driven transition.

## Repository Structure

```
.
├── 01_voronoi_uniform_vs_nonuniform.ipynb        # Voronoi: uniform vs. clustered synthetic data
├── 02_gridbinning_and_kmeans.ipynb               # Grid-binning, K-means, nearest-neighbour comparison
├── 03_poisson_voronoi_goodness_of_fit.ipynb      # Voronoi R²/KS vs. Poisson-Voronoi gamma law
├── 04_gridbinning_kmeans_goodness_of_fit.ipynb   # Grid + K-means R²/KS goodness-of-fit
├── 05_real_data_vortex_clustering.ipynb          # Real turbulence data: degree-of-uniformity analysis
├── RESULTS.md                                    # Consolidated results tables and discussion
├── projectresearch-final.pdf                     # Full write-up (background, methodology, results, conclusion)
└── README.md
```

## Reproducing the Results

Each notebook already contains its executed outputs and plots. To regenerate them from scratch:

```bash
python -m nbconvert --to notebook --execute --inplace --ExecutePreprocessor.timeout=600 <notebook>.ipynb
```

| Notebook | Produces |
|---|---|
| `01_voronoi_uniform_vs_nonuniform.ipynb` | Voronoi spread statistics (Table 1) |
| `02_gridbinning_and_kmeans.ipynb` | Alternative-estimator comparison (Table 2) |
| `03_poisson_voronoi_goodness_of_fit.ipynb` | Voronoi R²/KS goodness-of-fit (Table 3) |
| `04_gridbinning_kmeans_goodness_of_fit.ipynb` | Grid + K-means R²/KS goodness-of-fit |
| `05_real_data_vortex_clustering.ipynb` | Real-data degree-of-uniformity (final results table) |

**Dependencies:** `numpy`, `scipy`, `matplotlib` (Voronoi via `scipy.spatial`, K-means via `scipy.cluster.vq.kmeans2`, nearest-neighbour search via `scipy.spatial.cKDTree`).

## Key Findings

1. **Validated** four independent spatial-statistics estimators on synthetic ground truth — uniform patterns match theory (R² ≈ 0.999, KS cannot reject), clustered patterns fail every metric.
2. **Quantified** the empirical-vs-theory closeness with R² and KS, correctly distinguishing one-sample (continuous) vs. two-sample (discrete / no-closed-form) KS treatment.
3. **Applied to real turbulence data** and found, via four independently agreeing methods, that vortex centres are significantly clustered — up to 48σ beyond spatial randomness.

## Limitations / Deferred Work

- K-means is the weakest detector on real data (z = 2.2) — normalizing by the mean distance removes the scale shift, leaving only a faint shape difference. Treat it as corroborating, not primary.
- δ number-fluctuation scaling (Δn ~ n̄^δ) and the per-patch spatial CV/IQR map (roadmap steps 5a & 6) are intentionally not yet built.

## Reference

Kashyap, V., Kiran, U., & Gupta, R. *Emergence of Local Ordering and Mesoscale Giant Number Fluctuations in Active Turbulence*. arXiv:2507.04890v2.
