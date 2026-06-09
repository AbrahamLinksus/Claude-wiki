# Biometric Fusion

**Summary**: All levels at which multiple biometric evidence sources can be combined — from raw sensor data to final decisions — with formulas and a taxonomy of fusion strategies.

**Pre-Req**: [[multibiometrics]], [[pattern-recognition]]

**Sources**: Unit1_Biometrics(Part-1).pptx, Unit3_Biometrics(1).pptx

**Last updated**: 2026-05-10

#biometrics

---

## Where Can Fusion Happen?

The biometric pipeline has multiple stages, and fusion can occur at any of them:

```
Sensor → Feature Extraction → Matcher → Rank → Decision
  ↑             ↑               ↑         ↑        ↑
Sensor     Feature Level    Score Level  Rank    Decision
Level         Fusion          Fusion    Level     Level
                                        Fusion    Fusion
```

**Before matching**: sensor-level and feature-level (combine raw data or feature vectors)  
**After matching**: score-level, rank-level, decision-level (combine matcher outputs)

---

## 1. Sensor-Level Fusion

Combine raw signals from multiple sensors before any processing.

- Example: merge visible + IR images of a face into a single composite image
- Challenge: sensors must be co-registered (aligned spatially and temporally)
- Advantage: no information lost in feature extraction step
- Disadvantage: requires compatible sensor outputs; high data volume

---

## 2. Feature-Level Fusion

Concatenate or transform feature vectors from different modalities into a single joint feature vector.

```
x_combined = [x_modality1 | x_modality2]  (concatenation)
```

**Challenge**: feature vectors from different modalities may have:
- Different scales → **normalization** required before concatenation
- Very high dimensionality after concatenation → apply PCA or LDA to reduce (see [[dimensionality-reduction]])

**Feature selection**: the combined dimension should satisfy:
```
d_combined < d₁ + d₂
```
(select only the most informative features from the joint space)

(source: Unit3_Biometrics(1).pptx)

---

## 3. Score-Level Fusion (Most Common)

Each modality's matcher produces a **match score**. These scores are fused into a single composite score.

Most commonly used in practice because:
- Scores are easy to access from most commercial matchers
- Less sensitive to incompatibility issues than feature fusion
- Well-studied normalization and combination methods

### Score Normalization

Before fusion, scores from different matchers must be normalized to a common range:
- **Min-max normalization**: s' = (s − s_min) / (s_max − s_min)
- **Z-score normalization**: s' = (s − μ) / σ
- **Tanh normalization**: handles outliers better

### Score Fusion Taxonomy

#### A. Likelihood Ratio Based
- **Bayesian parametric**: model score distributions as Gaussians; compute likelihood ratio
- **Bayesian non-parametric**: estimate score distributions from data without Gaussian assumption

#### B. Transformation-Based (Classifier-Free)
Combine normalized scores using a fixed rule:

| Rule | Formula | Effect |
|---|---|---|
| **Sum** | s = Σᵢ sᵢ | Balanced combination |
| **Product** | s = Πᵢ sᵢ | Requires both high; strict |
| **Min** | s = min(sᵢ) | Dominated by weakest matcher |
| **Max** | s = max(sᵢ) | Dominated by strongest matcher |
| **Weighted Sum** | s = Σᵢ wᵢ sᵢ | Weights reflect matcher reliability |

#### C. Classification-Based
Train a classifier to map the vector of scores to a final decision:
- **Neural network (NN)**
- **Support Vector Machine (SVM)**
- **Decision tree**
- **Genetic algorithms**

(source: Unit3_Biometrics(1).pptx)

---

## 4. Rank-Level Fusion

Each modality's matcher returns a **ranked list** of candidate identities. Fusion combines these ranked lists.

| Method | Description |
|---|---|
| **Highest Rank** (eq 6.27) | Select identity with the highest rank across all matchers |
| **Borda Count** (eq 6.28) | Score each identity by summing its rank positions; highest total wins |
| **Logistic Regression** (eq 6.29) | Learn a weighted combination of rank features |

Used in identification (1:N) scenarios where each matcher produces a candidate ranking rather than a single score.

---

## 5. Decision-Level Fusion

Each modality's matcher independently outputs a binary decision (accept/reject). These are combined:

| Rule | FAR | FRR | Effect |
|---|---|---|---|
| **AND** | Very low | High | Both must accept → strict; high security |
| **OR** | High | Very low | Either accepts → permissive; high convenience |
| **Majority Voting** (eq 6.30) | Balanced | Balanced | >50% must accept |
| **Weighted Majority Voting** (eq 6.31–6.32) | Tunable | Tunable | Weights by matcher reliability |
| **Bayesian Decision Fusion** | Principled | Principled | Uses prior probabilities + likelihoods |

(source: Unit3_Biometrics(1).pptx)

---

## Fusion Level Comparison

| Level | Information available | Flexibility | Practical ease |
|---|---|---|---|
| Sensor | Maximum (raw signal) | Low (format compatibility) | Hard |
| Feature | High (raw features) | Medium | Medium |
| **Score** | Medium (single number) | **High** | **Easy** (most deployed) |
| Rank | Low (ordinal only) | Medium | Medium |
| Decision | Minimum (binary) | Low | Easy but information loss |

---

## Related Pages

- [[multibiometrics]] — the 5 evidence sources and 3 system types that produce inputs for fusion
- [[pattern-recognition]] — the general pipeline fusion extends
- [[dimensionality-reduction]] — PCA/LDA used in feature-level fusion
- [[biometric-security]] — fusion can improve spoof resistance
