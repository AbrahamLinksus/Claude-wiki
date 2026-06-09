# Biometrics Overview

**Summary**: A complete roadmap for the biometrics course — covering system fundamentals, image processing, feature extraction, multimodal fusion, security, and real-world applications.

**Sources**: Unit1_Biometrics(Part-1).pptx, Updated Unit1_Biometrics(Part-2).pptx, Unit2_Biometrics(1).pptx, Unit3_Biometrics(1).pptx, Unit4_Slides.pdf, Unit5_Slides.pdf

**Last updated**: 2026-05-10

#biometrics

---

## What Is Biometrics?

Biometrics is the automated recognition of individuals based on biological or behavioral traits. Every biometric system is fundamentally a **pattern recognition system**: it captures a sample, extracts features, and matches those features against stored templates.

Two core operations:
- **Verification (1:1)** — "Are you who you claim to be?" Compares probe against one enrolled template.
- **Identification (1:N)** — "Who are you?" Compares probe against all enrolled templates.

---

## Topic Map

### Foundations
- [[biometric-fundamentals]] — System pipeline, verification vs identification, gallery vs probe, enrollment vs authentication
- [[biometric-traits]] — 7 biometric trait factors (Universality → Circumvention), 5 trait taxonomy categories
- [[biometric-system-modules]] — 5 system modules: sensor, quality assessment, feature extraction, matching, database

### Image Processing (Pre-processing Layer)
- [[image-processing-basics]] — 10 fundamental steps, image types (binary/grayscale/color), point operations
- [[image-enhancement]] — Negatives, contrast stretching, dynamic range compression, gray-level slicing, histogram equalization
- [[image-segmentation]] — Threshold, edge-based (Sobel/Canny), region-based, clustering, ANN-based

### Feature Extraction
- [[fingerprint-recognition]] — Feature levels L1/L2/L3, minutiae types, singularity types, offline vs online acquisition
- [[fingerprint-sensors]] — FTIR, capacitance, ultrasound, temperature differential, piezoelectric; plain/rolled/sweep operation
- [[fingerprint-feature-extraction]] — 4-step algorithm: orientation estimation → ridge extraction → Poincaré index → binarization
- [[pattern-recognition]] — Sensor → segmentation → feature extraction → classification → decision pipeline
- [[dimensionality-reduction]] — PCA (unsupervised) vs LDA (supervised); Fisher's criterion; when to use each

### Multimodal Systems
- [[multibiometrics]] — 5 evidence sources, acquisition modes, multi-algorithmic/instance/sensorial types
- [[biometric-fusion]] — Fusion at sensor/feature/score/rank/decision level; formulas and taxonomy

### Security
- [[biometric-security]] — 4 threat categories, 5 vulnerability areas (A–E), attack types (spoof, MITM, replay, hill-climbing)
- [[template-protection]] — 3 required properties, 4 protection approaches, Table 7.1 comparison

### Applications
- [[biometric-applications]] — Access control, airport/immigration, Aadhaar, banking, AFIS, embedded systems

---

## Recommended Learning Order

1. [[biometric-fundamentals]] → [[biometric-traits]] → [[biometric-system-modules]]
2. [[image-processing-basics]] → [[image-enhancement]] → [[image-segmentation]]
3. [[fingerprint-sensors]] → [[fingerprint-feature-extraction]] → [[fingerprint-recognition]]
4. [[pattern-recognition]] → [[dimensionality-reduction]]
5. [[multibiometrics]] → [[biometric-fusion]]
6. [[biometric-security]] → [[template-protection]]
7. [[biometric-applications]]
