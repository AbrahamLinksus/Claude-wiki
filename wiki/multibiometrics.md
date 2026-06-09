# Multibiometrics

**Summary**: Systems that combine multiple biometric sources — the 5 evidence types, 3 acquisition modes, 3 system types, and why they outperform unimodal systems.

**Pre-Req**: [[biometric-fundamentals]], [[biometric-traits]]

**Sources**: Unit1_Biometrics(Part-1).pptx, Unit3_Biometrics(1).pptx

**Last updated**: 2026-05-10

#biometrics

---

## Why Multibiometrics?

Unimodal systems (single trait) have four inherent weaknesses:
1. **Noisy data**: sensor failures or poor capture quality
2. **Inter-class similarity**: some individuals share very similar trait values → high FMR
3. **Population incompatibility**: a fraction can't provide the trait (amputees, skin conditions)
4. **Spoof vulnerability**: a single trait is easier to fake than multiple

Multimodal systems combine 2+ evidence sources to:
- Reduce FAR and FRR simultaneously
- Provide fallback when one source fails
- Make spoofing significantly harder
- Handle large-scale populations (civil ID, border control)

(source: Unit1_Biometrics(Part-1).pptx)

---

## 5 Evidence Sources (Integration Scenarios)

| Scenario | What is combined |
|---|---|
| **Multiple sensors** | Same trait captured by two different sensor types (e.g., visible + IR camera for face) |
| **Multiple biometrics** | Different traits (e.g., face + fingerprint) |
| **Multiple units** | Same trait from different body parts (e.g., right index + right middle finger) |
| **Multiple snapshots** | Multiple captures of the same trait instance (e.g., two attempts of same finger) |
| **Multiple matchers** | Same feature vector processed by two or more algorithms/classifiers |

(source: Unit1_Biometrics(Part-1).pptx, pentagon diagram)

---

## 3 Acquisition Modes

| Mode | How it works | Trade-off |
|---|---|---|
| **Serial / Cascade** | One trait captured first; second only if first is inconclusive | Fast when first trait is decisive; slower when cascading |
| **Parallel** | All traits captured simultaneously | Always captures everything; higher throughput; more hardware |
| **Hybrid** | Mix of serial and parallel stages | Flexible; complexity in scheduling |

---

## 3 System Types

### Multi-Algorithmic
- **Single sensor** captures one trait once
- **2+ different algorithms** process the same feature vector
- Output: multiple scores → fused into one decision
- Example: two fingerprint matchers (minutiae-based + ridge-frequency-based) on the same image

### Multi-Instance
- **1+ sensors** capture **multiple samples of the same trait** from different body locations
- Example: left index + right thumb fingerprints
- Improves coverage; reduces impact of a damaged finger

### Multi-Sensorial
- **2+ different sensor types** capture the **same trait instance**
- Example: visible-light camera + infrared camera for the same face
- Each sensor captures a complementary representation of the same physical measurement

(source: Unit1_Biometrics(Part-1).pptx)

---

## System Components

Every multimodal system has:
- **Sensor modules** — one per captured modality
- **Feature extraction modules** — one per modality
- **Fusion module** — combines evidence (see [[biometric-fusion]])
- **Matching module** — single matcher or per-modality matchers
- **Decision-making module** — final accept/reject

Fusion can occur at different points in the pipeline — see [[biometric-fusion]] for the full taxonomy.

---

## Advantages vs Disadvantages

| Advantages | Disadvantages |
|---|---|
| Reduced FAR and FRR | More expensive (multiple sensors) |
| Harder to spoof (multiple traits required) | Slower enrollment (more samples needed) |
| Handles non-universal traits (fallback) | Bad fusion technique can degrade accuracy *below* unimodal |
| Robust to noisy single-modality capture | Higher system complexity |

---

## Unimodal vs Multimodal Comparison

| | Unimodal | Multimodal |
|---|---|---|
| **Accuracy** | Moderate | High (reduced FAR/FRR) |
| **Spoof resistance** | Lower | Higher (must fake all traits) |
| **Cost** | Lower | Higher |
| **Enrollment speed** | Fast | Slower |
| **Scalability** | Small–medium | Large-scale civil ID |

---

## Applications

- **Civil ID**: Aadhaar (India) uses fingerprint + iris + face
- **Border control**: fingerprint + face at airports
- **Secure banking**: palm vein + PIN at ATMs

See [[biometric-applications]] for details.

---

## Related Pages

- [[biometric-fundamentals]] — unimodal baseline
- [[biometric-traits]] — 7 factors and unimodal limitations
- [[biometric-fusion]] — how multiple evidence sources are combined into one decision
- [[biometric-applications]] — real-world deployments of multimodal systems
