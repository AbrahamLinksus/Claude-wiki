# Biometric Traits

**Summary**: The 7 factors that determine whether a physical or behavioral characteristic is suitable as a biometric, and the full taxonomy of biometric trait categories.

**Pre-Req**: [[biometric-fundamentals]]

**Sources**: Unit1_Biometrics(Part-1).pptx

**Last updated**: 2026-05-10

#biometrics

---

## The 7 Biometric Trait Factors

A characteristic must score well across all 7 to be a practical biometric.

| Factor | Definition | Failure mode |
|---|---|---|
| **Universality** | Every person in the target population must possess the trait | Excludes populations missing the trait (e.g., amputees for fingerprint) |
| **Uniqueness** | No two individuals share the same trait value | High inter-class similarity → high FMR/FAR |
| **Permanence** | Trait is stable over time | Ageing, injury, or disease changes → high FRR/FNMR |
| **Measurability** | Trait can be captured by a sensor without discomfort | Poor capture → FTE / FTA |
| **Performance** | System achieves acceptable accuracy, speed, and robustness | Depends on sensor quality + algorithm quality |
| **Acceptability** | Individuals are willing to present the trait | Cultural/hygiene concerns reduce cooperation |
| **Circumvention** | Trait is difficult to imitate or evade | Two attack vectors (see below) |

(source: Unit1_Biometrics(Part-1).pptx)

### Circumvention — Two Attack Vectors

1. **Spoofing (imitation)**: attacker creates an artifact (fake finger, 3D-printed face, contact lens) to impersonate a legitimate user
2. **Obfuscation (evasion)**: attacker deliberately alters their own trait to avoid recognition — e.g., sanding fingerprints, wearing disguises

See [[biometric-security]] for detailed attack taxonomy and countermeasures.

---

## Unimodal Limitations

Any single-trait system faces four inherent weaknesses:
1. **Noisy data** — sensor failures, environmental interference
2. **Inter-class similarity** — some people share very similar trait values
3. **Population incompatibility** — a fraction of the population cannot provide the required trait
4. **Spoof vulnerability** — a single trait is easier to fake than multiple

→ These drive the need for [[multibiometrics]].

---

## Trait Taxonomy (5 Categories)

### Visual Biometric
Captured by camera or optical sensor; most widely deployed.

| Trait | Notes |
|---|---|
| Fingerprint | Most mature; L1/L2/L3 feature levels |
| Face | Passive capture; aging challenge |
| Iris | High uniqueness; even identical twins differ |
| Retina | Very high uniqueness; invasive scan |
| Ear | Passive; less mature |
| Vein Geometry | Near-infrared imaging; hand/palm/finger |

### Auditory Biometric
| Trait | Notes |
|---|---|
| Voice / Speaker recognition | Behavioral + physiological combined; affected by illness |

### Visual/Spatial Biometric
Captured geometrically or as motion trajectory.

| Trait | Notes |
|---|---|
| Hand Geometry | Palm width/length measurements |
| Finger Geometry | Finger length/width |
| Signature | Dynamic (pressure + speed) or static (shape only) |
| Handwriting | Character-level patterns |

### Chemical Biometric
| Trait | Notes |
|---|---|
| DNA Matching | Highest uniqueness; slow; requires lab; privacy concerns |

### Behavioral Biometric
| Trait | Notes |
|---|---|
| Gait | Recognized from walking pattern; fully passive at a distance |

---

## Physiological vs Behavioral

- **Physiological** (relatively stable): Iris, Fingerprint, Ear, DNA, Vein print, Face
- **Behavioral** (learned/variable over time): Voice, Gait, Signature

Behavioral traits have lower permanence — they change with mood, health, and habit.

---

## Related Pages

- [[biometric-fundamentals]] — how traits are used in the recognition pipeline
- [[biometric-system-modules]] — the hardware and software that captures these traits
- [[multibiometrics]] — combining multiple traits to overcome unimodal limits
- [[fingerprint-recognition]] — detailed treatment of the most deployed visual biometric
