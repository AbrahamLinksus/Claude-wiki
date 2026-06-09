# Template Protection

**Summary**: Methods for securing stored biometric templates so that a database breach cannot compromise the underlying biometric — covering the 3 required properties and 4 protection approaches.

**Pre-Req**: [[biometric-security]], [[biometric-system-modules]]

**Sources**: Unit4_Slides.pdf

**Last updated**: 2026-05-10

#biometrics

---

## Why Template Protection?

Biometric templates are permanent — unlike passwords, they cannot be changed after compromise. A stolen template:
- Enables **replay attacks** (resubmit the template to any system)
- Can be used to **reconstruct the original biometric** (partial or full)
- Enables **cross-database linking** (same biometric across multiple systems → privacy violation)

Template protection schemes ensure that even a full database breach reveals nothing actionable.

(source: Unit4_Slides.pdf)

---

## 3 Required Properties

Any viable template protection scheme must satisfy all three:

| Property | Requirement |
|---|---|
| **Cryptographic Security** | Protected template reveals no information about the underlying biometric (computational security guarantee) |
| **Performance** | Recognition accuracy with the protected template must be comparable to the unprotected system |
| **Revocability / Cancelability** | If a protected template is compromised, it can be invalidated and replaced with a new one from the same biometric (using a different transformation key) |

These three requirements create tension: stronger cryptographic security typically degrades performance because it reduces the tolerance for intra-class variation.

---

## 4 Protection Approaches

### 1. Standard Encryption

Apply symmetric (AES) or public-key encryption to the stored template.

**Stored**: encrypted template E(x^E)  
**Auth**: decrypt → compare to probe features  

**Critical limitation**: during matching, the template must be **decrypted in memory** — the plaintext exists briefly, creating an attack window. Also does not handle intra-class variation well (exact match required after decryption).

---

### 2. Feature Transformation — Invertible

Apply a **reversible transformation** f(x, κ) using a user-specific key κ.

**Stored (public)**: f(x^E, κ) — the transformed template  
**Stored (secret)**: key κ  

**Auth**: transform probe → f(x^A, κ) → match in transformed domain:
```
m_t(f(x^E, κ), f(x^A, κ))
```

**Security**: relies on secrecy of key κ  
**Revocability**: change κ → different protected template from same biometric  
**Intra-class variation**: handled via quantization in transformed domain  

Example: **Random Multispace Quantization (A_κ)** — projects biometric features into a random subspace defined by κ.

---

### 3. Feature Transformation — Non-Invertible

Apply a **one-way transformation** — given f(x^E, κ), it is computationally infeasible to recover x^E.

**Stored (public)**: f(x^E; K), key κ  

**Auth**: transform probe → f(x^A, κ) → match in transformed domain

**Security**: relies on non-invertibility of f (even if attacker has f(x^E) and κ, cannot recover x^E)  
**Revocability**: change κ → new irreversible transform  

Examples:
- **Cartesian fingerprint transform**: maps minutiae to Cartesian representation under key
- **Polar fingerprint transform**: maps minutiae to polar coordinates under key

---

### 4. Biometric Cryptosystems

Bind cryptographic keys to biometric templates. Two sub-types:

#### Key-Binding Schemes
An external cryptographic key κ is **bound** to the enrollment template:

```
Helper data H = h(x^E, κ)
```

**Stored (public)**: helper data H  
**Not stored**: key κ, raw template x^E  

**Auth**: probe x^A + H → recover κ = m(f(x^E, κ), x^A) using error correction  

Example: **Fuzzy Commitment Scheme** — encodes κ as a codeword, XORs with biometric feature vector; decoding at auth uses error correction to recover κ despite intra-class variation.

#### Key-Generating Schemes
The key κ is **derived entirely from the biometric** — no external key:

```
Helper data H = h(x^E)
```

**Enrollment**:
```
x^E → h → h(x^E) stored as H
```

**Authentication** (Fig 7.21):
```
x^A + H (h(x^E)) → Recovery → Fuzzy Extractor → Secret Key κ
```

**Stored (public)**: helper data H only  
**Not stored**: key κ, raw template x^E — κ is regenerated from x^A + H  

**Mechanism**: user-specific quantization boundaries stored in H; error correction coding handles intra-class variation; κ is reproduced if x^A is close enough to x^E.

(source: Unit4_Slides.pdf, Fig 7.21)

---

## Table 7.1 — Complete Template Protection Comparison

| Approach | Security Feature | Entities Stored | Handles Intra-user Variation |
|---|---|---|---|
| **Invertible transform** | Secrecy of key κ | Public: f(x^E, κ); Secret: κ | Quantization in transformed domain: m_t(f(x^E,κ), f(x^A,κ)) |
| **Non-invertible transform** | Non-invertibility of f | Public: f(x^E; K), key κ | Matching in transformed domain: m_t(f(x^E;K), f(x^A,κ)) |
| **Key-binding cryptosystem** | Info revealed by H = h(x^E, κ) | Public: Helper Data H | Error correction + user quantization: κ = m(f(x^E,κ), x^A) |
| **Key-generating cryptosystem** | Info revealed by H = h(x^E) | Public: Helper Data H | Error correction + user quantization: κ = m(h(x^E), x^A) |

(source: Unit4_Slides.pdf, Table 7.1)

---

## Key Distinction: Key-Binding vs Key-Generating

| | Key-Binding | Key-Generating |
|---|---|---|
| **Key source** | External; bound to template | Derived from biometric itself |
| **Helper data contains** | h(x^E, κ) — mixed biometric + key info | h(x^E) — biometric info only |
| **If H is compromised** | Both biometric and key info exposed | Only biometric helper info exposed |
| **Key generation** | Key is recovered | Key is regenerated (reproducible) |

---

## Related Pages

- [[biometric-security]] — the threat model that template protection addresses
- [[biometric-system-modules]] — the database module that stores protected templates
- [[biometric-applications]] — match-on-card as an alternative to centralized template storage
