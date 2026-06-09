# Image Enhancement

**Summary**: Techniques for improving the visual quality and contrast of biometric images — including intensity transformations and histogram-based methods.

**Pre-Req**: [[image-processing-basics]]

**Sources**: Updated Unit1_Biometrics(Part-2).pptx

**Last updated**: 2026-05-10

#biometrics

---

## Overview

Enhancement adjusts pixel intensities to make features more visible and easier to extract. All methods here are **point operations**: `s = T(r)`.

Four core spatial-domain techniques, then histogram-based methods.

---

## a. Image Negatives

**Formula**:
```
g(x,y) = 255 − f(x,y)
```

Inverts the intensity scale: bright pixels become dark, dark become bright.

**Use in biometrics**: medical imaging (X-rays), monochrome film analysis, enhancing light features on dark backgrounds.

---

## b. Contrast Stretching

**Problem**: image intensities occupy only a narrow sub-range (e.g., 80–150 out of 0–255).  
**Solution**: linearly scale the range to fill [0, 255].

**Formula**:
```
g(x,y) = (f(x,y) − r_min) / (r_max − r_min) × 255
```

**Worked example** (source: Updated Unit1_Biometrics(Part-2).pptx):
- Input range: [80, 150]; pixel value 115
- g = (115 − 80) / (150 − 80) × 255 = 35/70 × 255 ≈ **127**

**Use in biometrics**: low-contrast fingerprint images, poorly lit face images.

---

## c. Compression of Dynamic Range (CDR)

**Problem**: scene has a dynamic range that exceeds the display device's range (very dark and very bright regions simultaneously).  
**Solution**: logarithmic compression maps the wide range to a displayable range.

**Formula**:
```
s = c · log(1 + |r|)
```

- c = scaling constant
- log compresses high values more than low values (non-linear)

**Use in biometrics**: face recognition under high-contrast lighting, iris imaging, vein pattern imaging in NIR where bright/dark regions co-exist.

---

## d. Gray-Level Slicing

Highlights a specific intensity range [A, B] — useful when the feature of interest occupies a known intensity band.

**Mode 1 — suppress background** (binary output):
```
g(x,y) = 255   if A ≤ f(x,y) ≤ B
g(x,y) = 0     otherwise
```

**Mode 2 — retain background** (selective highlight):
```
g(x,y) = 255       if A ≤ f(x,y) ≤ B
g(x,y) = f(x,y)    otherwise
```

**Use in biometrics**: hand vein imaging in near-infrared (NIR) — veins fall in a known intensity band; slicing isolates them against the surrounding tissue.

(source: Updated Unit1_Biometrics(Part-2).pptx)

---

## Histogram Processing

### What Is a Histogram?

A histogram H(r) counts how many pixels have each intensity value r ∈ {0, …, 255}.

```
x-axis = intensity value (0–255)
y-axis = pixel count (frequency)
```

A **dark image** has histogram concentrated near 0.  
A **bright image** has histogram concentrated near 255.  
A **low-contrast image** has a narrow, peaked histogram.

---

### Histogram Equalization

**Goal**: redistribute pixel intensities so the histogram is approximately uniform across [0, 255] — maximizing contrast.

**Algorithm**:
1. Compute histogram H(r_k) for each intensity level r_k
2. Compute probability: p(r_k) = H(r_k) / (total pixels)
3. Compute CDF: CDF(r_k) = Σ_{j=0}^{k} p(r_j)
4. Map: `s_k = round[(L − 1) × CDF(r_k)]`  
   where L = 256 (number of gray levels)

**Constraints on T(r)**:
- Must be **single-valued** (no ambiguity)
- Must be **monotonically increasing** in [0, 1] (preserves ordering)

**Applications**:
- Medical imaging (X-ray, MRI enhancement)
- Satellite/aerial imagery
- Low-light photography pre-processing
- Computer vision pre-processing before face/iris detection

(source: Updated Unit1_Biometrics(Part-2).pptx)

---

### Histogram Specification (Matching)

Instead of equalizing to a uniform distribution, maps the input histogram to a **target histogram** (any desired distribution). Useful when a specific contrast profile is needed.

### Local Enhancement

Applies histogram equalization to small local windows rather than the whole image — preserves local contrast in images with varying illumination across regions.

---

## Comparison Summary

| Technique | Effect | Key Formula |
|---|---|---|
| Negatives | Invert | `255 − f` |
| Contrast Stretching | Linear expand to [0,255] | `(f − r_min)/(r_max − r_min) × 255` |
| CDR | Log compress wide range | `c·log(1 + |r|)` |
| Gray-level Slicing | Isolate intensity band [A,B] | Conditional on [A,B] |
| Histogram Equalization | Uniform redistribution | `round[(L−1) × CDF(r_k)]` |

---

## Related Pages

- [[image-processing-basics]] — image types, point operations, thresholding
- [[image-segmentation]] — next step after enhancement
- [[fingerprint-feature-extraction]] — applies enhancement before ridge extraction
