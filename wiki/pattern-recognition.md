# Pattern Recognition

**Summary**: The general pattern recognition pipeline that underlies all biometric systems — from raw sensor input to a classified decision.

**Pre-Req**: [[biometric-fundamentals]], [[image-segmentation]]

**Sources**: Unit2_Biometrics(1).pptx

**Last updated**: 2026-05-10

#biometrics

---

## What Is Pattern Recognition?

Pattern recognition is the automated assignment of an input sample to one of a set of predefined classes. In biometrics, each **class** is an enrolled identity and the **sample** is a probe biometric capture.

A biometric system is a **special case** of a pattern recognition system.

Formally, the goal is to learn a function:
```
f: X → W
```
- X = feature space (all possible feature vectors)
- W = class space (all enrolled identities or accept/reject)

A training/enrollment dataset is the set of (x, w) pairs used to configure the system.

---

## The Pattern Recognition Pipeline

```
Input Signal
    ↓
[Sensor]             — capture raw biometric signal
    ↓
[Segmentation]       — isolate ROI from background  (see [[image-segmentation]])
    ↓
[Feature Extraction] — transform ROI into feature vector x
    ↓
[Classification]     — assign x to a class w
    ↓
[Post-processing]    — apply decision rules, thresholds, fusion
    ↓
Decision
```

(source: Unit2_Biometrics(1).pptx)

Each stage reduces the dimensionality and noise while preserving the information needed for the decision.

---

## Feature Vectors

The **feature vector** x = (x₁, x₂, …, x_d) encodes the discriminative properties of the sample.

Good features are:
- **Discriminative**: different classes produce different feature vectors
- **Invariant**: same class produces similar feature vectors despite intra-class variation (pose, lighting, noise)
- **Compact**: low-dimensional; see [[dimensionality-reduction]]

Examples:
- Fingerprint: minutiae set {(x_i, y_i, θ_i)} → often reduced via LDA
- Face: pixel intensities → PCA (eigenfaces) → compact vector
- Voice: MFCCs (Mel-frequency cepstral coefficients)

---

## Classification

Maps the feature vector to a class. Common approaches in biometrics:

| Classifier | How |
|---|---|
| **Threshold / distance-based** | Accept if distance to template < τ |
| **Bayesian** | Choose class maximizing posterior P(w\|x) |
| **SVM** | Find hyperplane maximizing margin between classes |
| **k-NN** | Assign class of nearest k training samples |
| **Neural Network** | Learned non-linear decision boundary |

In verification, the "classifier" is the matcher + threshold. In identification, it's a 1:N search.

---

## Decision Theory

How to make optimal decisions given uncertainty:

**Normative decision theory**: prescribes how agents *should* decide given a value function and constraints.
- Value-based, goal-oriented, idealistic
- Used in system design: set threshold to minimize expected cost (FAR/FRR trade-off)

**Descriptive decision theory**: models how agents *actually* decide.
- Empirical/behavioral; derived from surveys, experiments
- Used in understanding user behavior with biometric systems (acceptability factor from [[biometric-traits]])

(source: Unit2_Biometrics(1).pptx)

---

## Bayes' Theorem in Classification

Used in the [[dimensionality-reduction]] LDA classifier and in [[biometric-fusion]] decision-level fusion:

```
P(Y=k | X=x) = P(Y=k) · f_k(x) / Σ_l P(Y=l) · f_l(x)
```

- P(Y=k) = prior probability of class k
- f_k(x) = class-conditional density (likelihood of x given class k)
- Denominator = normalization constant

The class with the highest posterior is selected.

---

## Related Pages

- [[biometric-fundamentals]] — verification vs identification as pattern recognition problems
- [[image-segmentation]] — the segmentation stage of the pipeline
- [[fingerprint-feature-extraction]] — concrete feature extraction for fingerprints
- [[dimensionality-reduction]] — PCA/LDA for feature compression before classification
- [[biometric-fusion]] — extends the pipeline to multi-trait decisions
