# Image Processing Basics

**Summary**: The 10 fundamental steps of digital image processing, image type definitions, and basic point operations — the pre-processing foundation for all biometric modalities.

**Pre-Req**: Basic linear algebra (matrices)

**Sources**: Updated Unit1_Biometrics(Part-2).pptx

**Last updated**: 2026-05-10

#biometrics

---

## Why Image Processing in Biometrics?

Biometric sensors (cameras, fingerprint scanners, iris imagers) produce raw digital images. Before feature extraction can work reliably, those images must be cleaned, normalized, and transformed. Image processing provides the toolkit for that pre-processing layer.

---

## The 10 Fundamental Steps

| Step | Purpose |
|---|---|
| **1. Acquisition** | Capture image from sensor |
| **2. Enhancement** | Improve visual quality, increase contrast, reduce noise |
| **3. Restoration** | Remove specific degradations (blur, sensor noise models) |
| **4. Color Processing** | Convert between color spaces, handle RGB/HSV/YCbCr |
| **5. Wavelets** | Multi-resolution analysis; texture decomposition |
| **6. Compression** | Reduce file size (JPEG, PNG, JPEG2000) |
| **7. Morphological Processing** | Shape-based operations: erosion, dilation, opening, closing |
| **8. Segmentation** | Separate foreground (ROI) from background — see [[image-segmentation]] |
| **9. Representation & Description** | Convert regions to feature descriptors |
| **10. Object Detection / Recognition** | Identify and classify objects in the image |

Steps 1–7 are pre-processing. Steps 8–10 are analysis. Biometrics primarily uses steps 1–2 (enhancement) and step 8 (segmentation) before handing off to trait-specific feature extraction.

---

## Image Types

| Type | Bit depth | Levels | Description |
|---|---|---|---|
| **Binary** | 1 bit | 2 (black/white) | Used after thresholding (fingerprint binarization) |
| **Grayscale** | 8 bits | 256 (0–255) | Standard for fingerprint, iris, face |
| **Color (RGB)** | 24 bits | 16.7M | 3 channels × 8 bits; face recognition with color |

A **grayscale image** is a 2D matrix where each entry f(x,y) ∈ {0, 1, …, 255}.

---

## Point Operations

A **point operation** transforms each pixel independently:

```
g(x, y) = T(f(x, y))
```

- f(x,y) = input intensity at pixel (x,y)
- g(x,y) = output intensity
- T = transformation function

**Homogeneous**: same T applied to every pixel  
**Non-homogeneous**: T varies with position (x,y) — e.g., adaptive thresholding

The shorthand form: `s = T(r)` where r = input gray level, s = output gray level.

Point operations are the basis for all techniques in [[image-enhancement]].

---

## Interpolation

Used when resizing or rotating an image — must estimate pixel values at non-integer coordinates.

| Method | Neighbors used | Quality | Cost |
|---|---|---|---|
| **Nearest neighbor** | 1 | Low (blocky) | Cheapest |
| **Bilinear** | 4 | Medium | Moderate |
| **Bicubic** | 16 | High (smooth) | Expensive |

**Bilinear interpolation formula** (unit square, four corners):

```
f(x,y) = f(0,0)(1−x)(1−y) + f(1,0)·x·(1−y) + f(0,1)(1−x)·y + f(1,1)·x·y
```

(source: Updated Unit1_Biometrics(Part-2).pptx)

---

## Thresholding

Converts a grayscale image to binary using a threshold T:

```
g(x,y) = 255  if f(x,y) ≥ T
g(x,y) = 0    if f(x,y) < T
```

- **Global threshold**: single T for entire image
- **Adaptive/optimal threshold**: T determined from the histogram; different T per region

Thresholding is used in fingerprint binarization — see [[fingerprint-feature-extraction]].

---

## Related Pages

- [[image-enhancement]] — contrast stretching, histogram equalization, negatives
- [[image-segmentation]] — separating ROI from background
- [[fingerprint-feature-extraction]] — applies binarization and morphology to fingerprint images
