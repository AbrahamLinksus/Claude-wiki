# Fingerprint Feature Extraction

**Summary**: The 4-step algorithm that transforms a raw fingerprint image into a set of discriminative features used for matching.

**Pre-Req**: [[image-processing-basics]], [[image-enhancement]], [[fingerprint-sensors]]

**Sources**: Unit2_Biometrics(1).pptx

**Last updated**: 2026-05-10

#biometrics

---

## Overview

After a sensor captures a fingerprint image, four sequential processing steps extract the features needed for matching. The output feeds the matcher module in [[biometric-system-modules]].

```
Raw Image → (a) Orientation/Frequency → (b) Ridge Extraction → (c) Singularity → (d) Binarization/Thinning → Features
```

(source: Unit2_Biometrics(1).pptx, Fig 2.15)

---

## Step (a): Ridge Orientation and Frequency Estimation

**Purpose**: Estimate the local direction and spacing of ridges across the entire fingerprint — used to guide all subsequent steps.

**Method**:
1. Divide fingerprint image into non-overlapping blocks
2. For each block, compute the **dominant local direction** θ_ij (0° ≤ θ < 180°) — this is the block's orientation
3. The ridge pattern within each block approximates a cosine wave:

```
w(x,y) = A · cos(2π f₀ (x·cosθ + y·sinθ))
```

- A = amplitude
- f₀ = ridge spatial frequency (ridges per pixel)
- θ = ridge orientation angle

**Ridge count**: count of ridges intersected by an imaginary line drawn from the **delta** point to the **core** point. Used to characterize fingerprint pattern type.

---

## Step (b): Ridge Extraction

**Purpose**: Identify which pixels belong to ridges vs valleys.

**Basic rule**: Ridges are darker → pixels with gray value below a threshold are ridge pixels.

**Complications**:
- **Pores**: appear as bright spots on ridges — can be mistaken for valleys
- **Broken ridges**: ridge continuity is interrupted — must be filled or handled
- **Merging ridges**: adjacent ridges touch — must be separated

These complications make robust thresholding non-trivial; adaptive local thresholds are preferred over a single global threshold.

---

## Step (c): Singularity Extraction via Poincaré Index

**Purpose**: Locate the **core** and **delta** singular points that define fingerprint topology.

**Poincaré Index (PI)**: measures the cumulative change in ridge orientation along a closed path around a point.

```
PI = (1/2π) × Σ Δθ  (summed around the closed path using 8-neighbor angles)
```

| PI value | Singular point type | Symbol |
|---|---|---|
| 0 | Non-singular (regular ridge) | — |
| +1 | **Loop** (core) | ∩ |
| −1 | **Delta** | Δ |
| +2 | **Whorl** | O |

These singular point types directly determine the **fingerprint pattern class**:
- **Arch**: no singularities (PI=0 everywhere)
- **Loop**: one core (PI=+1) + one delta (PI=−1)
- **Whorl**: two cores + two deltas (PI=+2 and PI=−1)

(source: Unit2_Biometrics(1).pptx)

---

## Step (d): Binarization and Thinning

**Binarization**: Convert the grayscale ridge image to binary (ridge = 1, valley = 0) using a threshold. See [[image-processing-basics]] for thresholding.

**Thinning (Skeletonization)**: Iteratively erode binary ridges until they are **1 pixel wide**. This creates the ridge skeleton, which makes **minutiae detection** precise — minutiae are located at ridge endpoints and bifurcations in the 1-pixel skeleton.

After thinning → **minutiae extraction**: scan skeleton for:
- **Ridge ending**: a ridge pixel with only one neighbor
- **Bifurcation**: a ridge pixel with three neighbors

---

## Feature Levels (L1 / L2 / L3)

Commercial systems use a hierarchical strategy:

| Level | Name | Features | Extracted when |
|---|---|---|---|
| **L1** | Global | Orientation field, singular points (core/delta), pattern class | First; guides L2 extraction |
| **L2** | Local | Ridge skeleton, minutiae (type + location + angle), ridge count | Guided by L1 |
| **L3** | Fine | Contour, pores, dots, incipient ridges | High-resolution sensors only |

L1 is extracted first; its output (orientation map + singular points) **guides** where L2 features are extracted. L3 is only available from high-resolution (>500 dpi) sensors.

---

## Minutiae Types

Primary L2 features used in matching:

| Minutia | Description |
|---|---|
| **Ridge ending** | A ridge that terminates |
| **Bifurcation** | A ridge that splits into two |
| Hook | Short ridge branching off another |
| Dot | Very short isolated ridge |
| Enclosure | Two branches reuniting to form a closed loop |
| Delta | Three ridges meeting at a point |
| Bridge | Short ridge connecting two parallel ridges |
| Ridge crossing | Two ridges that cross |

In practice, most matchers rely only on **endings** and **bifurcations** as they are most reliably detected.

---

## Related Pages

- [[fingerprint-sensors]] — sensor types and acquisition modes (step before this)
- [[fingerprint-recognition]] — full recognition pipeline and matching
- [[image-processing-basics]] — binarization, morphological processing
- [[image-enhancement]] — applied before step (a) for contrast improvement
- [[dimensionality-reduction]] — PCA/LDA can reduce the feature vector before matching
