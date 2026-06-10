---
title: "Unsupervised Conformal Novelty Detection for Hierarchical Data"
authors:
  - me
  - Seonghun Cho
date: "2026-01-01T00:00:00Z"
weight: 10
publishDate: "2026-01-01T00:00:00Z"
publication_types: ["article"]
peer_reviewed: false
featured: true
reading_time: true
image:
  preview_only: true
links:
  - name: PDF
    url: /uploads/papers/unsupervised-conformal-novelty-detection.pdf
summary: "A conformal e-value framework for unsupervised novelty detection in hierarchically structured data, achieving FDR control at both group and unit levels."
tags:
  - Conformal Prediction
  - Novelty Detection
  - Hierarchical Data
  - False Discovery Rate
  - Multiple Testing
---

---

<div style="display:grid;grid-template-columns:160px 1fr;gap:1.5rem 2rem;margin-bottom:2rem;">

<div style="font-weight:700;padding-top:.1rem;">Abstract</div>
<div>

Electric vehicle (EV) battery packs exhibit a natural pack–module–cell hierarchy, which induces dependence among measurements within the same module. Such hierarchical dependence poses challenges for the direct application of conventional novelty detection methods. To address these challenges, we develop conformal e-value procedures for hierarchical novelty detection, with the goal of controlling the false discovery rate (FDR) at both the group and unit levels. We combine hierarchical conformal score construction with eBH and U-eBH multiple testing procedures, and consider split conformal, full conformal, and group-wise conformal regimes. The proposed methods are evaluated through simulation studies and an analysis of EV battery pack data, where they provide empirical FDR control and detect localized module- and cell-level irregularities.

</div>

<div style="font-weight:700;padding-top:.1rem;">Type</div>
<div>Preprint</div>

<div style="font-weight:700;padding-top:.1rem;">Publication</div>
<div>To be submitted to <em>Journal of the Korean Statistical Society</em> (JKSS, SCIE)</div>

<div style="font-weight:700;padding-top:.1rem;">Keywords</div>
<div>novelty detection · hierarchical structure · conformal inference · false discovery rate · multiple testing</div>

</div>

---

## Motivation

Real-world industrial data — such as EV battery manufacturing — presents three compounding challenges that standard anomaly detection cannot handle jointly:

- **Hierarchical dependence:** Cells within the same module share a common group effect, violating the IID assumption of most methods.
- **No anomaly labels:** Ground-truth defect labels are expensive or infeasible to obtain in manufacturing, ruling out supervised approaches.
- **Multiple comparisons:** Simultaneously testing hundreds of units inflates false discoveries without a principled error-control mechanism.

---

## Method Overview

We frame hierarchical novelty detection as a **multiple hypothesis testing problem** at two levels simultaneously:

- **Group-level (HC-GND)** — Does a test module contain any anomalous cells?
- **Unit-level (HC-UND)** — Which specific cells within each module are anomalous?

**Feature extraction:** Charging voltage time series are treated as functional observations. Functional PCA (FPCA) projects each cell's charging curve onto a low-dimensional score vector, placing cells from all packs on a common feature space.

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/fig_charge_curves.png"
       alt="Preprocessed charging voltage curves for all cells in the six test battery packs"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    Preprocessed charging voltage curves for all cells in the six test packs. Normal packs (blue) show regular charging behavior; Abnormal packs 4 and 5 (red) exhibit irregular patterns that are difficult to distinguish visually at the pack level.
  </figcaption>
</figure>

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/fig_fpca_scatter.png"
       alt="FPCA scatter plot separating normal and abnormal battery packs"
       style="width:80%;display:block;margin:0 auto;border-radius:8px;border:1px solid rgba(148,163,184,.25);">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    Two-dimensional FPCA score vectors under the split conformal regime. Normal reference packs (black) form a tight cluster, while abnormal test packs (red) appear in a distinct region, validating FPCA as a discriminative feature extraction step.
  </figcaption>
</figure>

**Nonconformity score:** For each cell, we compute a **Mahalanobis-distance-based nonconformity score** relative to a robust trimmed-mean group center. This naturally respects within-group dependence.

**Multiple testing:** Scores are converted to **conformal e-values** — non-negative statistics satisfying E[e] ≤ 1 under the null — and tested jointly via the **eBH / U-eBH procedure**, guaranteeing FDR control under minimal distributional assumptions.

We implement three conformal regimes with different data-efficiency / robustness trade-offs:

| Regime | Training data | Contamination risk | Theoretical FDR guarantee |
|:---|:---|:---|:---|
| **HSC** (Split) | Reference only | None | ✓ Both levels |
| **HFC** (Full) | Reference + all test | Higher | ✓ Group level |
| **HGC** (Group-wise) | Reference + one test group | Moderate | ✓ Group level |

---

## Results

### Simulation Study

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/fig_h_gnd.png"
       alt="HC-GND simulation results: FDR and power under varying outlier proportion and signal strength"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    <strong>HC-GND simulation results.</strong> Empirical FDR (top) and power (bottom) under varying outlier proportions (left) and signal strengths (right). All three regimes maintain FDR below the target α = 0.1. HFC and HGC achieve higher power than HSC, especially at low outlier proportions, by exploiting more calibration data.
  </figcaption>
</figure>

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/fig_comparison.png"
       alt="HC-UND vs non-hierarchical baselines: FDR and power comparison"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    <strong>HC-UND vs. non-hierarchical baselines.</strong> The proposed hierarchical methods (HSC, HFC, HGC) consistently outperform non-hierarchical counterparts (SC, FC, AdaDetect) in power while maintaining empirical FDR control. Unit-level anomalies that are subtle in the pooled population become detectable within their own group context.
  </figcaption>
</figure>

Key findings across 1,000 simulation replicates (α = 0.1):

- Empirical FDR remains **at or below the target level** across all outlier proportions and signal strengths.
- At weak signal, power reaches ~**40%**; as signal increases, power **approaches 100%**.
- Hierarchical methods **outperform non-hierarchical baselines** by leveraging within-group structure.

### EV Battery Pack Application

<figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/fig_pack_wise.png"
       alt="Pack-wise comparison of hierarchical and non-hierarchical detection methods"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    <strong>Pack-wise detection results (α = 0.1).</strong> Left three columns: hierarchical HC-GND/HC-UND results (yellow = module rejection, red = cell rejection). Right three columns: non-hierarchical baselines (SVM, Isolation Forest, AdaDetect). The proposed methods identify localized irregularities in Abnormal Pack 5 across multiple modules and cells that non-hierarchical methods largely miss.
  </figcaption>
</figure>

- The proposed procedures detect **localized module- and cell-level irregularities** not captured by pack-level labels.
- Non-hierarchical baselines concentrate false detections on Normal Pack 0 and miss structured signals in Abnormal Pack 5.
- Results provide an **additional diagnostic layer** when only coarse pack-level labels are available.

---

## Presentation Slides

<div style="margin-top:1rem;">
  <img
    src="/uploads/papers/thesis-page-4.png"
    alt="Presentation slide 1 — Overview"
    style="width:100%;border:1px solid rgba(148,163,184,.3);border-radius:8px;margin-bottom:1.25rem;"
  >
  <img
    src="/uploads/papers/thesis-page-5.png"
    alt="Presentation slide 2 — Real data application"
    style="width:100%;border:1px solid rgba(148,163,184,.3);border-radius:8px;"
  >
</div>

<!--more-->
