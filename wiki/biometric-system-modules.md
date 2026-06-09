# Biometric System Modules

**Summary**: The 5 functional modules in every biometric system and how they fit together during enrollment and authentication.

**Pre-Req**: [[biometric-fundamentals]]

**Sources**: Unit1_Biometrics(Part-1).pptx

**Last updated**: 2026-05-10

#biometrics

---

## The 5 Modules

```
[Sensor] → [Quality Assessment] → [Feature Extraction] → [Matching / Decision] → [Database]
```

### 1. Sensor Module
- Captures the raw biometric signal
- Examples: optical fingerprint scanner, CCD camera, microphone, infrared vein imager
- Quality of capture directly limits all downstream steps
- Poor sensors → high FTA (Failure to Acquire)

### 2. Quality Assessment Module
- Evaluates whether the captured sample is usable
- Rejects or flags low-quality samples before feature extraction
- Low quality → request re-capture; consistently poor → FTE (Failure to Enroll)
- Quality metrics vary by modality: sharpness for fingerprint, focus for face, SNR for voice

### 3. Feature Extraction Module
- Transforms the raw signal into a compact, discriminative feature vector
- Must be **repeatable** (same person → similar features across captures) and **discriminative** (different persons → different features)
- Examples: minutiae points for fingerprint, eigenfaces for face, MFCCs for voice
- See [[fingerprint-feature-extraction]] for a worked fingerprint example

### 4. Matching / Decision Module
- **Matcher**: computes a similarity score between the probe feature vector and the stored template
- **Decision engine**: compares score to a threshold τ
  - Score ≥ τ → Accept
  - Score < τ → Reject
- Threshold is tunable: lower τ → lower FRR but higher FMR; higher τ → lower FMR but higher FRR

### 5. Database Module
- Stores enrolled templates (the **gallery**)
- May store multiple templates per person to handle intra-class variation
- Security of the database is critical — see [[template-protection]]

---

## Enrollment vs Authentication Flow

**Enrollment**:
```
Sensor → Quality Assessment → Feature Extraction → Template Generation → Store in DB
```

**Authentication**:
```
Sensor → Quality Assessment → Feature Extraction → Matcher (vs DB template) → Decision
```

The feature extraction pipeline is identical in both phases — the difference is what happens to the output.

---

## Integration in Multimodal Systems

In [[multibiometrics]], multiple instances of these modules run in parallel:
- Multiple **sensor modules** (one per trait or per sensor technology)
- Multiple **feature extraction modules**
- A **fusion module** sits between feature extraction and the matcher
- A single **decision module** consolidates the fused result

---

## Related Pages

- [[biometric-fundamentals]] — verification vs identification, error rates
- [[biometric-traits]] — what the sensor module captures
- [[fingerprint-sensors]] — concrete sensor types for fingerprint modality
- [[fingerprint-feature-extraction]] — inside the feature extraction module for fingerprints
- [[template-protection]] — securing the database module
- [[multibiometrics]] — multi-module architectures
