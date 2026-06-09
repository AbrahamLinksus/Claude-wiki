# Fingerprint Recognition

**Summary**: End-to-end fingerprint recognition — feature levels, pattern types, offline vs online acquisition, and how matching works.

**Pre-Req**: [[fingerprint-sensors]], [[fingerprint-feature-extraction]]

**Sources**: Unit2_Biometrics(1).pptx

**Last updated**: 2026-05-10

#biometrics

---

## Why Fingerprint?

Fingerprint is the most widely deployed biometric modality. It scores well on all 7 [[biometric-traits]] factors:
- High uniqueness (even identical twins have different fingerprints)
- Permanent across a lifetime (barring injury)
- Easy to capture with compact, cheap sensors
- Mature algorithms with decades of forensic validation

---

## Feature Levels Summary

Three levels of increasing detail and resolution requirement:

| Level | Features | Sensor requirement |
|---|---|---|
| **L1** | Orientation field, singular points, pattern class | Standard (≥250 dpi) |
| **L2** | Ridge skeleton, minutiae (type + (x,y) + angle), ridge count | Standard (≥500 dpi) |
| **L3** | Contour shape, pores, dot features, incipient ridges | High-resolution (≥1000 dpi) |

Full extraction pipeline → [[fingerprint-feature-extraction]].

---

## Fingerprint Pattern Types

Determined by the singular points (Poincaré Index):

| Pattern | Singular points | Description |
|---|---|---|
| **Arch** | None | Ridges enter one side, curve, exit the other; no core or delta |
| **Loop** | 1 core + 1 delta | Ridges loop back; most common pattern (~65% of fingerprints) |
| **Whorl** | 2 cores + 2 deltas | Concentric circular ridges; ~30% of fingerprints |

Pattern type is an L1 feature and is used to index large databases (e.g., AFIS) to narrow the search space before L2 matching.

---

## Acquisition Modes

| Mode | Description | Use case |
|---|---|---|
| **Offline (latent)** | Lifted from surface post-event (ink/dust on crime scene) | Forensic AFIS |
| **Online (live-scan)** | Captured directly from finger on sensor in real time | Access control, phones, border |

Offline fingerprints are typically **partial**, **distorted**, and **noisy** — they present a harder matching problem than online live-scan.

Scanner operation modes: plain, rolled, sweep — see [[fingerprint-sensors]].

---

## Minutiae-Based Matching

The standard matching approach:

1. Extract minutiae set M_probe from probe image
2. Extract (or retrieve) minutiae set M_gallery from stored template
3. Find the **maximum alignment** between the two sets (rotation + translation)
4. Count the number of matching minutiae pairs within a tolerance window
5. **Match score** = fraction of matched minutiae / total minutiae

A higher score → more likely same finger. Score is compared to threshold τ to accept or reject. See [[biometric-system-modules]] for threshold trade-off.

---

## Distortion and Challenges

Fingerprint capture is subject to:
- **Elastic skin distortion**: pressing harder stretches ridges non-linearly
- **Rotation and translation**: finger placement varies each time
- **Partial overlap**: only a portion of the finger contacts the sensor
- **Noise**: sweat, dirt, cuts, dry skin degrade image quality
- **Intra-class variation**: same finger → different appearance across captures

Multiple templates per finger address intra-class variation during enrollment.

---

## AFIS — Automated Fingerprint Identification System

Large-scale 1:N identification system (law enforcement, civil ID):
- Database of millions of fingerprint records
- L1 pattern type used for indexing → narrows candidate list
- L2 minutiae matching against candidates
- Human latent print examiner confirms top match

Used in: [[biometric-applications]] (forensics, border control, Aadhaar).

---

## Related Pages

- [[fingerprint-sensors]] — how the raw image is captured
- [[fingerprint-feature-extraction]] — 4-step pipeline to produce features
- [[biometric-fundamentals]] — verification vs identification modes
- [[biometric-applications]] — AFIS, civil ID, border control, phone unlock
- [[multibiometrics]] — fingerprint combined with face/iris for higher accuracy
