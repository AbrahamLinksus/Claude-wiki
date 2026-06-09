# Biometric Security

**Summary**: Threats, attack types, and vulnerability areas in biometric systems — what can go wrong and where attackers can strike.

**Pre-Req**: [[biometric-fundamentals]], [[biometric-system-modules]]

**Sources**: Unit4_Slides.pdf

**Last updated**: 2026-05-10

#biometrics

---

## 4 Security Threat Categories

| Threat | Description |
|---|---|
| **DoS (Denial of Service)** | Prevent legitimate users from being recognized — e.g., submitting degraded samples, overwhelming the system |
| **Intrusion** | Impersonate a legitimate enrolled user to gain unauthorized access |
| **Repudiation** | A legitimate user denies having performed a transaction; lack of non-repudiation |
| **Function Creep** | Biometric data collected for one purpose is used for a different, unauthorized purpose |

(source: Unit4_Slides.pdf)

---

## Two Attack Categories

### Intrinsic Failures (non-adversarial)
System errors that occur without any adversarial intent:

| Error | Meaning |
|---|---|
| **FMR** | False Match Rate — impostor accepted |
| **FNMR** | False Non-Match Rate — genuine user rejected |
| **FTE** | Failure to Enroll — sample too poor to enroll |
| **FTA** | Failure to Acquire — sensor fails to capture usable sample |

### Adversary Attacks
Deliberate attacks by malicious actors:

**Insider attacks**: from within the organization — can modify stored templates, alter thresholds, or access raw biometric data.

**Infrastructure attacks**: target the communication channels, databases, or system components.

---

## 5 Vulnerability Areas (A–E)

The biometric pipeline has 5 identified points where attacks can be mounted:

```
[A: Sensor] → [B: Feature Extractor] → [C: Matcher] → [D: Database] → [E: Channel]
```

| Area | Vulnerability | Attack |
|---|---|---|
| **A — Sensor** | Physical interface to biometric | **Spoofing**: fake finger, 3D-printed face, contact lens |
| **B — Feature Extractor** | Software module | Code injection, tampering with extraction algorithm |
| **C — Matcher** | Decision engine | **Hill-climbing**: iteratively modify probe to increase score |
| **D — Database** | Stored templates | Template theft, modification, substitution |
| **E — Channels** | Communication between modules | **MITM**, **replay attacks** |

---

## Attack Types in Detail

### Spoofing (at Sensor — Area A)
Creating a physical artifact that mimics a biometric trait:
- **Fingerprint**: gelatin/silicone fake finger, lifted latent print transferred to film
- **Face**: 2D photo, 3D mask, deepfake video
- **Iris**: patterned contact lens, printed iris image

Countermeasure: **liveness detection (PAD — Presentation Attack Detection)** — detect pulse, skin texture, blinking, 3D depth.

### Replay Attack (at Channel — Area E)
Intercepting a legitimate biometric signal and replaying it later:
- Capture the digital template or feature vector in transit
- Retransmit it to the matcher without re-capturing from a live finger

Countermeasure: challenge-response protocols, timestamps, session tokens.

### MITM — Man-in-the-Middle (at Channel — Area E)
Intercepting and modifying data between modules:
- Intercept feature vector between extractor and matcher
- Substitute a legitimate user's features for an impostor's

Countermeasure: encrypted communication channels between modules.

### Hill-Climbing Attack (at Matcher — Area C)
Iterative score-optimization attack:
1. Attacker has access to the match score output
2. Generates a synthetic probe and submits it
3. Receives score s₁
4. Modifies the probe slightly → submits again → receives s₂
5. If s₂ > s₁, keep modification; else revert
6. Repeat until score exceeds threshold → **accepted without a real biometric**

Countermeasure: limit score feedback (return only accept/reject, not the actual score); add noise to returned scores; rate-limit attempts.

### Template Theft / Modification (at Database — Area D)
- **Theft**: stolen templates can be used in replay attacks or to reconstruct the biometric
- **Modification**: replace a legitimate user's template with an impostor's
- **Critical**: unlike passwords, a biometric template cannot be changed once compromised

Countermeasure: [[template-protection]] — encrypt or transform templates so the raw biometric cannot be reconstructed.

---

## Why Biometric Security Is Uniquely Hard

1. **Biometrics are irrevocable**: you cannot change your fingerprint after a breach (unlike a password)
2. **Templates reveal sensitive information**: raw templates may reveal health status, identity linkage across databases
3. **Intra-class variation**: matching must tolerate variation, which also creates attack surface
4. **Cross-database linkage (function creep)**: same biometric template used across multiple systems allows identity tracking

→ These drive the need for [[template-protection]].

---

## Related Pages

- [[template-protection]] — cryptographic and algorithmic defenses against template threats
- [[biometric-system-modules]] — the 5 modules map to the 5 vulnerability areas
- [[biometric-traits]] — Circumvention factor (7th factor) covers spoofing and evasion
- [[multibiometrics]] — harder to spoof multiple traits simultaneously
