# Biometric Fundamentals

**Summary**: Core concepts of any biometric system — the recognition pipeline, verification vs identification, and how enrollment and authentication work.

**Pre-Req**: None

**Sources**: Unit1_Biometrics(Part-1).pptx

**Last updated**: 2026-05-10

#biometrics

---

## What Is a Biometric System?

A biometric system is a **pattern recognition system** that:
1. Acquires biometric data from an individual
2. Extracts a feature set from that data
3. Compares the feature set against stored templates
4. Makes an accept/reject decision

It operates in either **verification** or **identification** mode.

---

## Verification vs Identification

| | Verification (1:1) | Identification (1:N) |
|---|---|---|
| **Question** | "Are you who you claim to be?" | "Who are you?" |
| **Comparison** | Probe vs one claimed identity's template | Probe vs all N enrolled templates |
| **Speed** | Fast | Scales with N |
| **Use case** | Phone unlock, border e-gates | AFIS, watchlists |

**Positive identification** — system finds a claimed identity in the database.  
**Negative identification** — system confirms an individual is *not* enrolled (welfare fraud prevention).

System types: **Open** (identification against unknown population), **Closed** (identification within known enrolled set), **Semi-automated** (human-in-the-loop).

---

## System Pipeline

```
Sensor → Pre-processing → Feature Extractor → Template Generator → Matcher → Decision
```

(source: Unit1_Biometrics(Part-1).pptx)

- **Sensor**: captures raw biometric signal (camera, fingerprint scanner, microphone)
- **Pre-processing**: removes noise, normalizes image — see [[image-processing-basics]]
- **Feature Extractor**: extracts discriminative measurements from processed signal
- **Template Generator**: encodes features into a stored reference template
- **Matcher**: computes similarity score between probe features and stored template
- **Decision**: applies threshold to score → accept or reject

---

## Gallery vs Probe

- **Gallery** (enrollment set): templates stored during the enrollment phase
- **Probe** (recognition set): sample presented at authentication time

Multiple templates may be stored per person to handle **intra-class variation** (same person, different capture conditions — lighting, pose, age, injury).

---

## Enrollment vs Authentication

**Enrollment**:
1. Capture biometric sample(s)
2. Assess quality — reject low-quality samples (→ FTE: Failure To Enroll)
3. Extract features
4. Store template in database

**Authentication**:
1. Capture probe sample
2. Pre-process and extract features
3. Match against gallery template(s)
4. Decision: accept if score ≥ threshold, reject otherwise

---

## Key Error Rates

| Error | Meaning |
|---|---|
| **FMR / FAR** | False Match Rate — impostor accepted (linked to Uniqueness) |
| **FNMR / FRR** | False Non-Match Rate — genuine user rejected (linked to Permanence) |
| **FTE** | Failure To Enroll — sample too poor to enroll (linked to Measurability) |
| **FTA** | Failure To Acquire — sensor fails to capture usable sample |

FMR and FNMR trade off against each other via the decision threshold. See [[biometric-traits]] for how each factor drives each error type.

---

## Related Pages

- [[biometric-traits]] — the 7 factors that define a good biometric
- [[biometric-system-modules]] — detailed breakdown of the 5 system modules
- [[multibiometrics]] — combining multiple traits to reduce error rates
