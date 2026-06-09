# Image Segmentation

**Summary**: Methods for separating the region of interest (e.g., fingerprint area, iris boundary) from the background in biometric images.

**Pre-Req**: [[image-processing-basics]], [[image-enhancement]]

**Sources**: Unit2_Biometrics(1).pptx

**Last updated**: 2026-05-10

#biometrics

---

## What Is Segmentation?

Segmentation partitions an image into meaningful regions. In biometrics, the goal is to **isolate the biometric ROI** (fingerprint area, iris ring, face boundary) from background and noise before feature extraction.

---

## 1. Threshold-Based Segmentation

Uses a threshold T on pixel intensity to classify pixels as foreground or background.

```
g(x,y) = 1  if f(x,y) ≥ T  (foreground)
g(x,y) = 0  otherwise       (background)
```

- **Global threshold**: single T for the whole image; fast but fails with uneven illumination
- **Adaptive threshold**: T computed locally per region from the histogram
- Works well for fingerprints where ridges and background have distinct intensity distributions

---

## 2. Edge-Based Segmentation

Detects boundaries between regions using gradient operators. Edges occur where intensity changes rapidly.

| Operator | Characteristics |
|---|---|
| **Sobel** | 3×3 kernel; smooth + differentiate; common for face/iris boundary |
| **Canny** | Multi-stage: Gaussian smoothing → gradient → NMS → hysteresis; best edge quality |
| **Prewitt** | Similar to Sobel, slightly different kernel weights |
| **Roberts** | 2×2 diagonal kernels; fast, sensitive to noise |
| **Kirsch** | 8 directional kernels; captures edges in all orientations |

After edge detection, boundaries are closed and filled to define regions.

---

## 3. Region-Based Segmentation

Groups pixels based on similarity criteria rather than boundaries.

### Region Growing
- Start from a seed pixel
- Grow region by adding neighboring pixels that satisfy a homogeneity criterion (similar intensity, texture)
- Stop when no more neighbors qualify

### Split and Merge
- Start with the entire image as one region
- **Split** any region that is not homogeneous: region is split into 4 quadrants if `|Z_max − Z_min| > threshold`
- **Merge** adjacent regions that are sufficiently similar
- Stop when no more splits or merges can be made

(source: Unit2_Biometrics(1).pptx)

---

## 4. Clustering-Based Segmentation

Groups pixels into clusters based on feature similarity (intensity, color, texture) without predefined boundaries.

### K-Means Clustering
1. Initialize K cluster centroids randomly
2. Assign each pixel to nearest centroid
3. Update centroids to mean of assigned pixels
4. Repeat until convergence

K must be specified in advance. Common in skin segmentation for face detection.

---

## 5. ANN-Based Segmentation

Trained neural network classifies each pixel (or region) as belonging to a biometric region or background.

- Requires labeled training data
- More flexible than rule-based methods
- Used in deep learning pipelines for iris, face, and fingerprint localization

---

## Segmentation in the Biometric Pipeline

```
Raw Image → Enhancement → Segmentation → ROI → Feature Extraction
```

Each modality has a specific segmentation challenge:
- **Fingerprint**: separate fingerprint area from background, handle rolled vs slap captures
- **Iris**: localize iris between pupil and sclera boundaries (Daugman's integro-differential operator)
- **Face**: detect face bounding box, separate from background and hair

---

## Related Pages

- [[image-processing-basics]] — image types and point operations
- [[image-enhancement]] — pre-processing applied before segmentation
- [[fingerprint-feature-extraction]] — uses binarization (threshold-based) to segment ridges from valleys
- [[pattern-recognition]] — segmentation feeds the feature extraction stage
