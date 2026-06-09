# Dimensionality Reduction

**Summary**: PCA and LDA — the two core dimensionality reduction techniques used in biometric feature processing, with full formulas and a worked example.

**Pre-Req**: [[pattern-recognition]], linear algebra (eigenvectors, matrices)

**Sources**: Unit2_Biometrics(1).pptx

**Last updated**: 2026-05-10

#biometrics

---

## Why Reduce Dimensions?

Feature vectors from biometric sensors can be very high-dimensional (e.g., thousands of pixels for a face image). High dimensionality causes:
- **Curse of dimensionality**: sparse data in high-dimensional space degrades classifier performance
- **Computational cost**: matching is slower with large vectors
- **Overfitting**: more features than samples leads to poor generalization

Dimensionality reduction maps x ∈ ℝ^d → z ∈ ℝ^k where k ≪ d, retaining the most useful information.

---

## PCA — Principal Component Analysis

### What It Does
Finds the directions (principal components) of **maximum variance** in the data and projects onto them.

**Unsupervised**: ignores class labels — optimizes for variance, not class separability.

### Algorithm
1. **Standardize** features to zero mean and unit variance
2. Compute the **covariance matrix** Σ (d × d)
3. **Eigen decomposition**: find eigenvalues λ and eigenvectors v such that Σv = λv
   - Equivalently: solve det(Σ − λI) = 0 for eigenvalues
4. **Order** eigenvalues in decreasing order: λ₁ ≥ λ₂ ≥ … ≥ λ_d
5. **Select top k** eigenvectors (those with largest λ) as the projection matrix P (d × k)
6. **Transform**: z = P^T · x

Each eigenvalue λᵢ represents the variance explained by the i-th principal component.

### Worked Example
(source: Unit2_Biometrics(1).pptx)

Original data: 5 samples × 4 features (5×4 matrix)  
Eigenvalues: λ = [2.516, 1.065, 0.394, 0.025]  
Total variance = 4.0

- λ₁ = 2.516 → explains 62.9% of variance
- λ₂ = 1.065 → explains 26.6% of variance
- Cumulative top-2: **89.5%** → select k=2

Result: 5×4 feature matrix → 5×2 projected matrix. 4 features → 2 components retaining ~90% of information.

### Limitation in Biometrics
PCA does not use class label information. Two classes with similar distributions but different means may not be separated well after PCA projection. This motivates LDA.

---

## LDA — Linear Discriminant Analysis

### What It Does
Finds the projection directions that **maximize class separability** — wide separation between class means, tight variance within each class.

**Supervised**: uses class labels. Unlike PCA, it explicitly optimizes for discrimination.

### Key Matrices

**Between-class scatter matrix S_b** (how far apart class means are from the global mean):
```
S_b = Σᵢ Nᵢ (x̄ᵢ − x̄)(x̄ᵢ − x̄)^T
```
- Nᵢ = number of samples in class i
- x̄ᵢ = mean of class i
- x̄ = global mean

**Within-class scatter matrix S_w** (spread of samples around their class mean):
```
S_w = Σᵢ (Nᵢ − 1)Sᵢ = Σᵢ Σⱼ (xᵢⱼ − x̄ᵢ)(xᵢⱼ − x̄ᵢ)^T
```
- Sᵢ = covariance matrix of class i

### Fisher's Criterion

Find projection matrix P_lda that maximizes the ratio of between-class to within-class scatter:

```
P_lda = argmax_P  |P^T S_b P| / |P^T S_w P|
```

Maximizing this ratio = pulling classes apart while keeping each class compact.

Solution: generalized eigenvectors of S_w⁻¹ S_b.

### Bayesian Prediction

After projecting to LDA space, class assignment uses Bayes' theorem:

```
P(Y=k | X=x) = P(Y=k) · f_k(x) / Σ_l P(Y=l) · f_l(x)
```

**Discriminant function** for class k (linear Gaussian case):

```
D_k(x) = x · (μ_k / σ²) − (μ_k² / 2σ²) + ln(π_k)
```

- μ_k = class mean
- σ² = shared variance
- π_k = prior P(Y=k)

Assign x to the class k with the **highest** D_k(x).

(source: Unit2_Biometrics(1).pptx)

---

## PCA vs LDA Comparison

| | PCA | LDA |
|---|---|---|
| **Type** | Unsupervised | Supervised |
| **Uses class labels** | No | Yes |
| **Optimizes for** | Maximum variance | Maximum class separability |
| **When to use** | Preprocessing / compression when classes unknown | Classification when labeled data available |
| **Max components** | min(d, n−1) | min(C−1, d) where C = number of classes |
| **In biometrics** | Face recognition (eigenfaces), initial compression | Identity classification after compression |

**Typical pipeline**: PCA first (compress d→m), then LDA on m-dimensional space (supervised separation).

---

## Related Pages

- [[pattern-recognition]] — where dimensionality reduction fits in the pipeline
- [[fingerprint-feature-extraction]] — features that are fed into PCA/LDA
- [[biometric-fusion]] — score-level fusion can follow LDA classification
