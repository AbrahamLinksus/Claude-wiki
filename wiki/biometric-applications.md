# Biometric Applications

**Summary**: Real-world deployments of biometric systems across government, commercial, forensic, and embedded domains.

**Pre-Req**: [[biometric-fundamentals]], [[multibiometrics]]

**Sources**: Unit5_Slides.pdf

**Last updated**: 2026-05-10

#biometrics

---

## Access Control

The most common commercial application. Replaces or supplements:
- **Something you know** (PIN, password)
- **Something you have** (token, smart card)
- **Something you are** (biometric — the 3rd factor)

Biometrics as the 3rd factor provides **non-repudiability** — the user cannot claim someone else used their credential.

Examples:
- Phone unlock: fingerprint (capacitive sensor) or face (structured light / IR)
- Office entry: fingerprint or palm vein reader at door
- Computer login: Windows Hello (face/fingerprint)

---

## Airport and Immigration

High-security, high-throughput identification:

**Automated border control (e-gates)**:
- Compare traveler's live face against passport chip photo (verification, 1:1)
- Often combined with fingerprint for higher confidence

**Watchlist screening**:
- Compare face against database of known persons of interest (identification, 1:N)
- Deployed in airport camera networks

**Visa issuance**:
- Fingerprint + face enrolled at consulate; checked against arrival database
- Ten-fingerprint slap capture (rolled for criminal records)

---

## Welfare and Civil ID

### Aadhaar (India)
Largest biometric civil ID system in the world — 1.4+ billion enrolled.

**Enrolled traits**: fingerprint (all 10) + iris (both) + face  
**Architecture**: centralized UIDAI database  
**Use cases**: welfare benefit disbursement, SIM card issuance, banking KYC, subsidized ration distribution  
**Verification mode**: 1:1 (Aadhaar number + biometric → confirm match)

The multimodal enrollment ensures near-universal coverage — if fingerprints are degraded (manual laborers), iris or face is used as fallback.

---

## Banking and Finance

### Fujitsu Palm Vein ATMs
- Near-infrared camera images the palm vein pattern
- Vein pattern is subcutaneous — cannot be lifted or spoofed with a surface artifact
- Deployed in Japanese banks; high acceptance due to hygiene (contactless)

### General Banking Use
- Branch teller: fingerprint for high-value transactions
- Mobile banking: fingerprint + face for app login
- KYC (Know Your Customer): face recognition against government ID photos

---

## Forensics — AFIS

**AFIS (Automated Fingerprint Identification System)**:
- 1:N identification against criminal/civil databases (millions of records)
- Processes **latent prints** (offline, partial, distorted) from crime scenes
- L1 pattern type used for indexing; L2 minutiae for matching
- Top candidates returned to human latent print examiner for final verification
- Used by FBI (NGI), Interpol, national police forces

---

## Embedded Biometrics

Biometric processing **on-device** — in resource-constrained hardware:

| Device | Modality | Challenge |
|---|---|---|
| ATM | Fingerprint, palm vein | Compact sensor, real-time matching |
| Smartphone | Fingerprint (capacitive/ultrasound under-display), face | Very low power, <500ms latency |
| Smart card | Fingerprint (match-on-card) | Extreme compute/memory constraints |
| Laptop | Fingerprint (sweep sensor) | Thin form factor, integration with OS |
| Industrial terminals | Fingerprint, face | Outdoor lighting, gloves, harsh environments |

**Match-on-card**: template stored on smart card chip; matching happens on the card itself — template never leaves the card, eliminating database breach risk.

**Real-time constraints**: embedded systems impose strict latency (<1s), power, and memory budgets. Algorithms must be optimized for MCUs or dedicated ASICs.

---

## Gender and Age Classification

Secondary application built on biometric infrastructure:
- Use face biometric features to classify demographic attributes
- Applications: retail analytics, targeted advertising, access restriction by age
- Concerns: bias (accuracy varies across demographic groups), privacy

---

## Industrial Automation

- Worker authentication at restricted machinery
- Time and attendance tracking (replace badge swipe with fingerprint)
- Hazardous area access (no physical contact with dirty hands → vein or iris)

---

## Application vs Modality Matrix

| Application | Primary Modality | Secondary |
|---|---|---|
| Phone unlock | Fingerprint / Face | — |
| Airport e-gate | Face | Fingerprint |
| AFIS (forensic) | Fingerprint (latent) | — |
| Aadhaar | Fingerprint + Iris | Face |
| ATM (Japan) | Palm vein | PIN |
| Match-on-card | Fingerprint | — |
| Workplace attendance | Fingerprint | Face |

---

## Related Pages

- [[biometric-fundamentals]] — verification vs identification modes used in each application
- [[multibiometrics]] — Aadhaar, airport use multimodal systems
- [[fingerprint-recognition]] — AFIS, phone unlock, attendance
- [[biometric-security]] — template protection critical for civil ID databases
- [[template-protection]] — match-on-card as a protection architecture
